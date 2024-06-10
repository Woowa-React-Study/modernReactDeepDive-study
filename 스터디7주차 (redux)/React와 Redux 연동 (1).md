## React Redux 라이브러리 소개

Redux 자체는 모든 UI 프레임워크와 함께 사용할 수 있지만, 서로 독립적이다. 그래서 어떠한 UI 프레임워크와 함께 Redux를 사용하는 경우, 일반적으로 UI 바인딩 라이브러리를 사용해서 서로를 연결한다.

Redux와 React를 함께 사용하는 경우 **React용 공식 Redux UI 바인딩 라이브러리인 React Redux**를 사용해야 한다.

React Redux 8.x 버전에서는 React 16.8.3 이상의 버전을 권장한다.

<br/>

## React Redux를 사용해야 하는 이유

### 1. React의 공식 Redux UI 바인딩

React팀에서 React Redux를 직접 유지 관리하기 때문에 두 라이브러리의 모든 API 변경 사항이 최신 상태로 유지하여 예상대로 작동하도록 한다.

### 2. 성능 최적화를 제공

React의 컴포넌트에 대한 업데이트는 해당 컴포넌트 트리의 모든 컴포넌트를 재렌더링하게 한다. 데이터가 변경되지 않은 경우에 재렌더링하는 것은 낭비이다. React Redux는 내부적으로 많은 성능 최적화를 구현하므로 실제로 필요할 때만 컴포넌트가 재렌더링된다.

### 3. 커뮤니티 지원

- Discord channel: [#redux on Reactiflux](https://discord.gg/0ZcbPKXt5bZ6au5t) ([Reactiflux invite link](https://reactiflux.com/))
- Stack Overflow topics: [Redux](https://stackoverflow.com/questions/tagged/redux), [React Redux](https://stackoverflow.com/questions/tagged/redux)
- Reddit: [/r/reactjs](https://www.reddit.com/r/reactjs/), [/r/reduxjs](https://www.reddit.com/r/reduxjs/)
- GitHub issues (bug reports and feature requests): https://github.com/reduxjs/react-redux/issues
- Tutorials, articles, and further resources: [React/Redux Links](https://github.com/markerikson/react-redux-links)

<br/>

## React Redux Quick Start

### 0. 참고한 공식 자료

https://react-redux.js.org/tutorials/quick-start

### 1. Install Redux Toolkit and React Redux

```jsx
npm install @reduxjs/toolkit react-redux
```

> 여기서 갑자기 Redux Toolkit이 등장하여 설치하는 이유
>
> Redux Toolkit은 Redux의 공식 도구 세트로, Redux 로직을 작성할 때의 복잡성과 중복을 줄이기 위해 만들어졌다.
>
> - configureStore, createSlice 등의 함수를 제공하여 반복적인 코드 작성을 줄여준다.
> - Immer 라이브러리를 내장하고 있어 불변성을 유지한다.

### 2. Create a Redux Store

애플리케이션 상태를 관리하는 스토어를 만든다.

```jsx
// app/store.js

import { configureStore } from "@reduxjs/toolkit";

export default configureStore({
  // Redux 스토어 생성
  reducer: {},
});
```

### 3. Provide the Redux Store to React

`<Provider>`를 사용하여 Redux 스토어를 React 애플리케이션에 주입한다. 이 컴포넌트는 애플리케이션 트리에서 Redux 스토어를 사용할 수 있도록 한다.

```jsx
// index.js

import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import store from "./app/store";
import { Provider } from "react-redux";

const root = ReactDOM.createRoot(document.getElementById("root"));

root.render(
  // Redux 스토어를 React에 제공
  <Provider store={store}>
    <App />
  </Provider>
);
```

### 4. Create a Redux State Slice

`createSlice`를 사용하여 문자열 이름, 상태의 초기 상태, 리듀서 함수를 만든다.

```jsx
// features/counter/counterSlice.js

import { createSlice } from "@reduxjs/toolkit";

export const counterSlice = createSlice({
  name: "counter", // 슬라이스 이름
  initialState: {
    // 초기 상태 값
    value: 0,
  },
  reducers: {
    // 상태 변경 리듀서 함수
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

export default counterSlice.reducer; // 리듀서 함수 내보내기
```

### 5. Add Slice Reducers to the Store

이전에 생성한 슬라이스 리듀서를 reducer 객체에 추가하여 스토어에 등록한다.

```jsx
// app/store.js

import { configureStore } from "@reduxjs/toolkit";
import counterReducer from "../features/counter/counterSlice"; // 리듀서 함수 가져오기

export default configureStore({
  reducer: {
    counter: counterReducer,
  },
});
```

### 6. Use Redux State and Actions in React Components

이제 React Redux Hooks를 사용하여 React와 Redux 스토어가 상호작용하도록 한다.

- `useSelector`를 사용하여 Redux 스토어에서 데이터를 읽어온다.
- `useDispatch`를 사용하여 디스패치 함수를 반화하여 액션을 Redux 스토어로 전달한다.

```jsx
// features/counter/Counter.js

import React from "react";
import { useSelector, useDispatch } from "react-redux";
import { decrement, increment } from "./counterSlice";
import styles from "./Counter.module.css";

export function Counter() {
  const selectCount = (state) => state.counter.value;
  const count = useSelector(selectCount);
  const dispatch = useDispatch();

  return (
    <div>
      <div>
        <button aria-label="Increment value" onClick={() => dispatch(increment())}>
          Increment
        </button>
        <span>{count}</span>
        <button aria-label="Decrement value" onClick={() => dispatch(decrement())}>
          Decrement
        </button>
      </div>
    </div>
  );
}
```

> 동작 설명
>
> 증가, 감소 버튼을 클릭할 때마다 액션이 스토어로 전송되고, 해당 리듀서는 액션을 보고 상태를 업데이트한다. 카운터 컴포넌트는 스토어에서 새 상태 값을 보고 재렌더링한다.

<br/>

## React Reudx의 주요 컴포넌트

### 1. Provider

- React 애플리케이션에 Redux 스토어를 제공하는 컴포넌트이다.
- 일반적으로 애플리케이션의 최상위 컴포넌트를 Provider로 감싸서 모든 하위 컴포넌트에서 Redux 스토어에 접근할 수 있도록 한다.

### 2. connet

- React 컴포넌트를 Redux 스토어에 연결하는 함수이지만, 현재는 useSelector와 useDispatch 훅을 사용하는 것이 권장된다.

### 3. useSelector

- Redux 스토어의 상태를 읽어오는 훅이다.
- 선택자(selector) 함수를 인자로 받아 필요한 상태 값을 추출한다.
- 선택자(selector) 함수는 스토어의 상태를 인자로 받고, 컴포넌트에 필요한 데이터를 반환한다.

### 4. useDispatch

- Redux 스토어에 액션을 디스패치하는 훅이다.
- 디스패치(dispatch) 함수를 반환하며, 이를 사용하여 액션을 스토어로 전달한다.

<br/>

## React Redux의 데이터 흐름

1. 컴포넌트에서 useDispatch를 사용하여 액션을 디스패치한다.
2. 디스패치된 액션은 Redux 스토어로 전달된다.
3. 스토어는 리듀서 함수를 호출하여 액션에 따라 새로운 상태를 생성한다.
4. 새로운 상태가 스토어에 저장된다.
5. React Redux는 스토어의 상태 변화를 감지하고, 변경된 상태를 구독하는 컴포넌트에게 알린다.
6. 컴포넌트는 useSelector를 사용하여 업데이트된 상태를 가져와 리렌더링한다.
