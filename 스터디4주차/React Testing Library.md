## React Testing Library (RTL)

### TypeScript와 함께 설치하기

`npm install --save-dev @testing-library/react @types/react-dom @types/react`

### 주요 API

1. `render(component, options?)` : React 컴포넌트를 렌더링하고 쿼리 및 유틸리티 함수를 반환합니다.
2. `screen` : 렌더링된 컴포넌트에서 요소를 선택하기 위한 쿼리 함수를 제공하는 객체입니다.
   - 쿼리 함수 : `getByText`, `getByRole`, `getByLabelText` 등 다양한 방식으로 요소를 선택할 수 있는 함수들입니다.
   - 검색 타입 : `getBy`, `queryBy`, `findBy` 등의 접두사로 요소 검색 시 동작 방식을 결정합니다.
   - 검색 변수 : `ByText`, `ByRole` 등의 접미사로 요소 검색의 기준을 지정합니다.
3. `fireEvent` : 이벤트를 발생시키는 함수를 제공합니다.
   - `fireEvent.click(element)`: 클릭 이벤트를 발생시킵니다.
   - `fireEvent.change(element, event)`: 변경 이벤트를 발생시킵니다.
   - `fireEvent.keyDown(element, event)`, `fireEvent.keyUp(element, event)`: 키 이벤트를 발생시킵니다.
   - `fireEvent.submit(element)`: 제출 이벤트를 발생시킵니다.
4. `userEvent` : 실제 사용자 상호작용에 가까운 이벤트를 발생시키는 함수들을 제공합니다.
   - `userEvent.click(element)`: 사용자의 클릭을 시뮬레이션합니다.
   - `userEvent.type(element, text)`: 사용자의 텍스트 입력을 시뮬레이션합니다.
   - `userEvent.tab()`: 탭 키 입력을 시뮬레이션합니다.
5. `cleanup` : 테스트 후 렌더링된 컴포넌트를 언마운트하여 메모리 누수를 방지합니다.
6. `act` : 컴포넌트의 상태 변경을 래핑하여 테스트의 안정성을 높입니다.
7. `renderHook` : 사용자 정의 훅을 독립적으로 테스트할 수 있게 해줍니다.
8. `configure` : React Testing Library의 전역 옵션을 설정합니다.

### 사용 예시

<details>
<summary>접기/펼치기</summary>
<div markdown="1">

```jsx
// React 및 React Testing Library 모듈 import
import React, { useState } from "react";
import {
  render,
  cleanup,
  act,
  renderHook,
  configure,
} from "@testing-library/react";

// React Testing Library의 전역 설정 구성
configure({ reactStrictMode: true });

// useCounter 훅 정의
const useCounter = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount(count + 1);
  return { count, increment };
};

// Counter 컴포넌트 정의
const Counter = () => {
  const { count, increment } = useCounter();
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
    </div>
  );
};

// 각 테스트 케이스 실행 후 정리 작업 수행
afterEach(cleanup);

// Counter 컴포넌트 테스트
test("Counter component works correctly", () => {
  // render 함수를 사용하여 Counter 컴포넌트 렌더링
  const { getByText } = render(<Counter />);

  // getByText를 사용하여 카운트 엘리먼트와 증가 버튼 선택
  const countElement = getByText(/Count: \d+/);
  const incrementButton = getByText("Increment");

  // 초기 카운트 값 확인
  expect(countElement).toHaveTextContent("Count: 0");

  // act를 사용하여 증가 버튼 클릭 시뮬레이션
  act(() => {
    incrementButton.click();
  });

  // 카운트 값이 올바르게 업데이트되었는지 확인
  expect(countElement).toHaveTextContent("Count: 1");
});

// useCounter 훅 테스트
test("useCounter hook works correctly", () => {
  // renderHook을 사용하여 useCounter 훅 렌더링
  const { result } = renderHook(() => useCounter());

  // 초기 카운트 값 확인
  expect(result.current.count).toBe(0);

  // act를 사용하여 increment 함수 호출 시뮬레이션
  act(() => {
    result.current.increment();
  });

  // 카운트 값이 올바르게 업데이트되었는지 확인
  expect(result.current.count).toBe(1);
});
```

