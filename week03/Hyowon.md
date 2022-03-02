# 3주차

## 아이템21 타입넓히기

- 모든 변수는 런타임에 유일한 값을 가진다
- 정적 분석 시점에 변수는 가능한 값들의 집합인 타입을 가진다.

### 타입 넓히기(Type Widening)

지정된 단일 값을 가지고 할당 가능한 값들의 집합을 유추하는 과정

```jsx
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
let x = 'x';
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);
               // ~ Argument of type 'string' is not assignable to
               //   parameter of type '"x" | "y" | "z"'
```

변수 x의 타입이 할당 시점에 넓히기가 동작해서 string으로 추론되었다.

string 타입은 `"x" | "y" | "z"` 타입에 할당이 불가능하므로 오류

```jsx
const mixed = ['x',1];
```

mixed의 타입이 될 수 있는 후보들

- ('x' | 1)[]
- ['x', 1]
- [string, number]
- readonly [string, number]
- (string|number)[]
- readonly (string|number)[]
- [any, any]
- any[]

타입스크립트는 `mixed: (string | number)[]` 타입으로 추측한다.

```jsx
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
const x = 'x';  // type is "x"
let vec = {x: 10, y: 20, z: 30};
getComponent(vec, x);  // OK
```

const를 사용하면 변수 x는 재할당될 수 없으므로 타입스크립트는 의심의 여지 없이 더 좁은 타입 “x”으로 추론할 수 있다.

하지만 객체와 배열의 경우에는 여전히 문제가 있다.

타입 추론의 강도를 직접 제어하려면 타입스크립트의 기본 동작을 재정의해야한다.

1. 명시적 타입 구문을 제공하는 것

```jsx
const v: {x: 1|3|5} = {
  x: 1,
};  // Type is { x: 1 | 3 | 5; }
```

1. 타입 체커에 추가적인 문잭을 제공하는 것
2. const 단언문을 사용한 것

```jsx
interface Vector3 { x: number; y: number; z: number; }
function getComponent(vector: Vector3, axis: 'x' | 'y' | 'z') {
  return vector[axis];
}
const v1 = {
  x: 1,
  y: 2,
};  // Type is { x: number; y: number; }

const v2 = {
  x: 1 as const,
  y: 2,
};  // Type is { x: 1; y: number; }

const v3 = {
  x: 1,
  y: 2,
} as const;  // Type is { readonly x: 1; readonly y: 2; }
```

값 뒤에 as const를 작성하면, 타입스크립트는 최대한 좁은 타입으로 추론한다

v3에는 넓히기가 동작하지 않음

배열을 튜플 타입으로 추론할 때에도 as const를 사용할 수 있다.

```jsx
const a1 = [1, 2, 3];  // Type is number[]
const a2 = [1, 2, 3] as const;  // Type is readonly [1, 2, 3]
```

요약

- 타입스크립트가 넗히기를 통해 상수의 타입을 추론하는 법을 이해해야한다.
- 동작에 영향을 줄 수 있는 방법인 const, 타입구문, 문맥, as const에 익숙해져야 한다.

## 아이템22 타입좁히기

### 타입좁히기(Type Narrowing)

- 타입 넓히기의 반대는 타입 좁히기이다
- 타입스크립트가 넓은 타입으로 부터 좁은 타입으로 진행하는 과정을 말한다.

가장 일반적인 예시는 null 체크

```jsx
const el = document.getElementById('foo'); // Type is HTMLElement | null
if (el) {
  el // Type is HTMLElement
  el.innerHTML = 'Party Time'.blink();
} else {
  el // Type is null
  alert('No element #foo');
}
```

instanceof 사용하기

```jsx
function contains(text: string, search: string|RegExp) {
  if (search instanceof RegExp) {
    search  // Type is RegExp
    return !!search.exec(text);
  }
  search  // Type is string
  return text.includes(search);
}
```

속성체크

```jsx
interface A { a: number }
interface B { b: number }
function pickAB(ab: A | B) {
  if ('a' in ab) {
    ab // Type is A
  } else {
    ab // Type is B
  }
  ab // Type is A | B
}
```

