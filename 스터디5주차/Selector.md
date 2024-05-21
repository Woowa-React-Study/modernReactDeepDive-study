b## Recoil Selector

### 1. Selector란?

> Selector는 Atom의 값을 기반으로 파생된 상태를 계산하는 역할을 합니다. Selector는 하나 이상의 Atom이나 다른 Selector를 입력으로 받아 새로운 상태 값을 계산하고 반환합니다. 이를 통해 상태 간의 의존성을 관리하고 파생된 값을 쉽게 계산할 수 있습니다.

#### Selector의 구조

- `selector`` 함수를 사용하여 Selector를 생성
- `key`는 Selector의 고유 식별자로 사용
- `get`` 함수에서 Atom의 값을 가져와 새로운 상태를 계산

</br>

### 2. 상태값 계산

#### 2-1 ) Atom의 입력으로 새로운 상태 값 계산하기

```javascript
import { selector } from "recoil";
import { cartItemCountState } from "./atoms";

export const cartTotalPriceState =
  selector <
  number >
  {
    key: "cartTotalPriceState",
    get: ({ get }) => {
      const count = get(cartItemCountState);
      const itemPrice = 10;
      return count * itemPrice;
    },
  };
```

- react_basecamp의 예시 코드. `cartItemCountState` Atom으로 새로운 상태값을 리턴하고있다.

##### 2-1-1 ) 1개의 Atom으로 새로운 상태 값 계산하기

```javascript
// atom.ts
export const cartItemCountState =
  atom <
  number >
  {
    key: "cartItemCountState",
    default: 0,
  };
```

```javascript
// selector.ts
export const cartTotalPriceState =
  selector <
  number >
  {
    key: "cartTotalPriceState",
    get: ({ get }) => {
      const count = get(cartItemCountState); // atom의 key값과 이름이 동일할 필요 없다.
      const itemPrice = 10;
      return count * itemPrice;
    },
  };
```

- `cartItemCountState`atom으로부터 값을 가져와 `count * itemPrice`라는 새로운 상태 값을 계산

##### 2-1-2 ) n개의 Atom으로 새로운 상태 값 계산하기

- 한개의 atom뿐만 아니라 여러개의 Atom을 가져 올 수도있다. 그 대신 이 경우 selector 내부에 사용된 atom 갯수만큼의 의존성이 생기기때문에 Atom을 한개의 상태만 변경되도 selector는 재실행된다.

- 아래는 장바구니 필터 상태(`cartFilterState`),장바구니 아이템 리스트 상태(`cartItemsState`)로 부터 필터로 필터링된 장바구니 상태(`filteredCartState`)를 만드는 Selector 예시코드이다.

```javascript
// Filter 상태에 대한 타입을 정의
type FilterType = "Show All" | "Show In Stock" | "Show Out of Stock";

// 카트 필터 상태
const cartFilterState =
  atom <
  FilterType >
  {
    key: "cartFilterState",
    default: "Show All",
  };


// 카트 아이템 상태
const cartItemsState = atom<CartItem[]>({
  key: "cartItemsState",
  default: [
    { id: 1, name: "Product 1", price: 10, inStock: true },
    { id: 2, name: "Product 2", price: 20, inStock: false },
    { id: 3, name: "Product 3", price: 30, inStock: true },
    { id: 4, name: "Product 4", price: 40, inStock: true },
    { id: 5, name: "Product 5", price: 50, inStock: false },
  ],
});
```

```javascript
const filteredCartState = selector({
  key: "filteredCartState",
  get: ({ get }) => {
    const filter = get(cartFilterState);
    const items = get(cartItemsState);

    switch (filter) {
      case "Show In Stock":
        return items.filter((item) => item.inStock);
      case "Show Out of Stock":
        return items.filter((item) => !item.inStock);
      default:
        return items;
    }
  },
});
```

#### 2-2 ) Selector의 입력으로 새로운 상태 값 계산하기