</div>
</details>

<br/>

## JEST

### 설치

`npm install --save-dev jest` or `npm install --save-dev ts-jest`

### 주요 API

- `describe(name, fn)`: 관련된 테스트 케이스를 그룹화하는 블록을 정의합니다.
- `test(name, fn)` 또는 `it(name, fn)`: 개별 테스트 케이스를 정의합니다.
- `expect(value)`: 값에 대한 어설션을 수행((기대하는 결과를 확인)하기 위한 객체를 반환합니다.
  - `toBe(value)`: 엄격한 동등성 (===) 검사를 수행합니다.
  - `toEqual(value)`: 깊은 동등성 검사를 수행합니다.
  - `toContain(item)`: 배열이나 문자열에 특정 요소가 포함되어 있는지 검사합니다.
  - `toHaveLength(number)`: 객체의 길이를 검사합니다.
  - `toBeNull()`, `toBeUndefined()`, `toBeDefined()`, `toBeTruthy()`, `toBeFalsy()` 등의 matcher도 사용 가능합니다.
- `beforeEach(fn)`, `afterEach(fn)`: 각 테스트 케이스 실행 전후에 호출되는 함수를 정의합니다.
- `beforeAll(fn)`, `afterAll(fn)`: 모든 테스트 케이스 실행 전후에 한 번씩 호출되는 함수를 정의합니다.
- `jest.fn()`: 모의 함수(Mock Function)를 생성합니다.
- `jest.mock(moduleName)`: 모듈을 모의(Mock)합니다.
- `jest.spyOn(object, methodName)`: 객체의 메서드를 감시(Spy)합니다.

<br/>

## React Testing Library와 다른 테스트 도구와의 관계

### React Testing Library와 Jest

1. Jest와 RTL은 상호 보완적인 도구입니다.
   - Jest는 JavaScript 애플리케이션을 위한 테스팅 프레임워크입니다.
   - RTL은 React 컴포넌트 테스트를 위한 라이브러리입니다.
2. Jest는 테스트 러너로서 `describe`, `it`, `expect` 등의 기능을 제공합니다.
3. `create-react-app`으로 생성한 프로젝트에는 Jest와 RTL이 기본 설치되어 있습니다.
4. Jest는 테스트 러너의 역할을 하지만, React 컴포넌트 테스트에는 RTL 등의 라이브러리가 필요합니다.

### React Testing Library와 Storybook

1. Storybook과 RTL은 상호 보완적인 도구입니다.
   - Storybook은 컴포넌트 개발과 문서화에 중점을 둡니다.
   - RTL은 컴포넌트 테스트에 중점을 둡니다.
2. Storybook에서 RTL을 사용하여 컴포넌트 테스트를 실행할 수 있습니다.
3. Storybook으로 개발한 컴포넌트를 RTL로 테스트하여 품질을 보장할 수 있습니다.
4. Storybook과 RTL을 함께 사용하면 컴포넌트 개발, 문서화, 테스트를 효과적으로 수행할 수 있습니다.

<br/>

## 정리

- Jest는 JavaScript 애플리케이션을 위한 포괄적인 테스트 프레임워크입니다.
- React Testing Library는 React 컴포넌트 테스트에 특화된 라이브러리로, Jest와 함께 사용됩니다.
- **Storybook으로 컴포넌트를 시각적으로 확인하고 문서화하며, Jest와 RTL로 컴포넌트의 동작을 검증하는 것이 좋은 개발 방식입니다.**

<br/>

### 참고자료

- [React Testing Library 공식문서](https://testing-library.com/docs/react-testing-library/intro)

- [React Testing Library Tutorial](https://www.robinwieruch.de/react-testing-library/)

- [Jest 공식문서](https://jestjs.io/docs/getting-started)

- [Storybook 공식문서](https://storybook.js.org/tutorials/intro-to-storybook/react/ko/get-started/)
