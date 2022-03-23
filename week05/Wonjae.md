## 36 | 해당 분야의 용어로 타입 이름 짓기

> 가독성을 높이고, 추상화 수준을 올리기 위해 타입을 정의할 때 해당 분야의 용어로 정의해야한다.

### 타입, 속성, 변수에 이름을 붙일 때 명심해야 할 세 가지 규칙

- 동일한 의미를 나타낼 때는 같은 용어 사용
- `data`, `info`, `thing`, `item`, `object`, `entity`같은 범위가 모호한 이름은 피해야한다.
- 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지를 나타내야 한다.

<br>

---

## 37 | 공식 명칭에는 상표를 붙이기

> TS는 구조적 타이핑을 사용하기 때문에 값을 세밀하게 구분하지 못하는 경우가 있다.<br>
> 값을 구분하기 위해 공식명칭이 필요하다면 상표(`brand`)를 붙이는 것도 고려해야 한다.

```ts
type Meters = number & { _brand: 'meters' };
type Seconds = number & { _brand: 'seconds' };

const meters = (m: number) => m as Meters;
const seconds = (s: number) => s as Seconds;

const oneKm = meters(1000); // type is Meters
const oneMin = seconds(60); // type is Seconds

// 하지만 산술 후에는 타입이 `number`로 초기화됨
const tenKm = oneKm * 10; // type is number
const oneHour = oneMin * 60; // type is number
```

> 상표 사용법

```ts
type AbsolutePath = string & { _brand: 'abs' };

function isAbsolutePath(path: string): path is AbsolutePath {
  return path.startsWith('/');
}

function listAbsolutePath(path: AbsolutePath) {}

function useAbsolutePath(path: string) {
  if (isAbsolutePath(path)) {
    listAbsolutePath(path);
  }
  // `string`형식의 인수는 `AbsolutePath`형식의 매개변수에 할당될 수 없다.
  listAbsolutePath(path);
}
```

<br>

---

## 38 | `any`타입은 가능한 한 좁은 범위에서만 사용하기

- 의도치 않은 타입 안전성의 손실을 피하기 위해 `any`의 사용 범위를 최소한으로 좁혀야 한다.
- 함수의 반환 타입을 `any`로 지정하면 타입 안정성이 나빠진다.
  - `any`타입을 반환하는 건 지양수준이 아니라 엄격하게 금해야 한다.
- 강제로 타입 오류를 제거하려면 `any`대신 `@ts-ignore`를 사용하는 것이 좋다.

> 이렇게 사용하는 것 자체에 오류가 있는 코드이긴 하나 예제로서 사용됨
>
> - `Foo` 타입을 `Bar`를 매개변수로 받는 함수에 전달한다는 것 자체가 말이 안됨

```ts
function f1() {
  const x = expressionReturningFoo();

  // @ts-ignore
  processBar(x);
  return x;
}
```

<br>

---

## 39 | `any`를 구체적으로 변형해서 사용하기

- `any`를 사용할 때는 정말 모든 값이 허용되어야만 하는지 아주아주 면밀히 검토해야 함
  - 그냥 사용하지 말기
- 정말 만약에 사용해야 한다면 타입 시스템을 활용할 수 있도록 조금 더 구체적으로 모델링해야 함
  - `any[]`, `{ [id: string]: any }`, `() => any`...

<br>

---

## 40 | 함수 안으로 타입 단언문 감추기

- 타입 선언문?(단언문이 맞는듯)은 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다.
- 불가피하게 사용해야 한다면 타입 범위를 확실하게 지정한, 즉 응집도 있는 함수 몸체에 작성하는 것이 좋다.
