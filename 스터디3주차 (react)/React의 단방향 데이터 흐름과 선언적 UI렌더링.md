# React의 단방향 데이터 흐름과 선언적 UI렌더링

## React 등장 배경

페이스북 개발팀은 자바스크립트 번들 사이즈를 최대한 줄이고 싶었고 여러 시도 끝에 탄생한게 React입니다.

### 당시 React를 도입한 개발자들의 React 장점

- 상태에 따른 UI를 선언적으로 구현할 수 있어서 코드를 좀 더 간결하게 작성할 수 있었고 그로 인해 자바스크립트 코드의 크기가 줄었다.
- 선언적 UI렌더링 덕분에 컴포넌트 트리를 간단하게 작성할 수 있고, 어떤 컴포넌트의 렌더링이 필요한 지 결정할 수 있어서 필요하지 않은 영역에 대한 DOM 변경을 하지 않아서 효율적으로 프로젝트를 마칠 수 있었다.
- 시간의 흐름에 따라 변경되는 데이터를 효율적으로 나타내기 위한 재사용 가능한 컴포넌트를 만들 수 있다.
- 기존에 있는 자바스크립트 코드와 HTML를 알면 손쉽게 리액트 코드를 작성할 수 있었다.

> 공통점: 코드의 간결성, 데이터 흐름 파악 쉬움

## React의 단방향 데이터 흐름

### 양방향 바인딩 구조

- 모델(상태 관리)과 뷰가 서로를 변경할 수 있는 구조
- 단점 :DOM의 변경점을 추적하기 어렵고 관리해야하는 파일 크기가 크다.

```html
<html>
  <!--코드...-->
  <body>
    <input type="text" id="inputField" onInput="updateValue(event)" />
    <p>value:<span id="value"></span></p>
    <script>
      let value = "";
      function updateValue(event) {
        value = event.target.value;
        document.getElementById("value").textContent = value;
      }
    </script>
  </body>
</html>
```

뷰(input 요소) -변경> 모델(value 변수) -변경-> 뷰(span 요소)

### React의 단방향 데이터 흐름

모델의 데이터가 변경되면 새로운 DOM를 생성하고, 이전 DOM과 비교해 변경이 있는 부분만 렌더링한다.

```jsx
import React, { useState } from "react";

const ChildComponent = ({ onIncrement }) => {
  return (
    <div>
      <h2>Child Component</h2>
      <button onClick={onIncrement}>Increment</button>
    </div>
  );
};

const ParentComponent = () => {
  const [count, setCount] = useState(0);

  const handleIncrement = () => {
    setCount(count + 1);
  };

  return (
    <div>
      <h1>Parent Component</h1>
      <p>Count: {count}</p>
      <ChildComponent onIncrement={handleIncrement} />
    </div>
  );
};

export default ParentComponent;
```

## React의 선언적 UI렌더링

### 선언적 UI 렌더링과 명령형 UI렌더링

- 선언적 UI 렌더링은 상태를 기반으로 UI를 렌더링하는 방식을 말합니다.
- 명령형 UI 렌더링은 직접 DOM을 조작하여 UI를 렌더링하는 것을 의미합니다.
  - `document.getElementById('value').textContent = value;`

### React의 선언적 UI 렌더링 예시 코드

```JSX
import React, { useState } from 'react';

const ToggleComponent = () => {
  const [isToggled, setIsToggled] = useState(false);

  const toggleState = () => {
    setIsToggled(!isToggled);
  };

  return (
    <div>
      <button onClick={toggleState}>
      <!--isToggled 상태에 따라 text가 변경됨-->
        {isToggled ? 'ON' : 'OFF'}
      </button>
    </div>
  );
};

export default ToggleComponent;
```

## React를 React답게 사용하기

🧐 React를 어떻게 하면 데이터 단방향, 상태에 따른 UI 선언적 렌더링하는 React답게 사용할 수 있을까?

### 불변성 유지하기

React가 상태 변경을 감지할 수 있도록 불변성을 유지하자

