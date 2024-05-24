# Zustand

## State 생성 및 변경

_(`createStore` 사용 예시 코드)_

```typescript
const createImpl = <T>(createState: StateCreator<T, [], []>) => {
  const api = typeof createState === 'function' ? createStore(createState) : createState;

  const useBoundStore: any = (selector?: any) => useStore(api, selector, equalityFn);

  Object.assign(useBoundStore, api);

  return useBoundStore;
};
```

<br>

> ```typescript
> const createImpl = <T>(createState: StateCreator<T, [], []>) => {
> ```
>
> 상태생성함수(`createState`)를 받아 React 훅을 반환한다.
> <br>`createState`는 '상태생성함수'로 상태의 초기값과 업데이트 로직을 정의한다.

<br>

> ```typescript
> const api = typeof createState === 'function' ? createStore(createState) : createState;
> ```
>
> `createState`가 함수인 경우, 상태 관리 API를 생성한다.
> <br>`createState`가 객체인 경우, 해당 객체를 그대로 api에 할당한다.

<br>

> ```typescript
> const useBoundStore: any = (selector?: any) => useStore(api, selector, equalityFn);
> ```
>
> `useBoundStore`는 `useStore` 훅을 사용하여 상태관리 'API'와 '선택기함수', 'equality' 함수를 받아 상태를 반환한다.
> <br> - `equalityFn` : 상태 변경 감지를 위한 함수 (`Object.is`를 사용하여 비교)

<br>

> ```typescript
> Object.assign(useBoundStore, api);
> ```
>
> `useBoundStore` 함수에 상태관리 API의 메서드를 추가한다.

<br>

### createStore : 스토어 만드는 방법

_(스토어 구현 코드)_

```typescript
const createStoreImpl: CreateStoreImpl = (createState) = {

// local variable
type TState = ReturnType<typeof createState>
type Listener = (state: TState, prevState: TState) = void
let state: TState
const listeners: Set<Listener> = new Set()

// closure 1
const setState: StoreApi<TState>['setState'] = (partial, replace) = {
const nextState = typeof partial 'function' ? (partial as (state: TState) =TState) (state) : partial
if (!0bject.is(nextState, state)) {
const previousState = state
state = replace ?? typeof nextState = 'object' ? (nextState as TState) : Object.assign({}, state, nextState)
listeners.forEach((Listener) = listener(state, previousState))

// closure 2
const getState: StoreApi<TState>['getState'] = () =state

// closure 3
const subscribe: StoreApi<TState>['subscribe'] = (listener) = {
listeners.add(Listener)
return () = listeners.delete(listener)

// closure 4
const destroy: StoreApi<TState>['destroy'] = () = {
Listeners.clear()

// return value
const api = { setState, getState, subscribe, destroy }
state = createState(setState, getState, api)
return api as any
```

<br>

> ```typescript
> const createStoreImpl: CreateStoreImpl = (createState) => {
> ```
>
> `createStoreImpl` : Zustand에 정의된 타입(`CreateStoreImpl`)으로 스토어를 생성하는 함수
> <br> `createState` 함수를 인자로 전달하여 스토어의 초기상태 생성

<br>

> ```typescript
> // local variable
> type TState = ReturnType<typeof createState>;
> type Listener = (state: TState, prevState: TState) => void;
> let state: TState;
> const listeners: Set<Listener> = new Set();
> ```
>
> `TState` : `createState` 함수의 반환 타입
> <br> `Listener` : 상태 변경 시 호출되는 콜백 함수의 타입
> <br> `state` : 현재 스토어의 상태를 저장하는 변수
> <br> `listeners` : 상태 변경시 호출되는 콜백 함수들의 집합

<br>

> ```typescript
> // closure1 : 상태 업데이트
> const setState: StoreApi<TState>['setState'] = (partial, replace) => {
>   const nextState = typeof partial === 'function' ? (partial as (state: TState) => TState)(state) : partial;
>   if (!Object.is(nextState, state)) {
>     const previousState = state;
>     state = replace ?? (typeof nextState === 'object' ? (nextState as TState) : Object.assign({}, state, nextState));
>     listeners.forEach((listener) => listener(state, previousState));
>   }
> };
> ```
>
> `setState` : 스토어의 상태를 업데이트하는 함수
> <br> `partial` : 상태를 부분적으로 업데이트하는 로직
> <br> `replace` : true-상태 오버라이딩, false-상태 병합(`Object.assign`)
>
> 상태 변화가 감지된 경우(`Object.is`) 상태를 업데이트하고,
> <br> listeners에 등록된 모든 콜백 함수를 호출한다.

<br>

> ```typescript
> // closure2 : 상태 반환
> const getState: StoreApi<TState>['getState'] = () => state;
> ```
>
> `getState` : 현재 스토어의 상태를 반환하는 함수

<br>

> ```typescript
> // closure3 : 구독 및 해제
> const subscribe: StoreApi<TState>['subscribe'] = (listener) => {
>   listeners.add(listener);
>   return () => listeners.delete(listener);
> };
> ```
>
> `subscribe` : 상태 변경 시 호출될 콜백 함수를 등록하는 함수
> <br> 반환된 함수 : 등록한 콜백 함수를 제거하는 함수

