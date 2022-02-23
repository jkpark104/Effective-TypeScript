# ~2장

## 들어가며

이펙티브 타입스크립트의 Item 11~20까지의 내용을 정리했습니다.

## Item 11 잉여 속성 체크의 한계 인지하기

> 잉여 속성 체크란?
> 타입이 명시된 변수에 객체 리터럴을 할당할 때 타입의 속성이 있는지체크한다. 이뿐만 아니라 타입에 '지정하지 않은 속성을 사용한건 아닌지'도 체크한다.
> 이 지정된 것 이외의 속성에 대해 체크하는 것이 잉여 속성 체크다.

### 잉여 속성 체크 적용되는 경우

- 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때
- 선택적 속성만 가지는 weak 타입에도 비슷한 체크가 동작

### 잉여 속성 체크가 적용되지 않는 경우

- 객체 리터럴이 아닌 경우
- 단언문인 경우 (선언문x)

```js
// Options에 선택 프로퍼티로 darkMode가 있다.
// 아래 예제달은 darkmode로 오타가 있는 프로퍼티를 입력했다.
interface Options {
  title: string;
  darkMode?: boolean;
}

// 잉여 속성 체크 적용
// 타입 오류 : Options 형식에 darkmode가 없습니다.
const o: Options = { darkmode: true, title:'Ski Free'};

// 잉여 속성 체크 적용 안됨
// 정상
const intermediate = { darkmode: true, title:'Ski Free'};
const o2: Options = intermediate;

const o3 = { darkmode: true, title:'Ski Free'} as Options;

```

### 주의사항

- 일반적인 구조적 할당 가능성 체크와 역할이 다르다.
- 임시 변수를 도입하면 잉여 속성 체크를 건너뛸 수 있다는 점을 기억하자.

## Item 12 함수 표현식에 타입 적용하기

- ts에서는 함수 표현식을 사용하는 것이 좋다. 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하며 다른 함수 표현식에 재사용할 수 있기 때문이다.
- 쉽게 말해 매개변수와 반환값에 일일이 타입을 지정하지 말고, 함수 표현식 전체 타입을 정의하여 재사용하자.

```js
type DiceRoolFn = (sides: number) => number;
const rollDice: DiceRoolFn = sides => {
  /*...*/
};
```

## Item 13 타입과 인터페이스의 차이점 알기

> ts에는 named type을 정의하는 두 가지 방법 `type`, `interface`이 있다.

```js
type State {
  name: string;
  [key: string]: string;
}

interface State{
  name: string;
  [key: string]: string;
}
```

### 비슷한 점

- 인덱스 시그니처 사용 가능
- 함수 타입 사용 가능
- 타입 확장 가능(인터페이스는 extends, 타입은 유니온과 & 등)

### 인터페이스가 가지는 다른 점

- 확장 시 인터페이스는 유니온 사용 불가
- 인터페이스는 튜플과 비슷하게는 사용할 수 있지만 concat 같은 메서드 사용 불가
- 보강(augment) 가능 => 선언 병합(declaration merging)

### 뭘 언제 쓰나

- 타입: 복잡한 타입
- 인터페이스: 간단한 객체 타입, 선언 병합을 사용한 보강의 가능성이 있는 타입(API 등)
- 일관되게 사용하는 것이 있다면 따르기

### 주의사항

- 타입과 인터페이스의 차이를 분명하게 알고, 같은 상황에는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야 한다.
- 내부적으로 사용되는 타입에 선언 병합이 발생하는 것은 잘못된 설계다. => 타입 사용하기

## Item 14 타입 연산과 제너릭 사용으로 반복 줄이기

