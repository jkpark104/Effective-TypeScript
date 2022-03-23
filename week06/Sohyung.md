# Item 36 ~ 40

## 들어가며

이펙티브 타입스크립트의 4장 타입 설계에 포함된 Item 36~40 내용을 정리했습니다.

## Item 36 해당 분야의 용어로 타입 이름 짓기

> **요약**
>
> - 가독성을 높이고 추상화 수준을 올리기 위해 해당 분야의 용어를 사용해야 한다.
> - 같은 의미에는 같은 단어를 사용해야 한다. 특별한 의미가 있을 때만 용어를 구분해야 한다.

- 구체적인 용어 사용 (name -> commonName, species ...)
- 공식적이거나 전문적인 용어가 있다면 활용
- 같은 의미라면 같은 단어를 사용
- 데이터 자체가 무엇인지를 고려

## Item 37 공식 명칭에는 상표를 붙이기

> **요약**
>
> - 타입스크립트는 구조적 타이핑(덕 타이핑)을 사용하기 때문에 값을 세밀하게 구분하지 못하는 경우가 있다. 값을 구분하기 위해 공식명칭이 필요하다면 `상표 기법`을 활용할 수 있다.
> - 상표 기법은 타입 시스템에서 동작하지만 런타임에 상표를 검사하는 것과 동일한 효과를 얻을 수 있다.

```ts
interface Vector2D {
  _brand: '2d'; // **상표**
  x: number;
  y: number;
}

// 상표 붙이기 함수
function ved2D(x: number, y: number): Vector2D {
  return { x, y, _brand: '2d' };
}

// 매개변수의 타입에 상표까지 체크
function calculateNorm(p: Vector2D) {
  return Math.sqrt(p.x * p.x + p.y * p.y);
}

calculateNorm(vec2D(3, 4)); // 정상, 5
const vec3D = { x: 3, y: 4, z: 1 };
calculateNorm(vec3D); // ~~~~ '_brand' 속성이 ... 형식에 없습니다.

// 타입에 상표가 없었다면 vec3D도 x,y 값을 덕타이핑으로 받아 정상적으로 동작하였을 것이다.
```

- 절대 경로 어쩌고 예제 이해하기

# 5장 any 다루기

> `TS`의 타입 시스템은 선택적이고 점진적이기 때문에 정적이면서 동적이다. 즉, **프로그램의 일부분에서만 타입 시스템을 적용할 수** 있다. 이러한 특성으로 **점진적인 마이그레이션**이 가능하다. 5장에서는 코드의 일부분에 타입 체크를 비활성화시켜주는 `any` 타입을 잘 활용하는 방법을 다룬다.

## Item38 any 타입은 가능한 좁은 범위에서 사용하기

> **요약**
>
> - 의도치 않은 타입 안전성의 손실을 피하기 위해서 **any의 사용 범위를 최소한**으로 좁혀야 한다.
> - 함수의 반환 타입이 any인 경우 타입 안정성이 나빠진다. 따라서 **any 타입을 반환하면 절대 안된다**.
> - 강제로 타입 오류를 제거하려면 any 대신 **`@ts-ignore`** 사용하는 것이 좋다.

### 함수에서의 좁은 any 예시

```ts
function bad() {
  const x: any = expressionReturningFoo(); // NO!
  processBar(x);
  // return x; 를 하면 반환값이 any가 될 수도 있음!
}

function better() {
  const x = expressionReturningFoo();
  processBar(x as any); // 매개변수의 타입만 any로 두어 any 범위 좁힘
}

function better2() {
  const x = expressionReturningFoo();
  // @ts-ignore
  processBar(x); // 임시로 any를 지정해야할 때는 @ts-ignore을 사용
}
```

### 객체에서의 좁은 any 예시

객체 안의 한 프로퍼티(c)에 대한 타입체크를 무시하려고 한다. 이때 해당 프로퍼티에만 any를 사용하자.

```ts
const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value,
  },
} as any; // NO! 객체 전체를 any로 단언하기 금지
// 모든 속성에 대해 타입체크를 하지 못한다.

const config: Config = {
  a: 1,
  b: 2,
  c: {
    key: value as any, // 이게 더 좁기 때문에 낫다.
  },
};
```

## Item39 any를 구체적으로 변형해서 사용하기

> **요약**
>
> - any를 사용할 때는 정말로 모든 값이 허용되어야만 하는지 면밀히 검토해야 한다.
> - any보다 더 정확하게 모델링할 수 있도록 any[] 또는 {{id: string}: any} 또는 () => any 처럼 구체적인 형태를 사용해야 한다.

```ts
// 함수의 매개변수
// 배열인데 내부 값을 알 수 없을 때
any[]

// 배열의 배열 형태일 때
any[][]

// 객체인데 내부 프로퍼티를 알 수 없을 때
{[key: string]: any}

object // 객체의 키를 열거할 수는 있지만 속성에 접근할 수는 없다.

// 객체이지만 속성에 접근할 수 없어야하는 경우는 unknown이 나을 수도 있다. Item 42에서 볼 예정

```

## Item40 함수 안으로 타입 단언문 감추기

> **요약**
>
> - 타입 선언문은 일반적으로 타입을 위험하게 만들지만 상황에 따라 필요하기도 하고 현실적인 해결책이 되기도 한다. 불가피하게 사용해야 한다면 정확한 정의를 가지는 함수 안으로 숨기도록 한다.

## 더 알아봐야할 것

- 불참하였던 지난 스터디 범위 별도로 공부

- `in` 키워드

  > 자바스크립트에서의 정의 :

  - ts에서 활용 [Ts Handbook Mapped types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)

  ```ts
  interface Person {
    name: string;
    age: number;
  }
  type Partial<T> = {
    [P in keyof T]?: T[P]; // P will be each key of T
  };

  type PersonPartial = Partial<Person>;
  // same as { name?: string;  age?: number; }
  ```
