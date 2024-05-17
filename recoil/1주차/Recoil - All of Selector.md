# Recoil - All of Selector

```typescript
function selector<T>({
  key: string,

  get: ({
    get: GetRecoilValue,
    getCallback: GetCallback,
  }) => T | Promise<T> | Loadable<T> | WrappedValue<T> | RecoilValue<T>,

  set?: (
    {
      get: GetRecoilValue,
      set: SetRecoilState,
      reset: ResetRecoilState,
    },
    newValue: T | DefaultValue,
  ) => void,

  dangerouslyAllowMutability?: boolean,
  cachePolicy_UNSTABLE?: CachePolicy,
})
```

```typescript
type ValueOrUpdater<T> = T | DefaultValue | ((prevValue: T) => T | DefaultValue);
type GetCallback =
  <Args, Return>(
    callback: CallbackInterface => (...Args) => Return,
  ) => (...Args) => Return;

type GetRecoilValue = <T>(RecoilValue<T>) => T;
type SetRecoilState = <T>(RecoilState<T>, ValueOrUpdater<T>) => void;
type ResetRecoilState = <T>(RecoilState<T>) => void;

type CachePolicy =
  | {eviction: 'lru', maxSize: number}
  | {eviction: 'keep-all'}
  | {eviction: 'most-recent'};
```

<br><hr>

## 1. get

### 1-1. get: GetRecoilValue

다른 `Atom`이나 `Selector`를 인자로 받아 해당 값을 가져오는 함수.

<br>

### ⭐ 1-2. getCallback: GetCallback

Recoil 상태에 비동기적으로 접근할 수 있는 콜백을 생성하는 함수. _(무한한 가능성이 느껴집니다.)_

❗ 리액트 외부에서 리액트 상태를 컨트롤 하기에 유용하다.

❗ 비동기 작업 후 수행할 액션으로 등록하기에 유용하다.

❗ 사이드 이팩트를 처리할 수 있다.<br>
_(얘는 그냥 미쳤다. 상태를 참조하는 로직을 모두 정의할 수 있다. 적어도 상태참조에 한해서는 유틸도 핸들러도 커스텀훅도 필요가 없는 것)_

_(단, 상태 변경(set)과 관련된 로직은 `useRecoilCallback`을 공부해보도록 하자)_

<br>

### ⭐ 1-3. getCallback 사용 예시

#### (1) 이벤트 핸들러 함수를 반환

버튼을 클릭하면 특정 ID의 모달을 플로팅하는 onClick 함수를 반환하는 예시

```typescript
const menuItemState = selectorFamily({
  key: 'MenuItem',
  get:
    (itemID) =>
    ({ get, getCallback }) => {
      const name = get(itemNameQuery(itemID)); // 동기적으로 Recoil 상태 조회
      const onClick = getCallback(({ snapshot }) => async () => {
        const info = await snapshot.getPromise(itemInfoQuery(itemID)); // 비동기적으로 Recoil 상태 조회
        displayInfoModal(info); // 조회한 상태를 사용하여 정보 모달 표시
      });
      return {
        title: `Show info for ${name}`,
        onClick,
      };
    },
});
```

```typescript
function Menu() {
  const itemID = '123'; // 예시 아이템 ID
  const menuItem = useRecoilValue(menuItemState(itemID));

  return (
    <div>
      <button onClick={menuItem.onClick}>{menuItem.title}</button>
    </div>
  );
}
```

<br>

#### ⭐ (2) 관련된 파생상태/사이드이펙트 그룹화

위 예시는 selectorFamily를 사용하여 구현하였기 때문에, 특정 key의 상태를 갖고 있었다.

selectorFamily 대신 selector를 사용하고, 콜백함수에서 id를 인자로 받도록 구현해보자.

하나의 도메인에 대한 다양한 파생상태 및 사이드 이펙트를 발생시키는 함수들을 단 1개의 selector로 그룹화할 수 있다.

컴포넌트에서 너무나 많은 상태를 Import 하고 있다고? 이걸로 해결해보자!

