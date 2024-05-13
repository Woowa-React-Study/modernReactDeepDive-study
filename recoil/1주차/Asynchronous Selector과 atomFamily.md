# Asynchronous Selector

> 💡 비동기 함수는 selector를 사용합니다.

## atom 과 비동기 함수

### atom에서 비동기 함수를 사용할 수 없는 이유

> Recoil의 atom은 상태를 나타내는 데 사용되며, 동기적으로 값을 반환해야 합니다.

##### 1. Atom은 컴포넌트가 렌더링될 때 즉시 값을 제공해야 합니다.

- 비동기 함수를 사용하면 값을 바로 반환할 수 없고, Promise를 반환하게 됩니다. 이는 컴포넌트의 렌더링을 지연시키거나 오류를 발생시킬 수 있습니다.

##### 2. Atom의 값은 다른 atom이나 selector에서 동기적으로 접근할 수 있어야 합니다.

- 비동기 함수를 사용하면 이러한 동기적인 접근이 어려워집니다.

##### 3. Recoil은 상태 변화를 추적하고 최적화를 위해 atom의 값을 캐싱합니다.

- 비동기 함수를 사용하면 캐싱과 상태 관리가 복잡해질 수 있습니다.

### 그럼에도 atom에서 비동기 함수를 상태로 사용하는 방법?

초기값이 null이거나 기본값을 설장한 atom의 상태를 useEffect를 사용해서 비동기 작업의 결과를 atom 값으로 업데이트합니다.

```js
const myAtom = atom({
  key: "myAtom",
  default: null,
});

function MyComponent() {
  const [value, setValue] = useRecoilState(myAtom);

  useEffect(() => {
    async function fetchData() {
      const response = await fetch("https://api.example.com/data");
      const data = await response.json();
      setValue(data);
    }

    fetchData();
  }, []);

  // ...
}
```

## selector와 비동기 함수

### selector에서 비동기 함수를 사용할 수 있는 이유

##### 1. Selector는 파생된 상태를 나타냅니다.

##### 2. Selector의 get 함수는 값을 동기적으로 반환할 필요가 없습니다.

##### 3. Recoil은 selector의 비동기 작업을 자동으로 처리합니다.

selector를 사용하는 컴포넌트는 비동기 작업의 완료를 기다리지 않고 렌더링될 수 있습니다. Recoil은 비동기 작업이 완료되면 컴포넌트를 자동으로 다시 렌더링하여 업데이트된 값을 표시합니다.

##### 4. Selector는 캐싱과 최적화를 지원합니다.

Recoil은 selector의 결과를 캐싱하므로, 동일한 입력에 대해 반복적인 계산을 피할 수 있습니다. **비동기 작업의 결과도 캐싱되므로, 불필요한 API 호출을 줄일 수 있습니다.**

### selector에서 비동기 함수 사용

```jsx
const myAsyncSelector = selector({
  key: "myAsyncSelector",
  get: async ({ get }) => {
    const id = get(myAtom);
    const response = await fetch(`https://api.example.com/data/${id}`);
    const data = await response.json();
    return data;
  },
});

