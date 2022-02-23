## 노션 링크

https://hustle-dev.notion.site/Week02-81060e37d771455489662c303cb1d22f

| 노션으로 보는 것이 더 깔끔합니다! 😎

---

### [아이템11] 잉여 속성 체크의 한계 인지하기

잉여 속성 체크: 객체 리터럴을 타입이 명시된 변수에 할당할 때 **해당타입의 속성유무와 그 외의 속성을 확인**하여 에러를 발생시킴

잉여속성 체크를 피하는법: 임시 변수 사용

```tsx
interface Options {
  title: string;
  darkMode?: boolean;
}

const intermediate = { darkmode: true, title: 'Ski Free' };
const o: Options = intermediate;

// 오류
const o: Options = { darkmode: true, title: 'Ski Free' };
// 개체 리터럴은 알려진 속성만 지정할 수 있지만 'Options' 형식에 'darkmode'이(가) 없습니다. 'darkMode'을(를) 쓰려고 했습니까?
```

공통속성 체크: 약한 타입(?:)과 관련된 할당문마다 수행하며 공통된 속성이 없으면 오류를 표시함!

### [아이템12] 함수 표현식에 타입 적용하기

TS에서 일반 함수선언문보다 함수 표현식을 사용하여 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하는 것이 좋다!(재사용 측면에서)

함수 타입 선언의 장점

- 불필요한 코드의 반복을 줄임
- 하나의 함수 타입으로 통합 가능
- 함수 매개변수에 타입 선언하는 것보다 코드가 간결해지고 안전해진다.

```tsx
// 이렇게 쓰기보다
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error('Request failed: ' + response.status);
  }
  return response;
}

// 이렇게 쓰자
declare function fetch(input: RequestInfo, init?: RequestInit): Promise<Response>;

const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    throw new Error('Request failed: ' + response.status);
  }
  return response;
};
```

→ 다른 함수의 시그니처 참조시, `typeof fn` 을 사용하자.

### [아이템13] 타입과 인터페이스의 차이점 알기

타입이나 인터페이스 접두어에 T or I를 붙이는것은 C#에서 비롯된 관례이므로 지양하자.

타입과 인터페이스의 다른점

- 인터페이스는 `extends`키워드를 통해 확장이 가능하다.
- 유니온과 같은 복잡한 타입을 `type`은 확장할 수 있다. interface는 X
- 튜플과 배열 타입도 `type` 키워드를 통해 더 간결하게 표현 가능
- 인터페이스로 튜플 구현시 `concat` 같은 메서드 사용 불가
- interface는 보강 기법 사용 가능 → 선언을 2번해서 속성 확장

복잡한 타입 → 타입 별칭사용 추천

일관성과 보강의 관점에서 → 인터페이스 추천

캡틴판교는 인터페이스 추천(확장성 측면에서)

### [아이템14] 타입 연산과 제너릭 사용으로 반복 줄이기

반복을 줄이는 방법

- 타입에 이름을 붙이기
- 중복된 함수 타입을 시그니처를 명명한 타입으로 분리하기
- 인터섹션 연산자 사용
- `Pick`, `Partial`과 같은 유틸리티 타입 사용
- 값의 형태에 해당하는 타입: `typeof`
- 함수나 메서드의 반환값에 해당하는 타입: `ReturyType<typeof 함수>`

제네릭 타입에서 매개변수를 **제한**하는법 → extends(’확장'이 아닌 ‘부분 집합'의 개념)

즉 이말의 의미는 `<T extends “어떤 타입">`일 때, T가 “어떤 타입”의 부분집합이라고 생각하면 쉽다. → 이부분 이해가 잘안감 ✅ 해결

```tsx
// K가 T의 key들만 올 수 있음(number | string | symbol)
type Pick<T, K extends keyof T> = {
  [k in K]: T[k];
};

// 객체의 경우
interface Name {
  first: string;
  last: string;
}
// 위 보다
interface Name2 {
  first: string;
  last: string;
  third: string;
}
// 이게 더 범위가 작으므로 올 수 있음
const couple2: DancingDuo<Name2> = [
  { first: 'Ginger', last: 'hi', third: 'ho' },
  { first: 'Ginger', last: 'hi', third: 'ho' },
];
```

### [아이템15] 동적 데이터 인덱스 시그니처 사용하기

인덱스 시그니처 → `type IndexSig = { [property: string] : string };`

인덱스 시그니처의 네 가지 단점

- 잘못된 키 포함 모든 키허용(name → Name을 사용해도 무방)
- 특정 키가 필요하지 않다.
- 키마다 다른 타입을 가질 수 없다.
- 키는 무엇이든 가능하기에 자동 완성 기능이 동작하지 않는다.

> 위와 같은 인덱스 시그니처는 **런타임 때까지 객체의 속성을 알 수 없을 경우**에만 사용!, 또한 안전한 접근을 위해 `undefined` 추가를 고려할 수 있지만 이 부분을 체크하는 코드가 필요!

Record: `Record<’x’ | ‘y’ | ‘z’, number>`

매핑된 타입: `{ [k in ‘x’ | ‘y’ | ‘z’]: number } or { [k in ‘a’ | ‘b’ | ‘c’]: k extends ‘b’ ? string: number; }`

위와 같은 인덱스 시그니처보다 **정확한 타입**을 사용하자!

### [아이템16] number 인덱스 시그니처보다는 Array, 튜플, ArrayLike를 사용하기

자바스크립트는 배열, 객체에 대한 인덱스 접근을 모두 문자열로 변환하여 인식한다.

