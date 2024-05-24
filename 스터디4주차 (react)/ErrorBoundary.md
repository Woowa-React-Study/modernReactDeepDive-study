## ErrorBoundary

React의 Error Boundary는 원래 클래스 컴포넌트 적용되는 개념이었다.
하지만 React 16.8 버전 이후 도입된 Hooks와 함께, 함수형 컴포넌트에서도 Error Boundary를 구현할 수 있게 되었다.

### 1. ErrorBoundary란?

React의 Error Boundary는 자식 컴포넌트 트리에서 발생하는 자바스크립트 오류를 캐치하고 오류가 발생했을 때 fallback UI(애플리케이션에서 오류가 발생했을 때 사용자에게 보여주는 대체 UI)를 보여주는 방법을 제공하는 컴포넌트이다. Error Boundary를 사용하면 애플리케이션의 일부분에서 오류가 발생하더라도 전체 애플리케이션이 중단되지 않게 처리할 수 있다.

- Error Boundary 특징
-

### 2. 클래스 컴포넌트에서의 ErrorBoundary

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 오류가 발생했을 때 상태 업데이트
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // 오류 정보를 로깅하거나 서버로 전송할 수 있음
    logErrorToMyService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      // fallback UI 렌더링
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

- `static getDerivedStateFromError()` 또는 `componentDidCatch()` 라이프사이클 메서드 중 하나 이상을 구현해야한다.

> `static getDerivedStateFromError()`,`componentDidCatch()`는 리딥다 2장 3챕터에서 내용 확인 가능 (클래스형 컴포넌트의 생명 주기 메서드)

### 3. 함수형 컴포넌트에서의 ErrorBoundary

- 함수형 컴포넌트에서는 `react-error-boundary`라이브러리를 사용해야한다.

#### 3-1 ) 함수형 컴포넌트에서는 왜 기본 내장이 아니라 라이브러리로 바뀌었는가?

1. 생명주기 메소드의 부재
   **Error Boundary**는 `componentDidCatch`,`static getDerivedStateFromError`와 같은 생명 주기 메소드를 통해 에러를 포착한다. 하지만 함수형 컴포넌트에는 이러한 생명주기 메서드가 없기 때문에 기본적으로 **Error Boundary**를 구현할 수 없다.
2. 훅의 한계
   React 훅은 컴포넌트의 상태 관리와 부수 효과를 다루는데는 강력하지만 아직까지 에러 핸들링 위한 훅은 제공되지 않고 있다. `useState`나 `useEffect`같은 훅으로는 컴포넌트 내에서 발생한 에러를 **직접적**으로 처리할 방법이 없다.

#### 3-2 ) 함수형 컴포넌트에서 `react-error-boundary` 사용하기

```javascript
import React from "react";
import { ErrorBoundary } from "react-error-boundary";

// 에러 발생시 보여줄 Fallback 컴포넌트
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>오류가 발생했습니다:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>다시 시도하기</button>
    </div>
  );
}

// 에러 경계를 사용하는 메인 컴포넌트
function MyComponent() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => {
        // 여기서 애플리케이션의 상태를 재설정하면
        // 오류가 다시 발생하지 않도록 할 수 있습니다
      }}
    >
      <AnotherComponent />
      // 이 컴포넌트에서 오류가 발생하면 ErrorFallback이 표시됩니다
    </ErrorBoundary>
  );
}

export default MyComponent;
```

- ErrorFallback 컴포넌트 : 에러 발생 시 사용자에게 보여질 대체 UI. 에러 메시지 출력, 다시 시도 버튼을 제공한다.
- MyComponent 컴포넌트 : **ErrorBoundary**를 사용하여 `AnotherComponent`를 감싸고 있다. 만약 `AnotherComponent`에서 에러가 발생한다면, `ErrorFallback` 컴포넌트가 렌더링된다.
- onReset 함수 : 사용자가 '다시 시도하기' 버튼을 클릭했을 때 호출되는 함수이다. 이 함수를 통해 에러 이전의 상태로 애플리케이션을 재설정 할 수 있다.

