# useEffect

- 클래스형 컴포넌트의 생명주기 메서드를 대체하기 위해 만들어진 훅이 아니다.
- **어떤 상태값과 함께 실행되는지가 중요. (상태값=state, props)**
- 렌더링할 때마다 의존성에 있는 값을 보고 이전과 달라진게 있다면 부수 효과 실행
- 컴포넌트가 외부 시스템과 동기화
  ex) window.addEventListener() 가 컴포넌트 리렌더링시에도 구독이 끊기지 않게 다시 붙여줌

#### cleanUp 함수 (선택)

```jsx
useEffect(() => {
  function addMouseEvent() {
    console.log(counter) // 이 함수가 정의될 당시의 counter가 환경으로 저장
  }

  window.addEventListener('click', addMouseEvent)

  // 클린업 함수
  return () => {
	console.1og('클린업함수실행!',counter)
	window.removeEventListener('click',addMouseEvent)
	// 다음 리렌더링 이전에, 위에서 붙인 click 이벤트를 제거 후 리렌더링
  }
}, [counter])
```

### 의존성 배열

1. 의존성 배열이 없는 경우

   - 의존성 비교할 필요 없이 렌더링마다 실행된다.

   <br>

2. 의존성 배열에 빈 배열 ([])을 두는 경우

   - 비교할 의존성이 없다 -> 최초 렌더링 이후 실행되지 않는다.

   <br>

3. 의존성 배열에 원하는 값을 넣는 경우
   - 배열의 값들 중 하나가 변경시 실행
   - **배열의 값은 이전과 얕은 비교를 수행**

# useLayoutEffect

- useEffect와 함수의 시그니처가 동일하다. 즉, 사용예제가 동일하다.
  차이점은 실행 순서.

  1.  리액트가 DOM을 업데이트
  2.  useLayoutEffect 실행
  3.  브라우저에 변경사항을 반영
  4.  useEffect 실행

<br>

- 리액트는 useLayoutEffect의 실행이 종료될 때까지 기다린 후, 화면을 그린다.  
  작업이 길면 컴포넌트가 일시 중지됨. -> 성능 문제
  (= 모든 DOM 변경 후 useLayoutEffect의 콜백 함수 실행이 동기적 발생한다)

<br>

- 활용 예시
  - 특정 요소에 따라 DOM 요소 애니메이션
  - 스크롤 위치 제어 등 화면 반영 전에 하고 싶은 작업

# 퀴즈

1. 다음 코드에서 에러가 나는 이유를 설명하시오

```jsx
function ParentComponent() {
  const [count, setCount] = useState("");

  return (
    <>
      <ChildComponent count={count} handleCount={setCount} />
    </>
  );
}

function ChildComponent(count, handleCount) {
  useEffect(() => {
    handleCount((c) => c + 1);
  }, [count]);

  // ...
}
```

# 참고

- 주의 - effect = 컴포넌트의 사이드 이펙트, 즉 부수효과.  
  -> 컴포넌트 렌더링 후 어떤 부수효과를 일으키니 조심하자.

  <br>

- 의존성 배열이 없는 경우, 즉 매 렌더링마다 실행되면 useEffect없이 쓰는 것과 어떤 차이인가?

  ```jsx
  //1
  function Component() {
    console.log('랜더링됨')
  }

  //2
  function Component() {
    useEffect(() => {
      console.log(‘롄더링됨')
    })
  }
  ```

  1 : 컴포넌트 렌더링 도중 실행

  2 : 클라이언트 사이드에서 실행되는 것 보장(렌더링 후 실행)  
   ex) window 객체 접근

  <br>

- 반복되는 useEffect를 커스텀훅으로 만들기

