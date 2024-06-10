

액션의 정의와 구조
액션 생성자 함수의 작성 방법
액션 타입 상수 정의의 중요성

# Action과 Action Creator

## 1. Action의 정의와 구조
### 1-1. Action의 정의

영어 단어 Action은 행동, 동작이라는 의미다.
그렇다면 Redux의 Action도 **어떤 행동**을 의미하는 것일거다.
-> Redux State에 변화를 주기 위한 행동
즉 Redux Store에 저장된 데이터에 변화를 주기 위한 행동이다.

### 1-2. Action의 구조

Redux Action은 어떻게 생겼을까?
Redux Action은 Javascript 객체 형태로 존재한다.
다만, 일반적인 자바스크립트 객체와 다른 점은 **type**이라는 필드를 **무조건 포함**해야된다는 점이다.

여기서 type은 Redux Action의 type을 의미한다.
type을 통해 어떤 액션인지 나타내는 것이다.



#### 1-2-1. type 필드
실제 Redux Action 객체를 예시를 들어 살펴보자.
```javascript
{
    type: 'ADD_TODO', 
    text: 'Redux 공부하기'
}
```
우리가 흔히 볼 수 있는 일반적인 자바스크립트 객체이다.
+ type
  + Redux Action의 type을 의미
  + 즉 Action의 이름

Redux Action 객체에서 타입 필드를 통해 자신이 어떤 액션인지 나타나게된다.('ADD_TODO'이름을 보고 투두를 추가하는 액션이겠구나 알 수 있음)

이 액션의 type들은 사전에 정의되어있는 것이 아니라 리덕스를 사용하서 개발하는 개발자가 정의하게된다.

#### 1-2-2. type 필드 외의 필드들

+ 위에서 알아본 type 필드 외의 나머지 데이터들은 Action을 처리하는데 필요한 부가 데이터이다. 
+ 보통 payload라고 부르는데 위 예시에서는 payload로 text라는 필드에 "Redux 공부하기"라는 문자열로 실제 투두 리스트에 추가할 아이템의 이름을 넣었다.

+ 위 예시의 Action 객체는 매우 단순하지만 실제 규모가 큰 애플리케이션에서는 Action 객체가 수많은 데이터들을 포함하고 있는 경우가 많다.

### 1-3. Action 요약

> 애플리케이션에서 발생하는 일들을 설명하는 Javascript 객체



</br>


## 2. Action Creator과 작성 방법

### 2-1. Action Creator의 정의

+ Action Creator는 단어 의미 그대로 Action을 생성하는 생성자 역할을 한다.
+ 앞서 알아본것처럼 Action은 자바스크립트 객체 형태로 존재하기때문에 Action Creator는 이 자바스크립트 객체를 생성하는 함수라고 보면되겠다.

```javascript
funcion addTodo(text) {
    return {
        type: 'ADD_TODO',
        text: text
    }
}
```

+ Action Creator 함수에 Action 객체 생성에 필요한 데이터를 파라미터로 넣어서 호출하면 그 결과로 Action 객체가 생성된다.


---
# Action 실습
+ Redux를 사용해 매우 간단한 TODO를 만들어보자.

[TODO 예시 코드](https://github.com/0jenn0/redux-study-todo)
위 링크에서 (.)을 누르시면 웹 에디터로 코드를 보실 수 있습니다