Array.isArray 같은 내장함수 이용하기

```jsx
function contains(text: string, terms: string|string[]) {
  const termList = Array.isArray(terms) ? terms : [terms];
  termList // Type is string[]
  // ...
}
```

명시적 태그를 붙이는 방법

```jsx
interface UploadEvent { type: 'upload'; filename: string; contents: string }
interface DownloadEvent { type: 'download'; filename: string; }
type AppEvent = UploadEvent | DownloadEvent;

function handleEvent(e: AppEvent) {
  switch (e.type) {
    case 'download':
      e  // Type is DownloadEvent
      break;
    case 'upload':
      e;  // Type is UploadEvent
      break;
  }
}
```

이 패턴은 `태그된 유니온` 또는 `구별된 유니온` 이라고 불린다

만약 타입스크립트가 타입을 식별하지 못한다면, 식별을 돕기 위해 커스텀 함수를 도입할 수 있다.

```jsx
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // Type is HTMLInputElement
    return el.value;
  }
  el; // Type is HTMLElement
  return el.textContent;
}
```

이러한 기법을 `사용자 정의 타입 가드` 라고 한다.

(p127)???

```jsx
const jackson5 = ['Jackie', 'Tito', 'Jermaine', 'Marlon', 'Michael'];
function isDefined<T>(x: T | undefined): x is T {
  return x !== undefined;
}
const members = ['Janet', 'Michael'].map(
  who => jackson5.find(n => n === who)
).filter(isDefined);  // Type is string[]
```

### 잘못된 방법

```jsx
const el = document.getElementById('foo'); // type is HTMLElement | null
if (typeof el === 'object') {
  el;  // Type is HTMLElement | null
}
```

자바스크립트에서 typeof null이 object 이기 때문에 if문에서 null이 제외되지 않았음

```jsx
function foo(x?: number|string|null) {
  if (!x) {
    x;  // Type is string | number | null | undefined
  }
}
```

빈문자열 ‘’과 0 모두 false가 되기때문에 타입이 전혀 좁혀지지 않음

여전히 블록 내에서 string 또는 number가 된다

편집기에서 타입을 조사하는 습관을 가지면 타입 좁히기가 어떻게 동작하는지 자연스레 익힐 수 있다.

타입이 어떻게 좁혀지는지 이해한다면 타입 추론에 대한 개념을 잡을 수 있고, 오류 발생의 원인을 알 수 있으며, 타입 체커를 더 효율적으로 이용할 수 있다.

요약

- 분기문 외에도 여러 종류의 제어 흐름을 살펴보며 타입스크립트가 타입을 좁히는 과정을 이해해야한다.
- 태그된/구별된 유니온과 사용자 정의 타입 가드를 사용하여 타입 좁히기 과정을 원할하게 만들 수 있다.

