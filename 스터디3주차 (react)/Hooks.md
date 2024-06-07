# Hooks가 해결하려는 문제

(class를 안쓰고 함수형 컴포넌트임을 전제)
class형보다 관심사 분리 가능

1. 반복되는 로직을 선언형으로 재사용하고자 한다.
2. 관심사의 로직을 한 곳으로 모으고자 한다.
3. render props, 고차 컴포넌트와 달리 컴포넌트 트리에 불필요한 중첩을 유발하지 않는다.
4. [drawback of mixin](https://legacy.reactjs.org/blog/2016/07/13/mixins-considered-harmful.html#why-mixins-are-broken)을 일으키지 않는다.

-> Hooks API의 최대 장점은 위의 장점을 결합한 custom hook의 존재인듯([해당 링크의 초반부](https://overreacted.io/why-do-hooks-rely-on-call-order/))

### Hooks가 없었다면?

Hooks 이전에는 render props나 고차 컴포넌트와 같이 복잡한 패턴을 사용해서 컴포넌트에서 UI부분이 아닌 로직 부분을 분리하였다.
-> 선언적인 Hooks로 코드 재사용, 로직 분리를 하고자 했다.

함수로 코드 재사용을 할수는 있지만 React의 상태를 함수 내부에서 가질 수 없었다.
(물론 추상화하면 가능하다고는 함)
예를 들어, 클래스 컴포넌트에서 "창 크기를 보고 상태를 업데이트", "시간에 따라 값에 애니메이션 적용"하는 동작을 추출할 순 없다.

Hooks는 한번의 함수 호출을 통해 함수에서 state와 같은 React의 기능을 사용하게 함.

### Hooks는?

Hooks는 상태를 공유하는 방법이 아닌 상태 저장 로직을 공유하는 방법이다
격리된 로컬 상태를 가져온다

클래스 컴포넌트의 state가 저장되는 곳과 같은 방식으로 state가 저장된다.
Hook은 React에만 국한된 개념이 아니다. Vue, 웹 컴포넌트, JavaScript 함수에서도 Hooks API를 구현 가능했다.

## useState 비동기 콜백

하나의 페이지나 컴포넌트 내에도 수많은 상태값이 존재한다.
만약 이 상태 하나하나가 바뀔 때마다 화면을 리렌더링 한다면 문제가 생길수도 있다.

때문에 리액트는 성능의 향상을 위해서 setState를 연속 호출하면 `배치` 처리하여 한 번에 렌더링하도록 하였다. 아무리 많은 setState가 연속적으로 사용되었어도 배치 처리에 의해서 한 번의 렌더링으로 최신 상태를 유지하는 것이다.

> 배치란 React가 너 나은 성능을 위해 여러개의 state 업데이트를 하나의 리렌더링으로 묶는 것을 의미한다. React는 16ms 동안 변경된 상태 값들을 하나로 묶는다. (16ms 단위로 배치를 진행한다.)

- 그럼 useState를 동기적으로 처리하려면?

1. `useEffeect`의 의존성 배열을 이용
2. `setState`의 인자로 함수를 집어넣는 것.

```jsx
const [num, setNum] = useState(1);

// 모두 동일한 num
async function plus() {
  console.log("before", num);
  setNum(num + 1);
  console.log("after", num);
}

async function plus() {
  setNum(num + 1);
  setNum(num + 1);
  setNum(num + 1);
}

// -> 1. useEffeect의 의존성 배열을 이용
useEffect(() => {
  // logic...
  console.log(num);
}, num);

setNum(num + 1);

// -> 2. setState의 인자로 함수를 집어넣는 것.

async function plus() {
  setNum((num) => num + 1);
  setNum((num) => num + 1);
  setNum((num) => num + 1);
}
```

# 이건 왜 Hook으로 만들지 않았을까?

### React API가 지키고자 하는 두가지 속성

- Composition
  Hooks API의 이유. custom 훅이 가능하게하고, 각자의 훅이 충돌이 없다.
- Debugging
  react는 특성상, 디버깅을 위해선 트리를 거슬러 올라가야한다. 이 때, 어디인지 prop이나 state만을 보며 거슬러 올라가면 된다.

예를 들어, useState()는 여러 개를 composition하여 호출해도 충돌하지 않는다.

또 hook간에 값을 전달 가능하다. 이때, 버그가 발생하면 프로젝트의 모든 프로젝트를 볼 필요 없이 발생한 부분에서 prop, state로 연관된 트리만 거슬러 올라가면 된다.
즉, Debuggin에서 이동 경로만 따라가면 된다.

- useBailout() 이라는 훅이 없는 이유
  React.memo() 의 기능처럼 기능을 하는 아래의 훅이 있다고 하자. 이전의 값과 같지 않을때만 값을 반환하는 최적화의 기능을 한다.

```tsx
useBailout((prevColor) => prevColor !== color, color);
```

아래와 같은 custom 훅을 정의하고

```tsx
function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);
  // ⚠️ Not a real API
  useBailout((prevIsOnline) => prevIsOnline !== isOnline, isOnline);
  useEffect(() => {
    const handleStatusChange = (status) => setIsOnline(status.isOnline);
    ChatAPI.subscribe(friendID, handleStatusChange);
    return () => ChatAPI.unsubscribe(friendID, handleStatusChange);
  });

  return isOnline;
}

function useWindowWidth() {
  const [width, setWidth] = useState(window.innerWidth);
  // ⚠️ Not a real API
  useBailout((prevWidth) => prevWidth !== width, width);
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  });
  return width;
}
```

아래와 같이 사용한다 하자. (composition)

```tsx
function ChatThread({ friendID, isTyping }) {
  const width = useWindowWidth();
  const isOnline = useFriendStatus(friendID);
  return (
    <ChatLayout width={width}>
      <FriendStatus isOnline={isOnline} />
      {isTyping && "Typing..."}
    </ChatLayout>
  );
}
```

이때, ChatThread 는 width, isOnline 두 가지 중 어떤 것에 의해 렌더링 될지, 안될지 알기 힘들고, 서로에게 영향을 준다.

Debuggin 측면에서, 어느 부분에서 에러가 나는지 모르기에 두가지 커스텀 훅 모두를 깊게 디버깅해봐야한다.

---

ps) useBailout와 비슷한 useMemo를 제공하지만, 각각 독립된 환경에서 실행될 수 있기에 존재.

[참고](https://overreacted.io/why-isnt-x-a-hook/)

# 왜 React Hooks가 call order에 의존할까?

[원문](https://overreacted.io/why-do-hooks-rely-on-call-order/)

# Hooks 관련 코드 팁

1. 여러 state를 관리하기 위해 useState에 단일 객체를 너무 고집하지 마라.
   다음은 마우스 포인터를 관리하기 위한 state.
   - setState마다 객체를 spread 해야함
   - 객체를 구조 분할해서 사용 안하고 position을 쓰며 코드 피로감을 덜어줌
   - \*\* **관련 로직을 쉽게 커스텀 훅으로 분리 가능**

```jsx
function Box() {
  const [state, setState] = useState({ left: 0, top: 0, width: 100, height: 100 });
  // ...

  setState(state => ({ ...state, left: e.pageX, top: e.pageY }));
}

-> // state를 분리

const [position, setPosition] = useState({ left: 0, top: 0 });
const [size, setSize] = useState({ width: 100, height: 100 });

-> // 분리한 state를 커스텀훅으로

const position = useWindowPosition();
const [size, setSize] = useState({ width: 100, height: 100 });

function useWindowPosition() {  const [position, setPosition] = useState({ left: 0, top: 0 });
  useEffect(() => {
    // ...
  }, []);
```

2. 이전 값을 저장하는 usePrevious 훅

```jsx
function usePrevious(value) {
  const ref = useRef();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```

PS) LMS에 있던 [react 개발자의 코멘트](https://github.com/reactjs/rfcs/pull/68)
내용이 겁나 어려움.. 그나마 건진건 useEffect는 클로저를 활용하기에 메모리를 많이 사용한다는 것.
