# useEffect

- 클래스형 컴포넌트의 생명주기 메서드를 대체하기 위해 만들어진 훅이 아니다. 어떤 상태값과 함께 실행되는지가 중요. (상태값=state, props)
- 렌더링할 때마다 의존성에 있는 값을 보고 이전과 달라진게 있다면 부수 효과 실행

#### cleanUp 함수

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

### useEffect 안티 패턴

1. 의존성 배열을 빈 배열로 두는 것을 지양하라  
   이는 componentDidMount에 기반한 접근법. 어떤 값의 변경과 부수효과가 별개라는 뜻이다. 정말 []가 필요하다면 해당 부수효과를 부모에서 실행되는 것을 고려하라  
   즉, 부모에서 렌더링 시키는 시점을 결정하고 props로 넘겨받아라.

   <br>

2. useEffect의 콜백 함수를 익명이 아닌 기명 함수로 (useEffect의 역할을 보기 쉽게)
   ex) useEffect( function logActiveUser () { // 어쩌구~~ })

   <br>

3. useEffect가 거대하다면 적은 의존성 배열을 가진 여러 useEffec로

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

  1 : 클라이언트 사이드에서 실행되는 것 보장(렌더링 후 실행)  
   ex) window 객체 접근

  2 : 컴포넌트 렌더링 도중 실행
