# React의 생명주기와 setState

## 도입부 - 오류 발생

```tsx
export default function ModalPortal(props: ModalPortalProps) {
  const { children } = props;
  const [modalRoot, setModalRoot] = useState<HTMLElement | null>(null);

  const createModalRoot = () => {
    if (document.getElementById("modal-root")) return;

    const body = document.body;
    const $modalRoot = document.createElement("div");
    $modalRoot.id = "modal-root";
    body.appendChild($modalRoot);
    setModalRoot($modalRoot);
  };
  createModalRoot();

  /* 오류가 나지 않는 수정본
  useEffect(() => {

 createModalRoot();
  }, [modalRoot]);*/

  return modalRoot ? (
    createPortal(
      <ModalPortalWrapper className="modal-portal">
        {children}
      </ModalPortalWrapper>,
      modalRoot,
      "modal-portal"
    )
  ) : (
    <div>모달이 열릴 장소를 찾을 수 없습니다.</div>
  );
}
```

### Q.

아래의 코드에서 createModalRoot를 useEffect안에 실행하면 오류가 없지만,
useEffect밖에서 실행하면 오류가 나는 이유?

단. modalRoot라는 상태가 아닌 document.getElementId로 찾는 방식을 사용하면 오류가 나지 않는다.

### A.

1. createModalRoot는 컴포넌트의 생명주기 중 **초기화 단계**에서 실행된다.
2. 초기화 단계는 state,props의 초기 상태를 설정(=기존에 코드로 넣어준 초기값을 확인)한다. 초기화 단계에서 setState를 사용해서 **setState로 업데이트/변경하려는 상태가 적용되지 않는다.**
   - 아직 컴포넌트가 마운트되기 전이므로, setState를 호출해도 상태 변경이 즉시 반영되지 않는다.
   - setState는 대기 중인 상태 업데이트로 처리되고, 초기 렌더링에는 영향을 주지 않는다.
3. setState로 상태를 업데이트하려면 **컴포넌트가 마운트된 후에 상태를 업데이트해야 변경된 상태가 적용된다.**

## React의 생명주기

### 1.컴포넌트 초기화 및 props, state 설정

- 초기 상태(state)를 설정하고, 컴포넌트에 필요한 속성(props)을 받아오고 해당 props의 기본값을 설정한다.

  - 컴포넌트가 렌더링될 때 특정 속성이 전달되지 않았을 경우, 기본값으로 사용됩니다.
  - 타입이 옵셔널이 props에서 기본값 넣기

    ```tsx
    const Input({border='1rem',.....}:Props)
    ```

### 2. 컴포넌트 렌더링 (JSX 반환)

- 컴포넌트의 UI를 정의하고 업데이트
- 렌더링 결과를 기반으로 Virtual DOM 업데이트

### 3. 컴포넌트 마운트 완료

- 렌더링 단계에서 생성된 DOM이 실제 DOM에 반영
- useEffect실행

(Author: 바다)
