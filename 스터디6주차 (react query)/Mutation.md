# Mutation

## Mutation의 개념과 사용 목적

### Mutation?

Mutation은 **서버의 데이터**를 변경하는 작업을 의미한다.

### 사용

서버의 데이터를 생성, 업데이트,삭제할 때 사용한다.

#### usQuery vs useMutation

| api         | 목적                                             |
| ----------- | ------------------------------------------------ |
| useQuery    | 서버에서 데이터를 가져올 때 사용 (**읽기 작업**) |
| useMutation | 서버에 데이터를 보낼때 사용 (**쓰기 작업**)      |

## Mutation Key와 Mutation Function

<details>
  <summary>🔍 useMutation option ,return values 보기</summary>
  <div markdown="1">

```jsx
const {
  data, // mutationFn의 반환값
  error, // Mutation 중 발생한 에러
  isError, // Mutation이 에러 상태인지 여부
  isIdle, // Mutation이 아직 실행되지 않은 상태인지 여부
  isPending, //Mutation이 현재 진행 중인 상태인지 여부
  isPaused, //Mutation이 일시 중지된 상태인지 여부
  isSuccess, // Mutation이 성공적으로 완료된 상태인지 여부
  failureCount, //Mutation 실패 횟수
  failureReason, //가장 최근 Mutation 실패 이유
  mutate, //Mutation을 트리거하는 함수
  mutateAsync, //Mutation을 트리거하고 Promise를 반환하는 함수
  reset, // Mutation 상태를 초기 상태로 재설정하는 함수
  status, // Mutation의 현재 상태를 나타냅니다 ('idle', 'loading', 'success', 'error').
  submittedAt, //Mutation이 제출된 시간
  variables, // Mutation 함수에 전달된 변수
} = useMutation({
  mutationFn, //Mutation 함수
  gcTime, // 가비지 컬렉션 시간을 지정 (단위:ms)
  meta, // Mutation에 대한 추가 메타 정보를 지정
  mutationKey, //Mutation 키를 지정
  networkMode, //네트워크 모드를 지정합니다 ('online', 'always', 'offlineFirst').
  onError, // Mutation 에러 콜백 함수
  onMutate, // Mutation 시작 전 콜백 함수
  onSettled, //Mutation 완료 후 (성공 또는 에러) 콜백 함수
  onSuccess, //Mutation 성공 콜백 함수
  retry, //실패 시 Mutation 재시도 설정을 지정
  retryDelay, // 재시도 간격을 지정
  scope, // Mutation의 범위를 지정
  throwOnError, // 에러 시 예외를 던질지 여부를 지정
});

mutate(variables, {
  onError,
  onSettled,
  onSuccess,
});
```

  </div>
</details>

### mutate 와 mutateAsync

mutate와 mutateAsync는 useMutation의 옵션으로 전달된 mutationFn을 실행하는 함수이다.

| fn          | 실행 시점  |
| ----------- | ---------- |
| mutate      | 즉시, 동기 |
| mutateAsync | 비동기     |

```js
const mutation = useMutation((newTodo) => axios.post("/todos", newTodo));
//mutate
const handleAddTodoByMutate = () => {
  mutation.mutate({ id: new Date(), title: "New Todo" });
  console.log("Mutation started");

  //mutateAsync
  const handleAddTodoByMutateAsync = async () => {
    try {
      await mutation.mutateAsync({ id: new Date(), title: "New Todo" });
      console.log("Mutation completed successfully");
    } catch (error) {
      console.error("Mutation failed", error);
    }
  };
};
```

### meta option

Mutation에 관련된 메타데이터로, 디버깅,로깅등의 목적에서 사용될 수 있다.

```jsx
const mutation = useMutation(addTodo, {
  meta: {
    user: {
      id: 123,
      name: "John Doe",
    },
    purpose: "Add a new todo item",
  },
  onSuccess: (data, variables, context) => {
    console.log("Mutation succeeded for user:", context.meta.user);
    console.log("Mutation purpose:", context.meta.purpose);
  },
});
```

### scope option