const data = useRecoilValue(myAsyncSelector);
```

1. myAtom의 값이 변경되면, myAsyncSelector의 get 함수가 다시 호출된다.
2. myAtom의 값이 변경되면 data값도 변경됩니다.

#### myAtom의 값이 변경되었지만 실제로 data의 값이 변경되지 않은 경우,예를 들어 API 응답이 이전과 동일한 데이터를 반환한 경우에는 어떻게 될까🤔?

**Recoil은 myAsyncSelector의 결과가 변경되었는지 감지**합니다. **만약 이전 결과와 현재 결과가 동일하다면, Recoil은 data를 구독하는 컴포넌트의 리렌더링을 트리거하지 않습니다.** 이는 Recoil의 기본 동작으로, 불필요한 리렌더링을 방지하고 성능을 최적화합니다.

### Suspense, ErrorBoundary와 함께 사용하기

비동기 작업을 구독하넌 컴포넌트는 React의 Suspense, ErrorBoundary화 함께 사용하면 시너지 효과가 있습니다.

비동기 작업이 완료되기 전까지 Suspense에서 설정한 작업을 보여줄 수 있고 오류가 나면 ErrorBoundary의 오류 화면을 보여 줄 수 있습니다.

```js
function App() {
  return (
    <div>
      {/* ... */}
      <ErrorBoundary FallbackComponent={ErrorFallback}>
        <Suspense fallback={<div>Loading...</div>}>
          <ProductList />
        </Suspense>
      </ErrorBoundary>
    </div>
  );
}
```

### selector의 비동기 작업을 어떻게 Suspense, ErrorBoundary가 받을까?

**Recoil은 React의 Context를 사용한다. selector의 비동기 작업 상태,결과도 Context를 타고 Suspense와 ErrorBoundary에 전달됩니다.**

#### selector와 Suspense 작동 순서

1. Recoil의 비동기 selector가 실행되면, Recoil은 해당 selector를 추적하기 시작합니다.
2. 비동기 작업이 진행 중인 동안, Recoil은 React의 useTransition 훅을 사용하여 Suspense에게 "보류 중" 상태를 알립니다.
3. Suspense는 보류 중인 상태를 받아 fallback UI를 렌더링합니다.
4. 비동기 작업이 완료되면, Recoil은 useTransition 훅을 사용하여 Suspense에게 "완료" 상태를 알립니다.
5. Suspense는 완료 상태를 받아 보류 중이던 컴포넌트를 렌더링합니다.

---

# atomFamily

```ts
function atomFamily<T, P: Parameter>({
  key: string,

  default?:
    | T
    | Promise<T>
    | Loadable<T>
    | WrappedValue<T>
    | RecoilValue<T>
    | (P => T | Promise<T> | Loadable<T> | WrappedValue<T> | RecoilValue<T>),

  effects?:
    | $ReadOnlyArray<AtomEffect<T>>
    | (P => $ReadOnlyArray<AtomEffect<T>>),

  dangerouslyAllowMutability?: boolean,
}): P => RecoilState<T>
```

- key: 내부적으로 atom을 식별하는 키
- default: atom의 초기값
- effects: atom이 생성될 때 실행되는 함수의 배열
- dangerouslyAllowMutability : 불변이 아닌 가변으로 상태가 존재하는 지(Recoil의 불변성을 해쳐서 권장되지 않음)

### what?

**동일한 형태**의 **여러 atom**을 매개 변수를 사용해 **동적으로 생성**하는데 사용되는 유틸 함수입니다. 생성된 atom을 반환합니다.

#### 메모이제이션과 최적화

atomFamily는 매개변수를 기반으로 atom을 메모이제이션합니다. 즉, **동일한 매개변수로 atomFamily를 호출하면 이전에 생성된 아톰이 반환**됩니다. 이를 통해 불필요한 재렌더링을 방지하고 성능을 최적화할 수 있습니다.

### 예제 코드

```js
const userAtomFamily = atomFamily({
  key: "User",
  default: (userId) => ({
    id: userId,
    name: "",
    email: "",
  }),
});

function UserProfile({ userId }) {
  const [user, setUser] = useRecoilState(userAtomFamily(userId));

  const handleNameChange = (event) => {
    setUser({ ...user, name: event.target.value });
  };
  //....
}
```

#### atomFamily 의 effect 옵션

atomFamily의 effects 옵션은 atom의 생명주기를 관리하고 부수 효과를 처리하는 데 사용됩니다. 이를 통해 atom이 생성되거나 파괴될 때 특정 작업을 수행할 수 있습니다.

1. atom 생성 시, 초기화 로직
2. 클린업 로직
3. 값 변경에 대한 부수 효과
4. 비동기 작업
   - atom의 초기값을 비동기로 설정하거나 값 변경 시비동기 작업을 수행

```js
const myAtomFamily = atomFamily({
  key: "myAtomFamily",
  default: null,
  effects: (param) => [
    ({ setSelf, onSet }) => {
      // atom이 생성될 때 실행되는 코드
      // ...

      onSet((newValue) => {
        // atom의 값이 변경될 때마다 실행되는 코드
        // ...
      });

      return () => {
        // atom이 파괴될 때 실행되는 코드
        // ...
      };
    },
  ],
});
```

(Author: 바다)
