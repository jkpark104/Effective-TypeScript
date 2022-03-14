## 11 | 잉여 속성 체크의 한계 인지하기

- 타입이 명시된 변수에 객체 리터럴을 할당할 때 TS는 해당 타입의 속성이 있는지, 그 외의 속성은 없는지 확인한다.

- 이 과정을 `잉여 속성 체크`라고 부르며 **`잉여 속성 체크`는 오직 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때만 동작한다.**

- 잉여 속성 체크를 이용하면 타입 시스템의 구조적 본질을 해치지 않으면서 객체 리터럴에 알 수 없는 속성을 허용하지 않도록 할 수 있다. (`엄격한 객체 리터럴 체크`)

- 잉여 속성 체크는 임시변수를 사용하거나 타입 단언문을 사용하면 적용되지 않는다.

- 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, TS의 타입체커가 수행하는 구조적 할당 가능성 체크와는 역할이 다르다.

```ts
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  // `Room`형식에 `elephant`가 없습니다.
  elephant: 'present',
};
```

구조적 타이핑 관점에서는 해당 오류가 이상하게 느껴진다.<br>
구조적 타이핑 관점에서는 해당 속성이 일치하면 같다고 판단해야하기 때문이다.

이는 임시변수를 사용해보면 오류가 나지 않는 것을 알 수 있다.

```ts
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present',
};

// 정상
const r: Room = obj;
```

첫 번째 예시와 두 번째 예시의 차이점은 `잉여 속성 체크`라는 과정이 수행되었냐 되지 않았냐이다.

<br>

---

## 12 | 함수 표현식에 타입 적용하기

> 시그니처: 함수 타입

- 함수 타입을 정의할 때 매개변수나 반환 값에 타입을 명시하기 보다는 함수 표현식 전체에 타입구문을 적용하는 것이 좋다.

- 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 시그니처를 적용하는 것이 좋다.

- 라이브러리를 만든다면 공통 콜백에 타입을 제공한다.

- 다른 함수의 시그니처를 참조하려면 `typeof fn`을 사용한다.

<br>

## 13 | 타입과 인터페이스의 차이점 알기

> `named type`을 정의할 때 `type`과 `interface` 두 가지 방법이 있다.

### 비슷한 점

1. 잉여 속성 체크가 동일하게 동작한다.
2. 인덱스 시그니처 사용 가능

   ```ts
   type TDict = {
     [key: string]: string;
   };

   interface IDict {
     [key: string]: string;
   }
   ```

3. 함수 타입 정의 가능

   ```ts
   type TFn = (x: numberr) => string;
   interface IFn {
     (x: number): string;
   }
   ```

4. 제네릭 사용 가능

   ```ts
   type TPair<T> = {
     first: T;
     second: T;
   };

   interface IPair<T> {
     first: T;
     second: T;
   }
   ```

5. 확장 가능

   > 이 때 `type`과 `interface`는 서로서로 확장 가능

   ```ts
   type TStateWithPop = IState & { pop: number };

   interface IStateWithPop extends TState {
     pop: number;
   }
   ```

### 차이점

1. `interface`는 `union`같은 복잡한 타입은 확장할 수 없다.
2. `type`은 `&` 연산자로 가능
3. 튜플과 배열 타입도 `type`키워드로 간결하게 표현 가능하다.

   ```ts
   type Pair = [number, number];
   type StringList = string[];
   type NamedNums = [string, ...number[]];
   ```

4. `interface`는 보강(`argment`)가 가능하다. -> 이를 **선언 병합**이라 한다.

   ```ts
   interface IState {
     name: string;
     capital: string;
   }

   interface IState {
     pop: number;
   }

   const wonjae: IState = {
     name: 'Wonjae',
     capital: 'Hi',
     pop: 500,
   };
   ```

### 둘 중 무엇을 사용해야 하나?