```jsx
// App.js
import { useState } from "react";
import { useWindowListener } from "./useWindowListener.js";

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useWindowListener("pointermove", (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  });

  return (
    <div
      style={{
        position: "absolute",
        backgroundColor: "pink",
        borderRadius: "50%",
        opacity: 0.6,
        transform: `translate(${position.x}px, ${position.y}px)`,
        pointerEvents: "none",
        left: -20,
        top: -20,
        width: 40,
        height: 40,
      }}
    />
  );
}

// useWindowListener.js
import { useState, useEffect } from "react";

export function useWindowListener(eventType, listener) {
  useEffect(() => {
    window.addEventListener(eventType, listener);
    return () => {
      window.removeEventListener(eventType, listener);
    };
  }, [eventType, listener]);
}
```

### useEffect 안티 패턴

1. 의존성 배열을 빈 배열로 두는 것을 지양하라  
   이는 componentDidMount에 기반한 접근법. 어떤 값의 변경과 부수효과가 별개라는 뜻이다.  
   정말 [ ]가 필요하다면 해당 부수효과를 부모에서 실행되는 것을 고려하라  
   즉, 부모에서 렌더링 시키는 시점을 결정하고 props로 넘겨받아라.

   <br>

2. useEffect의 콜백 함수를 익명이 아닌 기명 함수로 하면 useEffect의 역할을 보기 쉽다
   ex) useEffect( function logActiveUser () { // 어쩌구~~ })

<br>

3. useEffect가 거대하다면 적은 의존성 배열을 가진 여러 useEffect로 나누어봐라

<br>

4. 의존성 배열로 객체(or 함수)를 두는 것을 지양하라 (변수의 원시, 참조 타입 관련)

```jsx
function ChatRoom({ roomId }) {

const options = { // 🚩 이 객체는 재 렌더링 될 때마다 새로 생성됩니다
  roomId: roomId
  // ...
};

useEffect(() => {
  const connection = createConnection(options); // 객체가 Effect 안에서 사용됩니다
  // ...
}, [options]); // 🚩 결과적으로, 의존성이 재 렌더링 때마다 다릅니다
```

```jsx
function ChatRoom({ roomId }) {

useEffect(() => {
  const options = { // 🚩 useEffect 안에서 생성
    roomId: roomId
    // ...
  };
  const connection = createConnection(options);
  // ...
}, [roomId]); // 🚩 props로 받는 문자열에만 의존
```

5. [eslint-react-hooks/exhaustive-deps 를 무시하지 마라](https://ko.react.dev/reference/react/useEffect#my-effect-keeps-re-running-in-an-infinite-cycle)
   useEffect 콜백 함수 내부에서 반응형 값 (props, state)를 사용시, 의존성 배열에 넣어라.  
   (공식문서에서는 린터에게 증명하라는 표현. 리렌더링될 때, 변경되지 않는다는 것을 증명.)

### 자주 발생하는 useEffect 관련 에러

- React가 strict 모드로 실행 시, useEffect가 두번 실행 되는 경우 - cleanUP 필요  
   즉, 설정 -> 정리 -> 설정의 과정이 필요함.
  <br>
- useEffect가 무한루프에 빠지는 경우
  useEffect 안에서 setState를 하고, 이 state가 의존성 배열에 있는 경우임.
  <br>
  문제를 해결하기 전에 : 1. Effect가 외부 시스템(DOM, 네트워크, 서드파티 위젯 등)에 연결되어 있는가? 2. Effect에서 왜 state를 변경했는가? 3. 변경된 state가 외부 시스템과 동기화됐는가? 4. 또는 Effect를 통해 애플리케이션의 데이터 흐름을 관리하려고 하는 것인가?

  -> 외부 시스템이 없다면 useEffect를 제거하고 로직을 단순화하자.

### 불필요한 useEffect 지우기

사례 1)
리스트를 표시하기 전에 필터링하고 싶을 때  
 ❌ : 리스트가 변경시, state 변수를 업데이트 하는 useEffect 작성 -> 렌더링 후 리렌더링
**-> : 상위 컴포넌트에서 필터링한 리스트 보내기**
