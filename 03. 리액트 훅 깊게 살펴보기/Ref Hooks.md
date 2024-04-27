## Ref Hooks

- `useRef`는 ref를 선언합니다. 여기에는 어떤 값이라도 담을 수 있지만, 대부분 DOM 노드를 담는 데 사용됩니다.
- `useImperativeHandle`을 사용하면 컴포넌트에 노출되는 ref를 커스텀할 수 있습니다. 이는 드물게 사용됩니다.

<br/>

## useRef

### 정의

**렌더링에 필요하지 않은 값을 참조할 수 있는 React Hook이다.**

### **기본 코드**

```jsx
import { useRef } from 'react';

function MyComponent() {
	const ref = useRef(initialValue);
	// ...
```

- 매개변수 initialValue : `ref`객체의 `current` 프로퍼티 초기 설정값이다.
- 반환값 : 단일 프로퍼티를 가진 객체이다.

### 특징

- `useState`와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다는 공통점이 있다.
- 반환값인 객체 내부에 있는 `current`로 값에 접근 또는 변경할 수 있다.
- 그 값이 변하더라도 렌더링을 발생시키지 않는다.
  → 따라 화면에 표시되는 정보를 저장하는 데는 `ref`보다 `state`가 적합하다.

### TroubleShooting

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

```jsx
export default function MyInput({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}
```

위와 같이 컴포넌트에 ref를 전달하고자 하면, 다음과 같은 에러가 발생한다.

> Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?

그럴 땐 `forwardRef`로 감싸면 부모 컴포넌트가 `ref`를 가져올 수 있다.

```jsx
const MyInput = forwardRef(({ value, onChange }, ref) => {
  //
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref} //
    />
  );
});

export default MyInput;
```

<br/>

## forwardRef

### 정의

**forwardRef 를 사용하면 컴포넌트가 `ref.`를 사용하여 부모 컴포넌트의 DOM 노드를 노출할 수 있다.**

### 기본 코드

```jsx
import { forwardRef } from "react";

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

- 매개변수 `render` : 컴포넌트의 렌더링 함수
  - 매개변수 `props` : 부모 컴포넌트가 전달한 `props`
  - 매개변수 `ref` : 부모 컴포넌트가 전달한 `ref 어트리뷰트`로 전달하지 않은 경우 `null`이 된다.
  - 반환값 : JSX에서 렌더링할 수 있는 React 컴포넌트이다.
- 반환값 : JSX에서 렌더링할 수 있는 React 컴포넌트이다. 이 컴포넌트는 `ref` prop도 받을 수 있다.

### 배경

- `ref`를 전달하는 데 있어서 일관성을 제공하고
- `ref`를 전달할 때의 명확성과 안정성을 높이기 위해서

<br/>

## useImperativeHandle

### 정의

**ref로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해주는 React 훅이다.**

### 기본 코드

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... 메서드를 여기에 입력하세요 ...
    };
  }, []);
  // ...
```

`useImperativeHandle(ref, createHandle, dependencies?)`

- 매개변수 `ref` : `forwardRef` 렌더 함수에서 두 번째 인자로 받은 `ref`이다.
- 매개변수 `createHandle` : 인자가 없고 노출하려는 `ref` 핸들을 반환하는 함수이다.
- 매개변수 `dependencies` : `createHandle` 코드 내에서 참조하는 모든 방응형 값을 나열한 목록이다.
- 반환값 : `undefined`이다.

<br/>

## 퀴즈

1. 렌더링에 영향을 미치지 않는 고정된 값을 관리하기 위해서 `useRef`를 사용한다면 `useRef`를 사용하지 않고 그냥 함수 외부에서 값을 선언해서 관리하는 것도 동일한 기능을 수행할 수도 있지 않을까?

2. `useRef`로 생성된 `ref` 객체의 `current` 속성을 변경할 때, 컴포넌트의 리렌더링이 발생하는가?