---

## 4. ErrorBoundary의 여러 props들

```javascript
import React, { useState } from "react";
import { ErrorBoundary } from "react-error-boundary";

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>An error occurred: {error.message}</p>
      <button onClick={resetErrorBoundary}>Try Again</button>
    </div>
  );
}

function MyComponent() {
  const [value, setValue] = useState("");
  const [errorKey, setErrorKey] = useState(0);

  const handleErrorReset = () => {
    // 에러 후의 상태를 초기화하거나 추가적인 작업 수행
    console.log("Resetting error state after an error");
    setErrorKey((prevKey) => prevKey + 1);
  };

  const handleError = (error, errorInfo) => {
    // 에러 로깅 또는 추가적인 에러 처리 작업 수행
    console.error("Error caught by Error Boundary: ", error);
    console.error("Error details: ", errorInfo);
  };

  return (
    <div>
      <input
        type="text"
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="Type 'error' to simulate an error"
      />
      <ErrorBoundary
        FallbackComponent={ErrorFallback}
        onReset={handleErrorReset}
        resetKeys={[errorKey]}
        onError={handleError}
        fallbackRender={({ error, resetErrorBoundary }) => (
          <div>
            <p>Custom Fallback Render: {error.message}</p>
            <button onClick={resetErrorBoundary}>Custom Try Again</button>
          </div>
        )}
        fallback={<p>Simple fallback UI when needed</p>}
        fallbackProps={{ extraData: "Extra data can go here" }}
      >
        {value === "error" ? (
          throw new Error("Simulated error")
        ) : (
          <h2>All is well</h2>
        )}
      </ErrorBoundary>
    </div>
  );
}

export default MyComponent;
```

### 4-1 ) FallbackComponent

에러 발생 시 표시될 컴포넌트이다. 이 컴포넌트는 error, resetErrorBoundary 등의 props를 받아 사용자에게 에러 정보를 제공하고, 필요한 조치를 취할 수 있는 인터페이스를 제공한다.

### 4-2 ) onReset

resetKeys에 의해 에러 바운더리가 리셋될 때 호출되는 함수이다. 이를 통해 에러 후의 상태를 초기화하거나 추가적인 작업을 수행할 수 있다.

### 4-3 ) resetKeys

에러 바운더리를 리셋해야 하는 조건을 배열로 지정한다. 이 배열의 값이 변경될 때마다 에러 바운더리가 리셋된다.

### 4-4 ) onError

에러가 발생했을 때 호출되는 콜백 함수이다. 이 함수는 에러 객체(error)와 에러 정보(errorInfo)를 인자로 받는다. 이를 사용하여 에러 로깅이나 추가적인 에러 처리 작업을 할 수 있다.

### 4-5 ) resetErrorBoundary

Fallback 컴포넌트 내에서 호출할 수 있는 함수로, 수동으로 에러 바운더리를 리셋할 때 사용된다.

### 4-6 ) fallbackRender

FallbackComponent 대신 사용할 수 있는 함수로, 렌더링할 JSX 요소를 직접 반환한다. 이 함수는 error, resetErrorBoundary 등의 props를 인자로 받는다.

```javascript
fallbackRender={({error, resetErrorBoundary}) => (
    <div>
        <p>An error occurred: {error.message}</p>
        <button onClick={resetErrorBoundary}>Try again</button>
    </div>
)}

```

### 4-7 ) fallback

FallbackComponent 또는 fallbackRender의 대안으로 사용할 수 있는 React 노드이다. 간단한 에러 처리가 필요할 때 직접 React 요소를 지정할 수 있다.

### 4-8 ) fallbackProps

FallbackComponent에 전달할 추가 props를 지정할 수 있다. 이는 Fallback 컴포넌트가 받을 수 있는 추가적인 데이터나 콜백 함수 등을 제공할 때 사용할 수 있다.