### 순수한 컴포넌트 작성하기

동일한 입력에 대해 동일한 출력을 반환하는 순수한 컴포넌트를 사용한다.
순수한 컴포넌트를 사용하면 상태에 따른 UI렌더링 예측과 테스트가 쉬워진다.

### 함수형 프로그래밍 스타일 채택하기

함수형 컴포넌트로 컴포넌트를 작성하자!
함수형 컴포넌트는 상태관리와 렌더링 로직을 분리하여 코드를 더 명확하고 예측가능하게 만든다.

#### 함수형 컴포넌트

```jsx
import React, { useState, useEffect } from "react";

const UserList = () => {
  const [users, setUsers] = useState([]); // 상태 관리 로직

  useEffect(() => {
    // 상태 관리 로직: 데이터 fetching
    const fetchUsers = async () => {
      const response = await fetch("/api/users");
      const data = await response.json();
      setUsers(data);
    };
    fetchUsers();
  }, []);
  // 렌더링 로직 분리
  const renderList = (user) =>
    users.map((user) => (
      <li key={user.id}>
        <Profile user={user} />
      </li>
    ));
  // 렌더링 로직
  return (
    <div>
      <h1>User List</h1>
      <ul>{users.map(renderList(user))}</ul>
    </div>
  );
};

export default UserList;
```

#### 클래스형 컴포넌트

```jsx
import React from "react";

class UserList extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [],
    };
  }

  componentDidMount() {
    // 상태 관리 로직: 데이터 fetching
    const fetchUsers = async () => {
      const response = await fetch("/api/users");
      const data = await response.json();
      this.setState({ users: data });
    };
    fetchUsers();
  }

  render() {
    // 렌더링 로직
    return (
      <div>
        <h1>User List</h1>
        <ul>
          {this.state.users.map((user) => (
            <li key={user.id}>
              <Profile user={user} />
            </li>
          ))}
        </ul>
      </div>
    );
  }
}

export default UserList;
```

### 리액트 hook 활용하기

함수형 컴포넌트에 리액트에서 제공하는 hook을 사용하면 함수형 컴포넌트에서 상태 관리, 라이플 사이클 메서드를 사용할 수 있다.
또한 코드가 간결해지고 재사용성하기 좋다.

### 효율적인 렌더링과 데이터 흐름을 보장하기 위해 상태 관리를 잘하자

- 상태를 계층적으로 구조화하자
  - 필요한 상태를 식별한다.
  - 서로 관련 있는 상태를 그룹화하고 그렇지 않은 상태를 독립적인 상태로 분리한다.
  - 상태를 관리하는 컴포넌트에서 props를 통해 하위 컴포넌트에게 전달한다.
  - 하위 컴포넌트에서 상태를 업데이트해야하는 경우 상태 업데이트 함수를 props로 전달한다.
- 상태를 적절한 컴포넌트에 배치한다
  - 공유되는 컴포넌트는 상위 컴포넌트에서 관리한다.

### props를 통해 데이터를 전달하자

부모 컴포넌트에서 자식 컴포넌트로 넘겨주는 props를 되도록이면 자식 컴포넌트에서 이를 변경하지 못하는 읽기 전용으로 관리한다면 단방향 데이터 흐름을 유지하기 용이하다. 단, 위에서 말했듯이 자식 컴포넌트에서 상태를 조작해야하는 경우도 있다. 이런 경우에는 상태 업데이트 함수를 pros로 내려준다.

### 이벤트 핸들러 사용하기

사용자 상호작용을 통해 상태를 업데이트하려면 이벤트 핸들러를 사용해야한다.
이벤트 핸들러에서 상태 업데이트 함수를 호출하고, UI를 선언적으로 업데이트한다.

# 질문

### 1. React의 단방향 데이터 흐름에서 상위 컴포넌트의 상태를 하위 컴포넌트에서 변경하려면 어떻게 해야 할까요?

### 2. React의 선언적 UI 렌더링과 명령형 UI 렌더링의 주요 차이점은 무엇인가요?

(Author:바다)