- 타입이 복잡하다면 `type`이 훨씬 용이하다.
- `type`과 `interface`로 모두 표현이 가능한 객체 타입이라면 **일관성**과 **보강**의 관점에서 고려해야 한다.
- 특정 API에 대한 타입 선언을 작성해야 한다면 `interface`를 사용하는 것이 좋다.
  - 새로운 필드를 병합할 수 있기 때문이다.
  - 하지만 프로젝트 내부적으로 사용되는 타입에 선언 병합은 잘못된 설계이다.

<br>

---

## 14 | 타입 연산과 제너릭 사용으로 반복 줄이기

> TS에서 `extends`는 확장보다는 부분집합의 개념이다.

- 타입도 최대한 재사용해서 (`DRY 원칙`) 반복을 줄여야 한다.
- `named type`의 활용을 잘하고 `extends`를 사용해서 `interface` 필드의 반복을 피하자
- 타입들 간의 매핑을 위해 TS가 제공하는 `keyof`, `typeof`, 인덱싱, 매핑된 타입 등을 활용하자
- 제네릭 타입은 타입을 위한 함수이다.
- 타입을 반복하기 보다 제네릭 타입을 사용하여 타입들을 매핑하자.
- 제너릭 타입을 제한하려면 `extends`를 사용하면 된다.

  ```ts
    const dancingDuo = <T extends Name>(x: DancingDuo<T>) => x;

    const couple1 = dancingDuo([
      { first: 'Son' last: 'Wonjae' },
      { first: 'Hong' last: 'Gildong' }
    ]);

    // Name 타입에 필요한 last 속성이 { first: string } 타입에 없습니다.
    const couple2 = dancingDuo([
      { first: 'Bono' },
      { first: 'Prince' }
    ])

  ```

- 표준 라이브러리에 정의된 `Pick`, `Partial`, `ReturnType`같은 제네릭 타입에 익숙해져야 한다.

<br>

---

## 15 | 동적 데이터에 인덱스 시그니처 사용하기

- 인덱스 시그니처는 예측가능성이 낮기 때문에 런타임 때까지 객체의 속성을 알 수 없을 경우에만 사용한다.
- 안전한 접근을 위해 인덱스 시그니처의 값 타입에 `undefined`를 추가하는 것을 고려해야한다.
- 가능하다면 예측가능성을 위해 `interface`, `Record`, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

### 인덱스 시그니처

`[property: string]: string`

- 키의 이름: 키의 위치만 표시하는 용도, 타입체커에서는 사용하지 않음
- 키의 타입: `string`, `number` 또는 `symbol`의 조합이어야 하지만, 보통은 `string`
- 값의 타입: TS가 허용하는 모든 타입

#### 인덱스 시그니처의 단점

- 잘못된 키를 포함해 모든 키를 허용
- 특정 키가 필요하지 않음, `{}`도 유효한 타입으로 인정됨
- 키마다 다른 타입을 가질 수 없음, 특정 프로퍼티는 `string`이 아닌 `number`일 수도 있으나 허용할 수 없음
- 자동완성 기능을 제대로 활용할 수 없음

### `interface`

```ts
  // 너무 광범위
  interface Row1 {
    [column: string]: number
  }

  // 최선
  interface Row2 {
    a: number;
    b?: number;
    c?: number;
    d? : number;
  }

  // 가장 정확하지만 사용하기 번거로움
  type Row3 =
    { a: number; } |
    { a: number; b: number; } |
    { a: number; b: number; c: number; } |
    { a: number; b: number; c: number; d: number; } |

```

### `Record`

```ts
type Vec3D = Record<'x' | 'y' | 'z', number>;

// Type Vec3D = {
//   x: number;
//   y: number;
//   z: number;
// }
```

### 매핑된 타입

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

<br>

---

## 16 | `number` 인덱스 시그니처보다는 `Array`, 튜플, `ArrayLike`를 사용하기

- 배열은 객체이므로 키는 숫자가 아닌 문자열이다.
- 인덱스 시그니처로 사용된 `number`타입은 버그를 잡기 위한 순수 TS 코드이다.
- 인덱스 시그니처에 `number`를 사용하기보다 `Array`나 튜플, `ArrayLike`타입을 사용하자