```typescript
const menuItemState = selector({
  key: 'MenuItem',
  get: ({ getCallback }) => {
    // 가격을 조회하는 콜백이라던가,
    const getPriceById = getCallback(({ snapshot }) => async (id) => {
      return snapshot.getPromise(itemPromise(id));
    });

    // 해당 상품의 상세정보를 띄우는 콜백이라던가,
    const openDetailModalById = getCallback(({ snapshot }) => async (id) => {
      const info = await snapshot.getPromise(itemInfoQuery(id));
      displayInfoModal(info);
    });
    return {
      getPriceById,
      openDetailModalById,
    };
  },
});
```

더 다양하게 활용할 수 있을 것 같은데, 아직 많이 사용해보지 않아서 떠오르지 않는다.

_(useRecoilCallback는 컴포넌트에 코드를 작성해야하는 단점이 있지만 더 강력하다!)_

<br><hr>

## 2. set

나는 `set`을 이용해서 다양한 사이드이펙트를 처리하도록 했다.

`set`에서는 `useRecoilCallback`를 사용하지 않고도 상태를 변경할 수 있기 때문이다.

그러나 올바른 사용 용도는 `get`에서 반환하는 파생상태를 `newValue`로 받은 후, `get`의 역산으로 상태를 추출하여 저장하는 것이다. _(주의할 것!!)_

<br>

### 2-1. get: GetRecoilValue

`get`에서 설명한 바와 동일하다.

<br>

### 2-2. set: SetRecoilState

`atom` 또는 `select`의 상태를 변경할 수 있다.

<br>

### 2-3. reset: ResetRecoilState

`atom` 또는 `select`의 상태를 초기화 할 수 있다. `DefaultValue` 객체를 전달하는 듯 하다.

`reset`을 유용하게 사용하려면 `atom`의 `effect`에서 `onSet`을 사용할 수 있다. `onSet`의 `newValue`초기화 했을 때의 액션을 정의할 수 있다.

<br>

### ⭐ 2-4. set 사용방법

`selector`의 `set`은 optional이다. 필요하다면 `selector`를 선언할 때 추가할 수 있다. 만약 `set`을 선언하려면 `set`의 `newValue`의 타입을 `get`의 반환타입과 일치시켜야 한다.

다음은 사용자의 이름(`name`)과 이메일(`email`)을 `selector`로 관리하는 예시이다.

이 예시코드에서는 객체를 구조분해할당하여 각 상태에 대입시켜주는 단순한 로직을 수행하지만, 파생데이터를 입력받아 전처리 후 삽입하는 등의 방식에서도 유용하다.

```typescript
import { atom, selector } from 'recoil';

// 사용자 이름을 관리하는 atom
const userNameState = atom({
  key: 'userNameState',
  default: '',
});

// 사용자 이메일을 관리하는 atom
const userEmailState = atom({
  key: 'userEmailState',
  default: '',
});

// 사용자 프로필을 관리하는 selector
const userProfileState = selector({
  key: 'userProfileState',
  get: ({ get }) => {
    const name = get(userNameState);
    const email = get(userEmailState);
    return { name, email };
  },
  set: ({ set, reset }, newValue) => {
    if (newValue instanceof DefaultValue) {
      // 프로필을 기본값으로 리셋
      reset(userNameState);
      reset(userEmailState);
    } else {
      // 프로필 업데이트
      set(userNameState, newValue.name);
      set(userEmailState, newValue.email);
    }
    j;
  },
});

// 프로필 업데이트 함수 예시
function updateProfile(newProfile) {
  set(userProfileState, newProfile);
}

// 프로필 리셋 함수 예시
function resetProfile() {
  reset(userProfileState);
}
```

<br>

### 2-5. dangerouslyAllowMutability

`dangerouslyAllowMutability`는 `atom`과 `selector`가 공통으로 갖고 있는 옵션이다. Recoil 상태는 불변성 보장을 원칙으로 하는데, 이 옵션은 불변성을 위반하는 것을 허용한다.

다음과 같이 엄청난 크기의 데이터를 상태로 저장하고 있다고 가정하자. _(가독성을 위해 name, age만 추가하였다.)_ 다음의 Selector는 엄청난 양의 데이터 중

나는 이 데이터의 일부만 변경하고 싶을 수 있다. 만약 이것이 자주 발생할 경우 객체를 매 번 객체를 복사하는 것은 성능 오버헤드를 발생시키는데, `dangerouslyAllowMutability`는 이를 개선하는데 도움을 주는 옵션이다.