[TS Playground - An online editor for exploring TypeScript and JavaScript](https://www.typescriptlang.org/play#example/type-widening-and-narrowing)

## 아이템23 한꺼번에 객체 생성하기

변수의 값은 변경될 수 있지만, 타입스크립트의 타입은 일반적으로 변경되지 않는다.

객체를 생성할 때는 속성을 하나씩 추가하기보다는 여러 속성을 포함해서 한꺼번에 생성해야 타입추론에 유리하다.

```jsx
const pt = {};
pt.x = 3;
// ~ Property 'x' does not exist on type '{}'
pt.y = 4;
// ~ Property 'y' does not exist on type '{}'
```

```jsx
interface Point { x: number; y: number; }
const pt = {
  x: 3,
  y: 4,
};  // OK
```

작은 객체들을 조합해서 큰 객체를 만들어야 하는 경우에는

Object.assign 대신 객체 전개 연산자를 사용한다

```jsx
interface Point { x: number; y: number; }
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {};
Object.assign(namedPoint, pt, id);
namedPoint.name;
        // ~~~~ Property 'name' does not exist on type '{}'
```

```jsx
interface Point { x: number; y: number; }
const pt = {x: 3, y: 4};
const id = {name: 'Pythagoras'};
const namedPoint = {...pt, ...id};
namedPoint.name;  // OK, type is string
```

(P131) ???

```jsx
declare let hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
function addOptional<T extends object, U extends object>(
  a: T, b: U | null
): T & Partial<U> {
  return {...a, ...b};
}

const president = addOptional(firstLast, hasMiddle ? {middle: 'S'} : null);
president.middle  // OK, type is string | undefined
```

요약

- 속성을 제각각 추가하지 말고 한꺼번에 객체로 만들어야 한다
- 안전한 타입으로 속성을 추가하려면 객체 전개를 사용하면 된다
- 객체에 조건부로 속성을 추가하는 방법을 익히도록 한다.

## 아이템 24 일관성 있는 별칭 사용하기

```jsx
const borough = {name: 'Brooklyn', location: [40.688, -73.979]};
const loc = borough.location;
```

borough.location 배열에 loc이라는 별칭을 만들었다.

별칭의 값을 변경하면 원래 속성 값에서도 변경된다.

별칭을 남발해서 사용하면 제어 흐름을 분석하기 어렵다.

요약

- 별칭은 타입스크립트가 타입을 좁히는 것을 방해한다. 따라서 변수에 별칭을 사용할 때는 일관되게 사용해야 한다.
- 비구조화 문법을 사용해서 일관된 이름을 사용하는 것이 좋다.
- 함수 호출이 객체 속성의 타입 정제를 무효화 할 수 있다는 점을 주의해야 한다
- 속성보다 지역 변수를 사용하면 타입 정제를 믿을 수 있다.

## 아이템 25 비동기 코드에는 콜백 대신 async 함수 사용하기

- 콜백보다는 프로미스가 코드를 작성하기 쉽다.
- 콜백보다는 프로미스가 타입을 추론하기 쉽다.

프로미스를 직접 생성해야 할 때, 특히 setTimeout과 같은 콜백 API를 래핑할 경우에는

async/await를 사용해야한다.

- 일반적으로 더 간결하고 직관적인 코드가 된다
- async 함수는 항상 프로미스를 반환하도록 강제된다.

## 아이템 26 타입 추론에 문맥이 어떻게 사용되는지 이해하기

튜플 사용 시 주의점

```jsx
type Language = 'JavaScript' | 'TypeScript' | 'Python';
function setLanguage(language: Language) { /* ... */ }
// Parameter is a (latitude, longitude) pair.
function panTo(where: [number, number]) { /* ... */ }

panTo([10, 20]);  // OK

const loc = [10, 20];
panTo(loc);
//    ~~~ Argument of type 'number[]' is not assignable to
//        parameter of type '[number, number]'
```

```jsx
const loc: [number, number] = [10, 20];
panTo(loc);  // OK
```

```jsx
const loc = [10, 20] as const;
panTo(loc);
   // ~~~ Type 'readonly [10, 20]' is 'readonly'
   //     and cannot be assigned to the mutable type '[number, number]'
```

이 추론은 너무 과하게 정확하다

panTo의 타입 시그니처는  where의 내용이 불변이라고 보장하지 않는데

loc 매개변수가 readonly 타입이므로 동작하지 않음

```jsx
function panTo(where: readonly [number, number]) { /* ... */ }
const loc = [10, 20] as const;
panTo(loc);  // OK
```

따라서 panTo함수에 readonly 구문을 추가해준다.

요약

- 타입 추론에서 문맥이 어떻게 쓰이는지 주의해서 살펴야한다
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해야한다
- 변수가 정말로 상수라면 상수 단언(as const)를 사용해야 한다. 그러나 상수 단언을 사용하면 정의한 곳이 아니라 사용한 곳에서 오류가 발생하므로 주의해야한다.

## 아이템 27 함수형 기법과 라이브러리로 타입 흐름 유지하기

요약

- 타입흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이기 위해서는 직접 구현하기 보다는 내장된 함수형 기법과 로대시 같은 유틸리티 라이브러리를 사용하는 것이 좋습니다.
