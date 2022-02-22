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

- 미리 알 수 없는 데이터에 대해서만 인덱스 시그니처 사용

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

## 더 알아봐야할 것

- '할당'에 대한 개념 (65p)
- 표준 lib의 제너릭 타입(item14)