<br>

---

## 17 | 변경 관련된 오류 방지를 위해 `readonly`사용하기

- 만약 함수가 매개변수를 수정하지 않는다면 `readonly`로 선언하는 것이 좋다.
- `readonly` 매개변수는 인터페이스를 명확하게 하고, 매개변수가 변경되는 것을 방지한다.
- `readonly`를 사용하면 불변성을 지키기 수월하다.
- `const`와 `readonly`는 다르다.
  - `const`는 재할당은 막지만 불변성을 지키기는 어렵다.
  - `readonly`는 재할당을 허용하고 불변성을 지키기 용이하다.
- `readonly`는 얕게 동작한다.

<br>

---

## 18 | 매핑된 타입을 사용하여 값을 동기화하기

> 이전 상태와 현재 상태를 비교해서 값이 달라졌다면 업데이트하는 함수를 작성했다고 가정했을 때 실패에 닫힌 방법과 실패에 열린 방법이 있다.

### 실패에 닫힌 방법

- 오류 발생 시에 적극적으로 대처하며, 방어적, 보수적 접근법이다
- 보안과 관련된 곳이라면 해당 방식을 사용하는 것이 좋을 수 있다.

### 실패에 열린 방법

- 오류 발생 시에 소극적으로 대처하는 방법
- 기능에 무리가 없고 사용성이 중요하다면 해당 방식이 좋을 수 있다.

### TS를 활용하기

- 매핑된 타입을 사용해서 관련된 값과 타입을 동기화 한다.
- 인터페이스에 새로운 속성을 추가할 때 선택을 강제하도록 매핑된 타입을 고려해야 한다.

```ts
const REQUIRES_UPDATE: { [k in keyof ScatterProps]: boolean } = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(oldProps: Props, newProps: Props) {
  let k: keyof Props;

  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE) {
      return true;
    }
  }
  return false;
}
```

<br>

---

## 19 | 추론 가능한 타입을 사용해 장황한 코드 방지하기

- TS가 타입을 추론할 수 있다면 타입 구문은 작성하지 않는 것이 좋다.
- 이상적인 함수/메서드의 시그니처는 시그니처에는 타입 구문이 있지만 함수 내의 지역 변수에는 타입 구문이 없는 형태이다.
- 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려해야 한다.
  - 구현상의 오류가 정확한 위치에 표시되도록 도와준다.

### 반환타입을 명시했을 때의 이점

- 구현상의 오류를 정확히 표시한다.
- 함수에 대해 명확하게 알 수 있다.
- `named type`의 이점을 더 잘 살려 가독성을 높여준다.

아래와 같이 반환타입을 명시하지 않으면 타입 추론을 통해 `number` | `Promise<any>`로 반환값이 추론되어 `then`메서드를 사용했을 때 호출문에서 에러가 발생합니다.

```ts
  interface Cache = {
    [ticker: string]: number
  }

  const cache: Cache = {}

  function getQuote(ticker: string) {
    if (ticker in cache) return cache[ticker]

    return fetch(`https://quotes.example.com/?q=${ticker}`)
      .then(res => res.json())
      .then(quote => {
        cache[ticker] = quote;
        return quote
      })
  }

  // ~~~ `number | Promise<any>` 형식에 `then` 속성이 없습니다.
  // `number`형식에 `then` 속성이 없습니다.
  getQuote('MSFT').then(considerBuying);
```

반면 반환 타입을 명시하면 구현상의 오류를 정확히 표시합니다.

```ts
  interface Cache = {
    [ticker: string]: number
  }

  const cache: Cache = {}

function getQuote(ticker: string): Promise<number> {
  // `number`형식은 `Promise<number>`형식에 할당할 수 없습니다.
  if (ticker in cache) return cache[ticker]
    ...
  }

```

<br>

---

## 20 | 다른 타입에는 다른 변수 사용하기

- 변수의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않는다.
- 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 한다.