기본적으로 Mutation 키는 전역적으로 관리되며, 애플리케이션의 어느 곳에서나 해당 키를 사용하여 Mutation을 호출할 수 있습니다.
하지만 **scope 옵션을 사용하면 Mutation 키를 특정 범위 내에서만 사용할 수 있도록 제한**할 수 있다.

#### 언제 사용해야 유용할까?

**1. 컴포넌트 단위의 Mutation 관리**
해당 컴포넌트가 언마운트되면 자동으로 Mutation 상태가 정리됩니다.
```jsx
const TodoList = () => {
  const mutation = useMutation(addTodo, {
    onSuccess: () => {
      queryClient.invalidateQueries({ scope: 'TodoList' });
    },
    scope: 'TodoList',
  });

  // ...
};
```
**2. 중첩된 Mutation 키 관리**
각 범위에서 독립적으로 Mutation을 관리할 수 있습니다.
```jsx
const UserProfile = () => {
  const mutation = useMutation(updateUser, {
    onSuccess: () => {
      queryClient.invalidateQueries({ scope: 'UserProfile' });
    },
    scope: 'UserProfile',
  });

  // ...
};

const UserSettings = () => {
  const mutation = useMutation(updateUser, {
    onSuccess: () => {
      queryClient.invalidateQueries({ scope: 'UserSettings' });
    },
    scope: 'UserSettings',
  });

  // ...
};
```
## Mutation의 상태(Loading, Error, Success) 핸들링

```jsx
import React from "react";
import { useMutation } from "react-query";
import axios from "axios";

const addTodo = async (newTodo) => {
  const response = await axios.post("/api/todos", newTodo);
  return response.data;
};

const TodoForm = () => {
  const [title, setTitle] = React.useState("");

  const mutation = useMutation(addTodo, {
    // 성공 시, 타이틀 초기화
    onSuccess: () => {
      setTitle("");
    },
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    mutation.mutate({ title });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Enter a new todo"
      />
      <button type="submit" disabled={mutation.isLoading}>
        {mutation.isLoading ? "Adding..." : "Add Todo"}
      </button>
      {mutation.isSuccess && <div>Todo added successfully!</div>}
      {/*에러 핸들링*/}
      {mutation.isError && (
        <div>An error occurred: {mutation.error.message}</div>
      )}
    </form>
  );
};
```

## useMutation 훅을 사용한 데이터 변경

```jsx
import { useMutation, useQueryClient } from "react-query";
import axios from "axios";

const addToCart = async (productId) => {
  const response = await axios.post("/api/cart", { productId });
  return response.data;
};

const ProductItem = ({ product }) => {
  const queryClient = useQueryClient();

  const mutation = useMutation(addToCart, {
    onSuccess: (data) => {
      // 장바구미 목록을 나태내는 쿼리를 무효화해서, 장바구니 목록 조회 시 최신 데이터를 가져오게 함
      queryClient.invalidateQueries("cartItems");
      console.log("상품이 장바구니에 추가되었습니다:", data);
    },
    onError: (error) => {
      console.error("장바구니 추가 중 에러 발생:", error);
    },
  });

  const handleAddToCart = () => {
    mutation.mutate(product.id);
  };

  return (
    <div>
      <h3>{product.name}</h3>
      <button onClick={handleAddToCart}>장바구니에 추가</button>
    </div>
  );
};
```

### ✔️ mutation 시 주의 또는 같이 사용하면 좋을 것들

#### 1. 낙관적 업데이트 주의

<span a="#networkMode">networkMode</span>와 onMutate 콜백을 사용하여 서버 응답을 기다리지 않고 즉시 UI를 업데이트할 수 있다. 이는 사용자 경험을 향상 시키지만, 서버의 응답이 실패할 경우 UI를 원래 상태로 롤백해야한다.

#### 2. 서버 상태와의 동기화

뮤테이션 후에는 invalidateQueries나 refetchQueries를 사용하여 관련 쿼리를 무효화하거나 다시 가져와서 서버 상태와 동기화를 유지할 수 있다.

##### useQueryClient

QueryClient 인스턴스에 접근할 수 있는 훅이다. QueryClient는 React Query의 핵심 개념 중 하나로, 캐시 관리, 쿼리 및 뮤테이션 디스패칭, 그리고 전역 설정을 담당한다.