<br>

> ```typescript
> // closure4 : 모든 구독 해제
> const destroy: StoreApi<TState>['destroy'] = () => {
>   listeners.clear();
> };
> ```
>
> `destroy` : 모든 리스너 함수를 제거하는 함수

<br>

> ```typescript
> const api = { setState, getState, subscribe, destroy };
> state = createState(setState, getState, api);
> return api as any;
> ```
>
> `api` : 4개의 클로저를 병합하고 반환하는 객체
>
> `createState` 함수를 호출하여 초기 상태를 생성
> <br> `api` 객체를 반환

<br>

### createStore : 사용 예시

```typescript
const useBearStore = create<BearState>((set) => ({
  bears: 0,
  increase: (by = set((state = { bears: state.bears + by }))),
}));
```

`create`의 인자로 `createState`(초기상태)를 넘겨준다.
`useBearStore` 커스텀 훅은 bears와 increase를 갖는다.

<br>

## store 변화 감지

```typescript
const createImpl = <T>(createState: StateCreator<T, [], []>) => {
  const api = typeof createState === 'function' ? createStore(createState) : createState;

  const useBoundStore: any = (selector?: any) => useStore(api, selector, equalityFn);

  Object.assign(useBoundStore, api);

  return useBoundStore;
};
```

컴포넌트는 store의 변화를 어떻게 감지할까?<br>
`useBoundStore` 함수 내부에서 호출하는 `useStore` 함수가 담당한다.

<br>

```typescript
function useStore<TState, StateSlice>(
  api: WithReact<StoreApi<TState>>,
  selector: (state: TState) => StateSlice = api.getState as any,
  equalityFn?: (a: StateSlice, b: StateSlice) = boolean
) {
  const slice = useSyncExternalStoreWithSelector(
    api.subscribe,
    api.getState,
    api.getServerState || api.getState,
    selector,
    equalityFn
  )
  useDebugValue(slice)
  return slice
}
```

`useStore` 함수는 `api`(클로저)를 파라미터로 받아서 `useSyncExternalStoreWithSelector` 함수를 호출한다.<br>
이 함수는 리액트 내장함수로, 컴포넌트가 `external store`를 구독할 수 있도록 한다.

> external store 란?
> <br> React에서 제공하는 'prop', 'useState', 'useReducer', 'Context API' 등을 제외한 것
> <br> 'Redux', 'Zustand'와 같은 서드파티 상태 관리 라이브러리와 전역 변수 등을 의미

<br>

### subscribeToStore

_(useSyncExternalStoreWithSelector 구현 중 **'컴포넌트가 store의 변화를 감지하는 부분'**)_

```typescript
function checkIfSnapshotChanged<T>(inst: StoreInstance<T>): boolean {
  const latestGetSnapshot = inst.getSnapshot; // createStore에서 생성한 getState 클로저
  const prevValue = inst.value; // 업데이트 전 store
  try {
    const nextValue = latestGetSnapshot();
    return !is(prevValue, nextValue); // 값이 동일한지 비교
  } catch (error) {
    return true;
  }
}

function subscribeToStore<T>(
  fiber: Fiber,
  inst: StoreInstance<T>,
  subscribe: (() = void) = () =void
): any {
  const handleStoreChange = () => {
    if (checkIfSnapshotChanged(inst)) { // store의 데이터가 바뀌었는지 확인한다
      forceStoreRerender(fiber); // 컴포넌트 랜더링
    }
  };
  return subscribe(handleStoreChange); // createStore에서 생성한 subscribe 클로저
}
```

이 코드는 React 컴포넌트와 store의 변화를 감지하고 리렌더링 시키는 코드이다.

- `subscribeToStore` : `subscribe`를 통해 `handleStoreChange` 함수를 콜백 함수로 등록
- `handleStoreChange` : `checkIfSnapshotChanged`를 통해 상태 변화를 감지하고, 리렌더링 함수 호출
- `checkIfSnapshotChanged` : `getState`를 통해 상태 변화 여부를 반환

우리가 상태변화를 감지하기 위하여 주로 사용하는 `useState`, `useEffect` 대신에, 이 복잡한 함수를 사용하는 이유는 무엇일까?

<br>

### Tearing 문제

Tearing는 React18 부터 추가된 `Concurrency` 기능을 external store와 함께 사용할 때 발생할 수 있는 문제이다.

이전 버전의 React는 한 번 렌더링이 시작되면 멈출 수 없었다.<br>
=> 모든 컴포넌트가 일관된 State 값을 갖는다.

React18에서는 렌더링 과정을 멈추고, 다른 작업을 우선할 수 있는 기능이 추가되었다.<br>
=> 각 컴포넌트가 동일한 State에 대해 서로 다른 값을 출력할 수 있다.

이 문제를 해결하기 위하여 리액트는 `useSyncExternalStore` 함수를 제공한다.

![tearing 문제](https://www.nextree.io/content/images/size/w1000/2023/06/09-tearing-1.png)