그러나 타입스크립트는 숫자 키를 허용하여 문자열 키와 다른 것으로 인식한다.

```tsx
interface Array<T> {
  // ...
  [n: number]: T;
}
```

이를 통해 타입 체크 시점에 오류를 잡아준다.(인덱스 시그니처로 사용된 `number`타입은 버그를 잡기 위한 순수 TS 코드)

```tsx
const xs = [1, 2, 3];
const x1 = xs['1']; // (x), 에러 발생(인덱스 식이 number형식이 아닌 string이라 에러)
```

`Object.keys`같은 구문들은 여전히 문자열로 반환된다. 그러나 타입스크립트에서 for...in 문안에서 사용하는 것은 **배열을 순회하는 코드 스타일에 대한 실용적인 허용**이라고 생각하여 배열에 문자열 인덱스로 접근해도 된다.

for...in 은 for..of, for보다 타입이 불확실하면 몇 배나 느리다.

`Array`타입이 사용하질 않은 메서드를 가지는게 싫다면 `ArrayLike` 타입 사용! → 여전히 키는 문자열

인덱스 시그니처에 number를 사용하기보다 Array나 튜플, 또는 ArrayLike타입을 사용하는 것이 좋다! → 어떤 특별한 의미를 지닌다는 의미에서

### [아이템17] 변경 관련된 오류 방지를 위해 readonly 사용하기

자바스크립트의 참조 타입들은 함수로 호출해서 인수로 넘길때, 그 안의 내용을 변경할 수 있다.

→ 이로 인해 발생하는 문제가 있어서 `readonly` 접근 제어자를 통해 제한할 수 있다.

readonly

- 참조타입의 요소를 읽을 수 있지만 쓸 수는 없다.
- length를 읽을 수 있지만 바꿀수는 없다.(ex: `arr.length = 0` → X)
- 배열을 변경하는 `pop`을 비롯한 다른 메서드를 호출할 수 없다.

매개변수가 readonly이면

1. 타입스크립트는 함수 내에서 변경이 일어나는지 체크하고,
2. 호출하는 쪽에서 함수가 매개변수를 변경하지 않는다는 보장을 받고,
3. 함수에 readonly 배열을 매개변수로 넣을 수 있다.

`number[]`의 타입이 `readonly number[]`의 타입보다 기능이 많아서 **서브타입**이 되어 `readonly` 배열에 할당할 수 있지만, 반대는 불가능하다.

readonly 사용시 장점으로 **인터페이스를 명확히 하고 타입 안전성을 높일 수 있다.**

readonly는 **얕게 동작**하는 것임을 명심하자!

### [아이템18] 매핑된 타입을 사용하여 값을 동기화하기

```jsx
interface ScatterProps {
  xs: number[];
  ys: number[];

  xRange: [number, number];
  yRange: [number, number];
  color: string;

  onClick: (x: number, y: number, index: number) => void;
}

// 실패에 닫힌 접근법: 오류에 적극적으로 대처
// 새로운 프로퍼티 추가시 onclick으로만 return true반환 ㅜㅜ..
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'onClick') return true;
    }
  }
  return false;
}

// 실패에 열린 접근법: 오류에 소극적으로 대처
// 새로운 프로퍼티 추가시 고려 X
function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
  );
}

// 매핑된 타입을 이용하면 ScatterProps에 속성이 추가되었을 때,
// 이 부분도 에러가 발생하여 이 부분을 고쳐야만 함 (선택 강제)
// -> 매핑된 타입을 고려하는 이유
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
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

## 3장 - 타입 추론

### [아이템19] 추론 가능한 타입을 사용해 장황한 코드 방지하기

타입스크립트는 타입 추론기능이 제공되므로 굳이 불필요한 타입 구문을 추가하지 말자.

→ 어디에 타입 구문을 넣고, 어디를 빼야할지 아는 것이 중요하다!

함수의 인수로 객체를 넘기는 경우 비구조화 할당문을 사용하면 모든 지역변수의 타입이 추론되도록 할 수 있다.

이상적인 타입스크립트 코드는 **함수/메서드 시그니처에 타입 구문을 포함하지만, 함수 내에서 생성된 지역 변수에는 타입 구문을 넣지 않는다.**

객체 리터럴(잉여 속성 체크), 함수 반환에는 타입을 명시하는 것이 좋다.

→ 내부 구현의 오류가 사용자 코드 위치에 나타나는것을 방지하고, 개발자가 오류가 어디에서 났는지 알 수 있음

반환타입을 명시하면 함수에 대해 명확하게 알 수 있고, 명명된 타입을 사용할 수 있다.

### [아이템20] 다른 타입에는 다른 변수 사용하기

다른 타입의 경우 변수를 재사용하지 말고 별도의 변수로 분리하자.

- 서로 관련 없는 두 개의 값을 분리(id, serial)
- 변수명을 더 구체적으로 지을 수 있다.
- 타입 추론을 향상시키고, 타입 구문이 불필요해진다.
- 타입이 좀 더 간결해진다.(string | number 대신 → string과 number 사용
- let 대신 const로 변수 선언

```jsx
const id = '12-34-56';
fetchProduct(id);

{
  const id = 123456; // id 대신 serial을 사용하자!
  fetchProductBySerialNumber(id);
}
```

위와 같이 스코프가 다른경우 서로 아무런 관계 없이 사용할 수 있지만 사람에게 혼동을 줄 수 있기 때문에 목적이 다른 곳에는 별도의 변수명을 사용하는 것이 좋다.