> DRY (Don't repeat yourself) 원칙을 타입에도 적용하자

### 이름 붙이기, extends 로 인터페이스 확장하기

```ts
type HTTPFunction = (url: string, opts: Options) => Promise<Response>;
function get: HTTPFunction = (url, opts) => {/*...*/};
function post: HTTPFunction = (url, opts) => {/*...*/};
```

- 전체 앱의 상태 State와 일부분인 TopNavState 경우
  => State의 부분 집합으로 TopNavState를 정의하는 것이 전체 앱 상태를 하나의 인터페이스로 유지할 수 있게 한다.

```ts
interface State{
  userId: string;
  pageTitle: string;
  recentFiles: string[];
  pageContents: string;
}

// 이제 State의 타입이 바뀌면 그것을 공유하는 하위 타입에도 적용이 된다.
type TopNavState{
  userId: State['userId'];
  pageTitle: State['pageTitle'];
  recentFiles: State['recentFiles'];
}
```

### 타입간 매핑 활용하기

- keyof, typeof, 인덱싱, 매핑된 타입들

```ts
// 매핑된 타입을 사용해 중복을 더 줄이자
type TopNavState = {
  [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k];
};
```

### 제너릭 타입 활용하기

### 표준 lib의 제너릭 타입에 익숙해지기 (보완 필요)

> type Pick<T,K> = { [k in K]: T[k]};

- Pick, Partial, ReturnType

```ts
// 중복을 더 줄인다면!!
type TopNavState = Pick<State, 'userId' | 'pageTitle' | 'recentFiles'>;
```

## Item 15 동적 데이터에 인덱스 시그니처 사용하기

> 인덱스 시그니처 [property: string]: string

### 인덱스 시그니처 특징 및 단점

- 키의 이름: 키의 위치만 표시하는 용도로 타입 체커에서는 사용하지 않는다.
- 키의 타입: string이나 number 또는 symbol의 조합이어야 하지만 보통은 string으로 사용(item16)
- 값의 타입: 어떤 것이든 된다.

```ts
type Rocket = { [property: string]: string };
const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
}; // 정상
```

모든 키 이름과 갯수를 허용하며 키마다 다른 타입을 지정할 수 없다. 자동완성도 안된다.

### 언제 사용하나?

- `미리 알 수 없는 데이터`에 대해서만 인덱스 시그니처 사용

가능하다면 인터페이스, Record, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

### 어떤 타입에 사용가능한 필드가 정해진 경우

#### Record 제너릭 타입

```ts
type Vec3D = Record<'x' | 'y' | 'z', number>;
//type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }
```

#### 매핑된 타입 사용 : 키마다 별도 타입 사용 가능

```ts
type Vec3D = { [k in 'x' | 'y' | 'z']: number };
// Type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }

type ABC = { [k in 'a' | 'b' | 'c']: k extends 'b' ? string : number };
// Type ABC = {
//   a: number;
//   b: string;
//   c: number;
// }
```

## Item 16 number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기 (보완 필요)

- for-in은 for-of 또는 for루프에 비해 몇 배나 느리다.

* 배열은 객체이므로 키는 숫자가 아니라 문자열이다. 인덱스 시그니처로 사용된 number 타입은 버그를 잡기 위한 순수 타입스크립트 코드다.
* 인덱스 시그니처에 number를 사용하기보다 Array나 튜플, 또는 ArrayLike 타입을 사용하는 것이 좋다.

## Item 17 변경 관련된 오류 방지를 위해 readonly 사용하기

### readonly 속성

`arr: readonly number[]`

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
- length를 읽을 수 있지만 바꿀 수 없다.
- pop, push 등의 메서드를 호출할 수 없다.

- ts는 매개변수가 함수 내에서 변경이 일어나는지 체크
- 호출하는 쪽에서 함수가 매개변수를 변경하지 않는다는 보장을 받게 된다.
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있다.

### 정리

- 함수가 매개변수를 변경하지 않는다면 readonly로 선언하는 것이 좋다.
- readonly를 사용하면 변경하면서 발생하는 오류를 방지할 수 있고 변경이 발생하는 코드도 쉽게 찾을 수 있다.
- const 와 readonly 차이를 이해해야 한다.
- readonly는 얕게 동작한다.

## ITem 18 매핑된 타입을 사용하여 값을 동기화하기

- 매칭된 타입은 한 객체가 또 다른 객체와 정확히 같은 속성을 가지게 할 때 이상적이다.
- 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려해야 한다.

```ts
interface ScatterProps {
  xs: number[];
  ys: number[];
  color: string;
  onClick: (x: number, y: number, index: number) => void;
}
// REQUIRES_UPDATE가 ScatterProps와 동일한 속성을 가져야 한다.
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  xy: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

# 3장 타입 추론

> 타입 추론에서 발생할 수 있는 몇 가지 문제와 해법 안내. 어떻게 타입을 추론하고 언제 타입 선언을 작성해야 하는지, 타입 추론이 가능하더라도 명시적으로 선언을 작성하는 것이 필요한 상황은 언제인지 알아본다.

- ts는 타입 추론을 적극적으로 수행 => 타입 구문의 수를 줄여주어 안정성 향상

## Item 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

- 타입 추론이 된다면 명시적 타입 구문은 필요하지 않다. 타입에 확신이 없다면 편집기를 통해 체크하면 된다.

- const axis2 = 'y'; // 타입은 "y" ????(110)

- 추론이되지만 타입을 명시하는 경우:

1. 객체 리터럴

- 잉여 속성 체크
- 객체가 사용되는 곳이 아닌 선언한 곳에서 타입 오류가 발생하도록 한다.

2. 함수 반환

- 더 직관적이다.
- 타입에 대한 주석을 작성할 수 있다.

## Item 20 다른 타입에는 다른 변수 사용하기

> 변수의 값은 바뀔 수 있지만 그 타입은 보통 바뀌지 않는다.
> 혼란을 막기 위해 타입이 다른 값을 다룰 때는 변수를 재사용하지 않도록 하자.

유니온 타입으로 범위를 좁혀 사용하기 보다는 다른 타입에 별도의 변수를 사용하는게 더 바람직하다.

- 서로 관련이 없는 두 개의 값을 분리
- 변수명을 더 구체적으로 짓기
- 타입 추론을 향상시키며 타입 구문이 불필요해짐
- 타입이 더 간결해짐
- let 대신 const로 변수를 선언하게 됨 => 코드가 더 간결하고 타입 체커가 추론하기에도 더 좋음

## 더 알아봐야할 것

- '할당'에 대한 개념 (65p)
- 표준 lib의 제너릭 타입(item14)
