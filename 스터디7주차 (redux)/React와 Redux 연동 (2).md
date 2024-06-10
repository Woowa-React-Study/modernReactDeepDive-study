# React와 Redux 연동 (2)

## 컨테이너 컴포넌트와 프레젠테이셔널 컴포넌트 분리

### 컨테이너 컴포넌트

- **상태 관리와 비즈니스 로직**을 담당합니다.
- Redux와 직접적으로 연동됩니다.
- 프레젠테이셔널 컴포넌트에 필요한 데이터와 콜백을 props로 전달합니다.

```js
// DataListContainer.js
import React, { useEffect } from "react";
import { useSelector, useDispatch } from "react-redux";
import { fetchData } from "./actions";
import DataList from "./DataList";

const DataListContainer = () => {
  const dispatch = useDispatch();
  const data = useSelector((state) => state.data);
  const loading = useSelector((state) => state.loading);
  const error = useSelector((state) => state.error);

  useEffect(() => {
    dispatch(fetchData());
  }, [dispatch]);

  return <DataList data={data} loading={loading} error={error} />;
};

export default DataListContainer;
```

### 프레젠테이셔널 컴포넌트

- UI와 관련된 부분만 신경 씁니다.
  - 스타일링과 레이아웃에 집중합니다.
- 데이터나 상태 관리에 대한 로직을 포함하지 않습니다.

```js
// DataList.js
import React from "react";

const DataList = ({ data, loading, error }) => {
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      {data.map((item) => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
};

export default DataList;
```

### 컨테이너 컴포넌트와 프레젠테이셔널 컴포넌트를 분리해야하는 이유

#### ✔️ 관심사의 분리

- UI와 상태 관리 로직을 분리함으로써 코드의 가독성과 유지보수성을 높일 수 있습니다.

#### ✔️ 재사용성

- 프레젠테이셔널 컴포넌트는 특정한 데이터나 상태에 의존하지 않으므로 여러 곳에서 재사용될 수 있습니다.

#### 테스트 용이성

- 프레젠테이셔널 컴포넌트는 단순히 UI만을 담당하므로, 별도의 상태 관리 로직 없이 쉽게 테스트할 수 있습니다.

#### 구조적 명확성

- 애플리케이션의 구조가 더 명확해지고, 각 컴포넌트가 어떤 역할을 담당하는지 쉽게 이해할 수 있습니다.

### 분리 시, 주의해야 할점

#### 명확한 역할 분담

- 프레젠테이셔널 컴포넌트
  - **UI만 담당**
  - 의존성 최소화
  - 가능한 작게, 단일 책임 원칙을 따르도록 설계
- 컨테이너 컴포넌트 :
  - 상태 관리와 비즈니스 로직만 담당
    -> 리덕스 관련 코드는 컨테이너 컴포넌트에만 위치해야 합니다.

Q. 프레젠테이셔널 컴포넌트에 필요한 데이터나 이벤트 핸들러는 어떻게?

A. 데이터,이벤트 핸들러를 props로 받아야 합니다.

### 분리시 , 디렉토리 구조

- 컨테이너 컴포넌트와 프레젠테이셔널 컴포넌트의 폴더 구조를 명확하게 나눕니다.
  - components 폴더: 프레젠테이셔널 컴포넌트
  - containers 폴더 : 컨테이너 컴포넌트

## Redux와 연동된 컴포넌트의 최적화 방법

### useSelector 최적화

#### 선택자 함수 최적화 : shallowEqual

useSelector에서 반환하는 값이 객체,배열인 경우 새로운 참조를 생성하지 않도록 shallowEqual을 사용할 수 있습니다.두 객체의 최상위 속성만을 비교하여 동일한지 여부를 판단합니다.

shallowEqual을 사용하면, 객체/배열의 값이 변경이 없다면 리랜더링이 일어나지 않습니다.

- shallowEqual?
  - 두 객체의 얕은 비교를 진행, shallowEqual을 사용하면 선택된 값의 내용이 동일한 경우 불필요한 리렌더링을 방지할 수 있습니다.

#### shallowEqual 이 객체 비교하는 동작 코드

```javascript
const shallowEqual = (objA, objB) => {
  if (objA === objB) {
    return true;
  }

  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    return false;
  }

  for (let i = 0; i < keysA.length; i++) {
    if (
      !Object.prototype.hasOwnProperty.call(objB, keysA[i]) ||
      objA[keysA[i]] !== objB[keysA[i]]
    ) {
      return false;
    }
  }

  return true;
};
```

### shallowEqual 을 통한 객체 비교 예시 - 동일한 객체

```javascript
const prevObj = {
  name: "Alice",
  age: 25,
  address: {
    city: "Wonderland",
  },
};

const nextObj = {
  name: "Alice",
  age: 25,
  address: {
    city: "Wonderland",
  },
};

const isEqual = shallowEqual(prevObj, nextObj); // true
```

### shallowEqual 이 객체 비교하는 동작

```js
const prevObj = {
  name: "Alice",
  age: 25,
  address: {
    city: "Wonderland",
  },
};

const nextObj = {
  name: "Alice",
  age: 26,
  address: {
    city: "Wonderland",
  },
};

const isEqual = shallowEqual(prevObj, nextObj); // false

const prevObj2 = {
  name: "Alice",
  age: 25,
  address: {
    city: { name: "land" },
  },
};

const nextObj2 = {
  name: "Alice",
  age: 25,
  address: {
    city: { name: "Wonderland" },
  },
};

const isEqual2 = shallowEqual(prevObj2, nextObj2); // false
```

### 그 외

React에서 컴포넌트 최적화하는 방법과 같다.

- React.memo
- useCallback
- useMemo
- React.lazy를 사용해 필요한 시점에 컴포넌트를 로드