```javascript
interface CartItem {
  id: number;
  name: string;
  price: number;
  quantity: number;
}

// 장바구니 아이템 상태
const cartItemsState = atom<CartItem[]>({
  key: "cartItemsState",
  default: [
    { id: 1, name: "Product 1", price: 10, quantity: 2 },
    { id: 2, name: "Product 2", price: 20, quantity: 1 },
    { id: 3, name: "Product 3", price: 30, quantity: 3 },
  ],
});


// 장바구니에 든 아이템들의 총금액과 총 수량을 구하는 Selector
const cartTotalState = selector({
  key: "cartTotalState",
  get: ({ get }) => {
    const cartItems = get(cartItemsState);
    const totalAmount = cartItems.reduce(
      (total, item) => total + item.price * item.quantity,
      0
    );
    const totalQuantity = cartItems.reduce(
      (total, item) => total + item.quantity,
      0
    );

    return {
      totalAmount,
      totalQuantity,
    };
  },
});

// cartTotalState와 cartItems를 같이 반환해주도록하는 Selector
const cartItemsWithTotalState = selector({
  key: "cartItemsWithTotalState",
  get: ({ get }) => {
    const cartItems = get(cartItemsState);
    const { totalAmount, totalQuantity } = get(cartTotalState);

    return {
      cartItems,
      totalAmount,
      totalQuantity,
    };
  },
});

```

```javascript
// selector 사용 예시
function CartComponent() {
  const { cartItems, totalAmount, totalQuantity } = useRecoilValue(cartItemsWithTotalState);

  return (
    {...}
  );
}
```

### Selector로 Selector 만들기의 장점

1. 파생된 상태의 조합: 여러 개의 `selector`에서 계산된 파생 상태를 조합하여 새로운 상태를 만들어야 할 때 유용하다. 예를 들어, `cartItemsState`와 `userState`에서 파생된 상태를 조합하여 사용자별 장바구니 정보를 계산하는 경우가 있다. 이때 `selector`로 `selector`를 만들면 코드의 가독성과 재사용성을 높일 수 있다.

   ```javascript
   const cartItemsState = atom<CartItem[]>({
   key: "cartItemsState",
   default: [
       { id: 1, name: "Product 1", price: 10, quantity: 2 },
       { id: 2, name: "Product 2", price: 20, quantity: 1 },
       { id: 3, name: "Product 3", price: 30, quantity: 3 },
   ],
   });

   const userState = atom<User>({
   key: "userState",
   default: {
       id: 1,
       name: "John Doe",
       email: "john@example.com",
   },
   });

   const cartTotalState = selector({
   key: "cartTotalState",
   get: ({ get }) => {
       const cartItems = get(cartItemsState);
       const totalAmount = cartItems.reduce(
       (total, item) => total + item.price * item.quantity,
       0
       );
       return totalAmount;
   },
   });

   const userCartState = selector({
   key: "userCartState",
   get: ({ get }) => {
       const user = get(userState);
       const totalAmount = get(cartTotalState);
       return {
       user,
       totalAmount,
       };
   },
   });
   ```

2. 상태 간 의존성 관리: `selector`로 `selector`를 만들면 상태 간의 의존성을 명확하게 표현할 수 있습니다. 한 `selector`가 다른 `selector`에 의존하는 경우, 해당 의존성을 `selector`의 입력으로 명시함으로써 코드의 의도를 명확히 전달할 수 있습니다. 이는 코드의 가독성과 유지보수성을 향상시킵니다.

   ```javascript
   const productState =
     atom <
     Product >
     {
       key: "productState",
       default: {
         id: 1,
         name: "Product 1",
         price: 10,
       },
     };

   const cartItemState = selector({
     key: "cartItemState",
     get: ({ get }) => {
       const product = get(productState);
       const quantity = 1;
       return {
         product,
         quantity,
       };
     },
   });

   const cartTotalState = selector({
     key: "cartTotalState",
     get: ({ get }) => {
       const cartItem = get(cartItemState);
       const totalAmount = cartItem.product.price * cartItem.quantity;
       return totalAmount;
     },
   });
   ```

3. 캐싱 및 최적화: `selector`는 내부적으로 캐싱 메커니즘을 사용하여 불필요한 재계산을 방지한다. `selector`로 `selector`를 만들면 중간 결과를 캐싱할 수 있으므로 성능 향상에 도움이 된다. 특히 복잡한 계산이 필요한 경우 `selector`를 조합하여 사용하면 계산 비용을 줄일 수 있다.

   ```javascript
   const productListState = atom<Product[]>({
   key: "productListState",
   default: [
       { id: 1, name: "Product 1", price: 10 },
       { id: 2, name: "Product 2", price: 20 },
       { id: 3, name: "Product 3", price: 30 },
   ],
   });

   const filteredProductListState = selector({
   key: "filteredProductListState",
   get: ({ get }) => {
       const productList = get(productListState);
       const filteredList = productList.filter((product) => product.price > 15);
       return filteredList;
   },
   });

   const productTotalState = selector({
   key: "productTotalState",
   get: ({ get }) => {
       const filteredList = get(filteredProductListState);
       const totalAmount = filteredList.reduce((total, product) => total + product.price, 0);
       return totalAmount;
   },
   });

   ```

