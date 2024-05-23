## 1. React Query

### 1.1 등장 배경

전통적인 상태 관리 라이브러리는 클라이언트 상태를 다루는 데는 좋지만, 비동기 또는 서버 상태를 다루는 데는 그다지 좋지 않습니다. 서버 상태는 완전히 다른 특성을 가지고 있기 때문입니다.

- 서버 상태는 원격에 저장되며, 개발자가 제어하거나 소유하지 않을 수 있습니다.
- 데이터를 가져오고 업데이트하려면 비동기 API가 필요합니다.
- 다른 사람에 의해 개발자 모르게 변경될 수 있습니다.
- 주의하지 않으면 애플리케이션에서 "오래된" 상태가 될 수 있습니다.

**따라서 React Query는 Server State를 쉽게 관리할 수 있도록 도와주는 라이브러리입니다.**

### 1.2 React Query의 3가지 주요 컨셉

- **Queries** : 서버에서 데이터를 가져오며, 자동 캐싱 및 중복 제거로 성능을 최적화하는 작업
- **Mutations** : 서버의 데이터를 변경(생성, 수정, 삭제)하고, 상태 추적 및 관리가 용이한 작업
- **Query Invalidation** : 특정 조건에 따라 쿼리 결과를 무효화하고 새 데이터를 가져와 일관성을 유지하는 기능

<br/>

## 2. Query와 useQuery

### 2.1 useQuery란

- 서버에서 데이터를 fetchinng하고 캐시하여 React 컴포넌트에 제공하는 훅입니다.
- Query Key와 Query Function을 인자로 받으며, 반환값으로 쿼리의 상태와 데이터를 제공합니다.

```tsx
import { useQuery } from "@tanstack/react-query";

function App() {
  const info = useQuery({ queryKey: ["todos"], queryFn: fetchTodoList });
}
```

### 2.2 Options : Query Key

- 쿼리를 식별하는 고유한 키입니다.
- 최상위 레벨에서 배열로 제공되어야 합니다. (일관성과 확장성의 이유)

  > 이는 쿼리 키의 가장 바깥쪽 구조가 배열 형태여야 한다는 것을 의미합니다.

  ```tsx
  // 최상위 레벨에서 배열인 key
  ['todos', { status: 'done' }]

  // 최상위 레벨에서 배열이 아닌 key
  'todos'
  { type: 'todo', id: 5 }
  ```

- 단일 문자열부터 많은 문자열과 중첩된 객체까지 다양한 형태로 사용할 수 있습니다.

  ```tsx
  // A list of todos, 가장 간단한 형태
  useQuery({ queryKey: ['todos'], ... })

  // A list of todos that are "done"
  useQuery({ queryKey: ['todos', { type: 'done' }], ... })
  ```

- 결정론적으로 해시되므로 객체 내 키의 순서와 관계없이 동일한 쿼리로 간주됩니다.

  > 쿼리 키 내부의 객체에서 키-값 쌍의 순서가 바뀌어도 항상 동일한 해시 값을 생성한다는 것을 의미합니다. 그렇지만 배열 내부의 순서는 중요합니다.

  - 객체 내부의 순서에 관계없이 동일한 데이터에 대해 항상 같은 쿼리 키를 생성할 수 있으므로, 캐싱을 더 효과적으로 활용할 수 있습니다.

  ```tsx
  // 다음의 세가지 쿼리 키는 모두 동일함
  ["todos", { status: "done", page: 1 }][("todos", { page: 1, status: "done" })][
    ("todos", { page: 1, status: "done", other: undefined })
  ][
    // 다음의 세 가지 쿼리 키는 모두 동일하지 않음
    ("todos", "done", 1)
  ][("todos", 1, "done")][("todos", undefined, 1, "done")];
  ```

- 쿼리 함수가 변수에 의존하는 경우, 해당 변수를 쿼리 키에 포함시켜야 합니다.

  - 이렇게 하면 변수가 변경될 때마다 쿼리가 자동으로 재요청되고
  - 독립적으로 캐시될 수 있습니다.

  ```tsx
  // 각 userId에 대해 별도의 todo 목록이 캐시
  function fetchTodosByUser(userId) {
    return axios.get(`/api/users/${userId}/todos`).then((res) => res.data);
  }

  // UserATodos 컴포넌트는 'userA'의 todo 목록을, UserBTodos 컴포넌트는 'userB'의 todo 목록을 가져옵니다.
  function UserATodos() {
    const { data } = useQuery({
      queryKey: ["todos", "userA"],
      queryFn: () => fetchTodosByUser("userA"),
    });
    // ...
  }

  function UserBTodos() {
    const { data } = useQuery({
      queryKey: ["todos", "userB"],
      queryFn: () => fetchTodosByUser("userB"),
    });
    // ...
  }
  ```

### 2.3 Options : Query Function

- 서버에서 데이터를 fetching하는 함수입니다.
- Promise를 반환해야 하며, 데이터를 resolve하거나 에러를 throw해야 합니다.

### 2.4 Returns

- `data` : 쿼리에서 반환된 데이터입니다.
- `isLoading` : 쿼리의 첫 번재 요청이 진행 중인지 여부를 나타냅니다.
- `isFetching` : 쿼리의 요청이 진행 중인지 여부를 나타냅니다.
- `isError` : 쿼리에서 에러가 발생했는지 여부를 나타냅니다.
- `eror` : 쿼리에서 발생한 에러 객체입니다.
- `refetch` : 쿼리를 수동으로 다시 가져오는 함수입니다.
- status
  - Query는 데이터를 가져오는 과정에서 다음과 같은 상태를 가질 수 있습니다.
  - `idle` : 쿼리가 실행되지 않은 초기 상태
  - `Loading (isPending)` : 데이터를 가져오는 중이며, 아직 결과가 없는 상태
  - `Error (isError)` : 데이터를 가져오는 과정에서 오류가 발생한 상태
  - `Success (isSuccess)` : 데이터를 성공적으로 가져온 상태

### 2.5 그 외 options

- `cacheTime` : 쿼리 결과가 캐시에 유지되는 시간입니다. (기본값 : 5분)
- `staleTime` : 쿼리 결과가 신선하다고 간주되는 시간(ms)으로 이 시간이 지나면 쿼리가 재실행됩니다. (기본값 : 0)
- `refetchOnMount` : 컴포넌트가 마운트될 때 쿼리를 자동으로 리페칭할지 여부를 제어합니다. (기본값: true)

<br/>

## 3. 참고 자료

- [배경과 동기](https://tanstack.com/query/latest/docs/framework/react/overview)
- [useQuery](https://tanstack.com/query/latest/docs/framework/react/reference/useQuery)