```typescript
import { atom, selector } from 'recoil';

// 만약 엄청난 크기의 API 응답을 갖고 있다고 가정하자
// (API 응답이 아니라도 상관 없다)
interface APIResponse {
  name: string;
  age: number;
}

export const apiResponseState = atom<APIResponse>({
  key: 'apiResponseState',
  default: {
    name: 'test',
    age: 0,
  },
  dangerouslyAllowMutability: true,
});

export const firstSelector = selector({
  key: 'testSelect',
  get: ({ get }) => {
    const response = get(apiResponseState);

    // dangerouslyAllowMutability가 false 이라면..
    const newResponse = { ...response, name: '나는 요것만 변경하고 싶었는데...' };

    return newResponse;
  },
});

export const secondSelector = selector({
  key: 'testSelect',
  get: ({ get }) => {
    const response = get(apiResponseState);

    // dangerouslyAllowMutability가 true라면!!!
    response.name = '나는 요것만 변경했지롱';

    return response;
  },
});
```

단, 이것은 Recoil의 불변성 보장 원칙을 위반하는 것이기에 성능 상의 메리트가 존재하지 않는다면 권장되지 않는 방법이다.

<br>

### 2-6. cachePolicy_UNSTABLE

이 옵션은 장기적으로 봤을 때, `selector`의 성능을 낮추는 대신 프로그램의 성능을 높이는 옵션이다. _(이 옵션을 사용하기 위해서는 캐시에 대한 높은 이해도를 필요로 한다.)_

이를 설명하기 위해 잠시 Recoil의 동작에 대해 얘기해보자.

만약 당신의 `selector`이 4개의 상태를 참조하여 파생 상태를 생성한다고 가정하자. 이 때, 각 상태가 한 번씩만 업데이트 되더라도 `select`의 연산은 4번 수행된다. 만약 내부 로직이 무겁다면..? 엄청난 성능 오버헤드가 발생할 것이다.

따라서 Recoil은 `useMemo`와 같이 동작한다. 각 상태는 의존성 배열을 갖고, 결과 값을 캐시한다. 각 상태가 한 번씩 업데이트 되었다면 4개의 의존성 배열 상태가 캐시된 것이다.

여기서 문제. Recoil은 캐시를 어떻게 관리하고 있을까?<br>
정답은 무제한으로 저장한다. 즉 의존성 배열이 빈번하게 업데이트 된다면 언젠가 내 Recoil은 메모리의 대주주가 되어있을지도 모른다.

즉, 이 옵션은 캐시를 어떻게 삭제할지에 대한 것이다.

#### (1) keep-all : Default

- keep-all 정책은 셀렉터의 모든 의존성과 그 값들을 무기한으로 캐시에 저장한다.

```typescript
import { selector } from 'recoil';

const mySelector = selector({
  key: 'mySelector',
  get: ({ get }) => {
    const someValue = get(someAtom);
    // 복잡한 연산 수행
    return someValue * 2;
  },
  cachePolicy_UNSTABLE: {
    eviction: 'keep-all',
  },
});
```

#### (2) lru (Least Recently Used)

- lru 정책은 캐시의 크기가 설정된 최대 크기(maxSize)를 초과할 때 가장 최근에 사용되지 않은 값을 캐시에서 제거한다.

```typescript
import { selector } from 'recoil';

const mySelector = selector({
  key: 'mySelector',
  get: ({ get }) => {
    const someValue = get(someAtom);
    // 복잡한 연산 수행
    return someValue * 2;
  },
  cachePolicy_UNSTABLE: {
    eviction: 'lru',
    maxSize: 50, // 최대 50개의 값까지 캐시
  },
});
```

#### (3) most-recent

- most-recent 정책은 캐시 크기를 1로 설정하여 가장 최근에 저장된 의존성과 그 값만을 유지한다.

```typescript
import { selector } from 'recoil';

const mySelector = selector({
  key: 'mySelector',
  get: ({ get }) => {
    const someValue = get(someAtom);
    // 복잡한 연산 수행
    return someValue * 2;
  },
  cachePolicy_UNSTABLE: {
    eviction: 'most-recent',
  },
});
```