- invalidateQueries : 지정한 쿼리 무표화
- refetchQueries : 지정한 쿼리를 강제로 다시 가져옴

<details>
  <summary>useQuery와 QueryClient</summary>
  <div markdown="1">

```jsx
import { QueryClient, QueryClientProvider } from "react-query";

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      {/* 하위 컴포넌트 */}
    </QueryClientProvider>
  );
}
```

useQuery는 key를 사용해 QueryClientProvider를 통해 제공된 QueryClient 인스턴스에 접근하여 동작합니다.

  </div>
</details>

#### 3. 처리 : onError

#### 4. 로딩 처리

#### 5. react-query devtools 사용

- 쿼리와 뮤테이션의 상태를 모니터링하고 디버깅할 수 있다.
- ReactQueryDevtools 컴포넌트를 애플리케이션에 추가하여 사용할 수 있습니다.

##### 사용 방법

**설치**

```dash
npm install -D react-query-devtools
```

**컴포넌트 추가**

```jsx
import { ReactQueryDevtools } from "react-query/devtools";

function App() {
  return (
    <div>
      {/* 애플리케이션의 다른 컴포넌트들 */}
      <ReactQueryDevtools initialIsOpen={false} />
    </div>
  );
}
```

**사용 예시**
[사용 예시 블로그 바로가기](https://velog.io/@juanito_y247/React-Query-Devtools)

## Optimistic Update와 Rollback

### <div id="networkMode">networkMode</div> option 과 낙관적 업데이트

useMutation의 networkMode 옵션은 Mutation이 실행될 때 네트워크 요청을 어떻게 처리할 것인지를 지정한다.

- online(기본값) : 네트워크 연결되어 있을때만 Mutation 실행
- always : 항상 Mutation 실행 (실패 시 error 상태로 변경)
- offlineFirst : 오프라인에서 먼저 Mutation 실행하고 (실패 시 error 상태로 변경되지 않음) 네트워크 연결되면 서버와 동기화

#### Q. 🤔 서버로 요청을 보내는 거면 네트워크가 연결되어야하는 거 야닌가?

A. networkMode 옵션이 필요한 **이유는 오프라인 환경에서의 사용자 경험을 개선하고, 애플리케이션의 응답성을 향상**시키기 위해서입니다.

예시:

- `offlineFirst`:

  - 오프라인 환경에서도 사용자가 데이터를 변경 시, 로컬에 변경 사항을 저장하고 네트워크가 다시 연결되면 오프라인에서 수행된 Mutation들을 서버로 전송하여 동기화 한다.

- `always` :
  - 네트워크 연결 여부와 관계 없이 Mutation을 실행해 사용자가 즉시 응답을 받을 수 있고, 네트워크가 연결되면 서버와 동기화 한다.

##### ↘️ 😱 낙관적 업데이트???!!!!!

### 낙관적 업데이트시 롤백하는 방법

#### `onError + useQueryClient` 를 이용하자!!

```jsx
const queryClient = useQueryClient();

const mutation = useMutation(updateTodo, {
  // mutate 실행 전에 낙관적 업데이트 수행, todos쿼리 상태 변경
  onMutate: async (newTodo) => {
    // 현재 쿼리 취소
    // todos:  useQuery의 key
    await queryClient.cancelQueries(["todos", newTodo.id]);
    // 이전 상태를 가져옴
    const previousTodo = queryClient.getQueryData(["todos", newTodo.id]);
    // 새로운 상태로 쿼리 데이터를 업데이트
    queryClient.setQueryData(["todos", newTodo.id], newTodo);
    // 롤백을 위해 이전 상태를 반환
    return { previousTodo };
  },
  onError: (error, newTodo, context) => {
    // 에러 발생 시 롤백 처리 (이전 상태로 복구)
    // context: onMutate 콜백에서 반환한 값
    queryClient.setQueryData(["todos", newTodo.id], context.previousTodo);
  },
  onSettled: () => {
    // 뮤테이션 완료 후 쿼리 무효화
    // 해당 쿼리를 사용하는 곳에서 서버에서 변경된 데이터를 가져오게 함
    queryClient.invalidateQueries(["todos", todo.id]);
  },
});
```