위 코드에서 캐싱 및 최적화가 적용되는 부분은 `filteredProductListState` **selector**이다.
이 **selector**는 `productListState`를 입력으로 받아 가격이 **15** 이상인 상품만 필터링한 결과를 반환한다.
캐싱의 동작 과정은 다음과 같다.

처음 `filteredProductListState`가 호출되면, `productListState`의 값을 가져와서 필터링 로직을 수행한다.
필터링된 결과는 `filteredProductListState` 내부에 캐싱된다.
이후에 `filteredProductListState`가 다시 호출되면, **Recoil**은 `productListState`의 값이 변경되었는지 확인한다.

만약 `productListState`의 값이 변경되지 않았다면, 이전에 캐싱된 필터링 결과를 그대로 반환한다.
만약 `productListState`의 값이 변경되었다면, 필터링 로직을 다시 수행하고 결과를 캐싱한다.

> 이러한 캐싱 메커니즘의 장점

- 불필요한 계산 회피: `productListState`의 값이 변경되지 않은 경우, `filteredProductListState`는 이전에 캐싱된 결과를 그대로 반환하므로 불필요한 필터링 계산을 회피할 수 있다.
  성능 향상: 캐싱된 결과를 재사용함으로써 계산 비용을 줄일 수 있다. 특히 대량의 데이터를 다루는 경우 캐싱의 효과가 두드러진다.
- 선택적 업데이트: `productListState`의 값이 변경된 경우에만 filteredProductListState가 다시 계산되므로, 관련된 상태만 선택적으로 업데이트할 수 있다..

  위 코드에서 productTotalState는 filteredProductListState를 입력으로 받아 필터링된 상품의 총 가격을 계산합니다. filteredProductListState의 캐싱된 결과를 사용하므로 productListState의 값이 변경되지 않은 경우 총 가격 계산도 캐시된 값을 그대로 사용할 수 있습니다.

  이렇게 selector로 selector를 만들면 중간 결과를 캐싱할 수 있어 성능 최적화에 도움이 됩니다. 불필요한 계산을 회피하고 캐싱된 값을 재사용함으로써 애플리케이션의 성능을 향상시킬 수 있습니다.

4. 테스트 용이성: `selector`로 `selector`를 만들면 각 `selector`를 독립적으로 테스트할 수 있다. 각 `selector`는 입력과 출력이 명확하게 정의되어 있으므로 단위 테스트를 작성하기 쉽다. 이는 코드의 신뢰성을 높이고 버그를 줄이는 데 도움이 된다.

   ```javascript
   const counterState = atom({
     key: "counterState",
     default: 0,
   });

   const doubledCounterState = selector({
     key: "doubledCounterState",
     get: ({ get }) => {
       const counter = get(counterState);
       return counter * 2;
     },
   });

   // 테스트 코드
   it("doubles the counter value", () => {
     const { result } = renderRecoilHook(useRecoilValue, counterState);
     expect(result.current).toBe(0);

     act(() => {
       setRecoilState(counterState, 5);
     });

     const { result: doubledResult } = renderRecoilHook(
       useRecoilValue,
       doubledCounterState
     );
     expect(doubledResult.current).toBe(10);
   });
   ```

하지만 당연히 selector 조합이 항상 좋은건아님 [Recoil 공식문서 | Selector ](https://recoiljs.org/ko/docs/basic-tutorial/selectors)를 보면

> 우리는 다음 통계를 표시하려 한다.
>
> - todo 항목들의 총개수
> - 완료된 todo 항목들의 총개수
> - 완료되지 않은 todo 항목들의 총개수
> - 완료된 항목의 백분율
>   각 통계에 대해 selector를 만들 수 있지만, 필요한 데이터를 포함하는 객체를 반환하는 selector 하나를 만드는 것이 더 쉬운 방법일 것이다.

이런 부분도 있다. 잘 생각해서 써야할듯
