## 1. Recoil 설치 및 프로젝트 세팅

`npm install recoil`

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import { RecoilRoot } from "recoil";
import App from "./App.tsx";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <RecoilRoot>
      <App />
    </RecoilRoot>
  </React.StrictMode>
);
```

<br/>

## 2. RecoilRoot

### 2-1. 컴포넌트 개념 및 동작 원리

- Recoil 상태를 관리하고 하위 컴포넌트에게 상태를 제공하는 역할입니다.
- Recoil을 사용하려면 이 컴포넌트로 애플리케이션을 감싸야 합니다.
- 내부적으로 React의 컨텍스트를 사용하여 Recoil 상태를 관리합니다.
  이를 통해 Atoms와 Selectors의 상태를 전역적으로 접근할 수 있습니다.
- 중앙 저장소를 초기화하고, 컴포넌트들은 이 저장소를 통해 Atoms와 Selectors의 상태를 읽고 쓸 수 있습니다.
- 컴포넌트가 특정 Atom이나 Selector의 상태를 구독하면, 해당 상태가 변경될 때 자동으로 리렌더링됩니다.

### 2-2. RecoilRoot의 속성

- `initializeState?: (MutableSnapshot => void)`

  - `MutableSnapshot`은 Recoil 상태의 스냅샷
  - 초기 렌더링 시 상태를 설정하는 데 사용됩니다.
  - 이후 상태 변경이나 비동기 초기화를 위해서는 `useSetRecoilState()`나 `useRecoilCallback()`과 같은 훅을 사용해야 합니다.

- 예시
  ```jsx
  function AppRoot() {
    return (
      <RecoilRoot
        initializeState={({ set }) => set(myAtom, "initialized value")}
      >
        <MyComponent />
      </RecoilRoot>
    );
  }
  ```
- `override?: boolean`
  - 이 속성은 기본값이 `true`이며, RecoilRoot가 중첩되어 있을 때만 의미가 있습니다.
  - `override`가 `true`이면, 해당 RecoilRoot는 새로운 Recoil 범위를 생성합니다.
  - `override`가 `false`이면, 해당 RecoilRoot는 자식 컴포넌트를 렌더링하는 것 외에 별다른 기능을 하지 않으며, 자식 컴포넌트들은 가장 가까운 상위 RecoilRoot의 상태에 접근하게 됩니다.

### 2-3. 여러 RecoilRoot 사용

- 예시

  ```jsx
  function AppRoot() {
    return (
      <div>
        <RecoilRoot>
          <Counter />
        </RecoilRoot>
        <RecoilRoot>
          <Counter />
        </RecoilRoot>
      </div>
    );
  }
  ```

<br/>

## 3. Atom

### 3-1. Atom 개념

- Recoil 상태의 가장 작은 단위로, 애플리케이션의 상태를 표현합니다.

  ```jsx
  function atom<T>({
    key: string,
    default: T | Promise<T> | RecoilValue<T>,

    effects_UNSTABLE?: $ReadOnlyArray<AtomEffect<T>>,

    dangerouslyAllowMutability?: boolean,
  }): RecoilState<T>
  ```

  - `key`
    - Atom을 식별하는 고유한 문자열입니다.
  - `default`
    - Atom의 초기값입니다.
    - 기본 값, Promise, 다른 Atom이나 Selector가 올 수 있습니다.
    - `effects_UNSTABLE`
      - Atom에 대한 선택적인 효과 배열입니다. 현재 불안정한 API로, 향후 변경될 수 있습니다.
  - `dangerouslyAllowMutability`
    - 기본적으로 Recoil은 불변성(immutability)을 유지하지만, 이 옵션을 true로 설정하면 상태의 변경을 허용합니다.
      이는 특별한 경우에만 사용해야 합니다.

### 3-2. Atom 사용 시 주의할 점

- Atom의 key는 애플리케이션 전체에서 고유해야 합니다.
- Recoil은 불변성을 유지하기 때문에 상태를 직접 변경하지 않고 set 함수를 사용해야 합니다.
- Atom의 기본값으로 비동기 상태(Promise)를 사용할 수 있긴 하지만, 비동기 함수 호출 시에는 Selector를 사용하는 것이 더 적합합니다.

### 3-3. Atom 예시

```jsx
import { atom, useRecoilState } from "recoil";

export const cartItemCountState =
  atom <
  number >
  {
    key: "cartItemCountState", // Atom의 고유 식별자
    default: 0, // Atom의 초기값
  };

function Counter() {
  const [count, setCount] = useRecoilState(counterState);

  const incrementByOne = () => setCount(count + 1);

  return (
    <div>
      Count: {count}
      <br />
      <button onClick={incrementByOne}>Increment</button>
    </div>
  );
}
```

### 3-4. Atom과 상호작용하기 위해 가장 자주 사용되는 Hooks

- `useRecoilState`: Atom의 값을 읽고 쓰기 위해 사용합니다. 이 훅은 `[state, setState]` 형태로 값을 반환합니다.
- `useRecoilValue`: Atom의 값을 읽기만 할 때 사용합니다.
- `useSetRecoilState`: Atom의 값을 쓰기만 할 때 사용합니다.
- `useResetRecoilState`: Atom을 초기값으로 재설정할 때 사용합니다.
