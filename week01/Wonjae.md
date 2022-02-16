## 01 | TS와 JS의 관계

![TS,JS](https://user-images.githubusercontent.com/87295692/154225229-a7015450-594c-492d-a9fa-051af2c060e1.png)

> 모든 JS 프로그램은 TS 프로그램이다.

> 모든 TS는 별도의 문법을 갖고 있기 때문에 일반적으로 유효한 JS 프로그램이 아니다.

<br>

### TS의 타입시스템은 JS런타임 동작을 모델링 한다

- 런타임 오류를 발생시키는 코드 탐색
- **타입 체커를 통과하면서도 런타임 오류를 발생시키는 코드는 존재할 수 있다**
- TS 문법 설정의 엄격함은 온전히 취향의 차이이다.

<br>

---

## 2 | TS 설정

### CLI보다 `tsconfig.json` 파일을 사용하는 것이 좋다

> TS를 어떻게 사용할 계획인지 동료들이나 다른 도구들이 알 수 있다.

<br>

CLI 사용법

```shell
  tsc --noImplicitAny program.ts
```

<br>

파일 사용법

```shell
  tsc --init
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "noImplicitAny": true
  }
}
```

<br>

### TS에서 엄격한 체크를 하고 싶다면 `strict`설정을 고려해야 한다

> 대표적인 설정 값
>
> `noImplicitAny`: 암묵적 `any`의 사용을 설정
>
> `strictNullChecks`: `null`과 `undefined`가 모든 타입에서 허용되는지 확인하는 설정

```json
{
  "compilerOptions": {
    "strict": true
    /* 명시적이지 않은 'any' 유형으로 표현식 및 선언 사용 시 오류 발생 */
    // "noImplicitAny": true,
    /* 엄격한 null 검사 사용 */
    // "strictNullChecks": true,
    /* 엄격한 함수 유형 검사 사용 */
    // "strictFunctionTypes": true,
    /* 엄격한 'bind', 'call', 'apply' 함수 메서드 사용 */
    // "strictBindCallApply": true,
    /* 클래스에서 속성 초기화 엄격 검사 사용 */
    // "strictPropertyInitialization": true,
    /* 명시적이지 않은 'any'유형으로 'this' 표현식 사용 시 오류 발생 */
    // "noImplicitThis": true,
    /* 엄격모드에서 구문 분석 후, 각 소스 파일에 "use strict" 코드를 출력 */
    // "alwaysStrict": true,
  }
}
```

<br>

## 3 | 코드 생성과 타입이 관계없음을 이해하기

> - 타입 오류가 있는 코드도 컴파일이 가능하다.
>   - 엉성해 보일 수 있지만 오류에 유연한 대처가 가능하다.
> - 런타임에는 타입 체크가 불가능하다.
>   - 타입은 값으로 사용할 수 없다.
>   - 런타임에 타입을 접근하기 위한 방법
>     - 속성 체크
>     - 태그 기법
>     - 클래스
> - 타입 연산은 런타임에 영향을 주지 않는다.
> - 런타임 타입은 선언된 타입과 다를 수 있다.
> - TS 타입으로는 함수를 오버로드 할 수 없다.
> - TS 타입은 런타임 성능에 영향을 주지 않는다.

<br>

### TS 컴파일러는 두 가지 역할을 하며 완벽히 독립적으로 동작한다

1. TS를 구버전의 JS로 `transpile`

> `transplie`이란?
>
> `tanslate`와 `compile`이 합쳐진 신조어
>
> 번역하고자 하는 소스코드를 동일한 동작을 하는 다른 형태의 소스코드로 변환하는 행위를 뜻한다.
>
> 결과물이 여전히 컴파일되어야 하는 소스코드이기 때문에 컴파일과는 구분해서 부른다.

2. 코드의 타입오류 체크

<br>

## 4 | 구조적 타이핑에 익숙해지기

### JS는 덕 타이핑 기반언어이고 TS는 이를 모델링하기 위해 구조적 타이핑을 사용한다

> 봉인된 타입: **선언된 속성만**을 가지는 타입

- 특정 인터페이스에 할당 가능한 값이라면 타입 선언에 명시적으로 나열된 속성들을 가지고 있다.
- 이때 타입은 봉인되어 있지 않다.

<br>

### 클래스도 구조적 타이핑 규칙을 다른다

```ts
class C {
  constructor(public foo: string) {}
}

const c = new C('instance of C');
const d: C = { foo: 'object literal' };
```

`d`가 `C`타입에 할당되는 이유는 구조적 타이핑의 특징인 속성이 일치하기 때문이다.

<br>

### 구조적 타이핑을 사용하면 유닛 테스팅을 보다 쉽게 할 수 있다

테스트는 모킹한 타입을 받아오는 것보다 구체적인 인터페이스를 정의해서 사용할 수 있으므로, 테스팅 환경에서는 더 간편하게 타입을 알 수 있고 TS가 동작을 예측할 수 있게 한다.

<br>

## 05 | `any`타입 지양하기

> - `any`타입을 사용하면 타입 체커와 TS 언어 서비스를 무력화시켜버린다.
> - `any`타입의 단점
>   - 코드 리팩터링 때 버그를 감춘다.
>   - 타입 설계를 감춘다.
>   - 타입 시스템의 신뢰도를 낮춘다.
> - 최대한 사용을 지양해야 한다.

<br>

### TS의 타입 시스템은 점진적이고 선택적이다

> `any`가 아래 기능의 핵심

- 점진적: 코드에 타입을 조금씩 추가할 수 있기 때문
- 선택적: 언제든지 타입 체커를 해제할 수 있기 때문

<br>

### `any`는 타입 안정성이 없다

```ts
let age: number;
age = '12'; // '"12"' 형식은 'number'형식에 할당할 수 없다.

age = '12' as any; // OK

// age가 number이기를 기대한다면 13이 정상적인 결과
age += 1; // "121"

// 하지만 중간에 any로 타입 단언을 해버려서 TS는 에러를 발생시키지 않는다.
```

즉 타입을 예측할 수 없게 된다.

<br>

### `any`는 함수 시그니처를 무시해 버린다

```ts
function calculateAge(birthDate: Date): number {
  // ...
}

let birthDate: any = '1990-01-01';

// birthDate에 Date가 전달되기를 기대하지만 string이 바인딩되어 있는 any타입의
// birthDate가 넘어오게 되며 런타임시점에는 에러가 발생하겠지만
// TS는 오류를 잡지 못함
calculateAge(birthDate); // 정상
```

<br>

### `any`는 언어 서비스가 적용되지 않는다

> TS 언어 서비스는 자동완성 기능과 적절한 도움말을 제공한다.

하지만 `any`를 사용하면 어떤 서비스도 지원받지 못한다.

예를 들어 이름 변경 기능이 있는데 `Rename Symbol`로 특정 심벌 속성의 이름을 한번에 바꾸려 할 때 `any`타입은 어떤 지원도 받지 못한다.

```ts
interface Person {
  firstName: string;
  last: string;
}

const formatName = (p: Person) => p.firstName + ' ' + p.last;
const formatNameAny = (p: any) => p.first + ' ' + p.last;
```

TS의 모토는 **확장 가능한 JS**이다, 이 부분을 충족시키기 위해 TS 경험의 핵심 요소는 언어 서비스로서 언어 서비스를 제대로 누려야 개발자 자신과 동료의 생산성이 향상된다.

<br>

## 6 | 편집기를 사용하여 타입 시스템 탐색하기

> - 편집기에서 TS 언어 서비스를 적극 활용해야 한다.
> - 편집기를 사용하면 어떻게 타입시스템이 동작하는지, 그리고 TS가 어떻게 타입을 추론하는지 개념을 잡을 수 있다.
> - TS가 동작을 어떻게 모델링하는지 알기 위해 타입 선언 파일(`.d.ts`)을 찾아보는 방법을 터득해야 한다.

<br>

## 7 | 타입의 값들의 집합이라고 생각하기

> - 타입을 값의 집합으로 생각하면 이해하기 편하다(범위의 개념)
> - 하나의 집합은 유한(`boolean` || `literal`)하거나 무한(`string` || `number`)하다.
> - TS의 타입은 엄격한 상속 관계가 아니라 겹쳐지는 집합(벤 다이어그램)으로 표현된다.
> - 두 타입은 서로 서브타입이 아니면서 겹쳐질 수 있다.
> - 한 객체의 추가적인 속성이 타입 선언에 언급되지 않더라도 그 타입에 속할 수 있다
> - 구조적 타이핑
> - 타입 연산은 집합의 범위에 적용된다.
> - A와 B의 Intersection은 A의 범위와 B의 범위의 Intersection이다.
> - 객체 타입에서는 A & B인 값이 A와 B의 속성을 모두 가짐을 의미한다.
> - A는 B를 상속, A는 B에 할당 가능, A는 B의 서브타입 === A는 B의 부분 집합

<br>

## 8 | 타입 공간과 값 공간의 심벌 구분하기

> - TS 코드를 읽을 때 타입인지 값인지 구분하는 방법을 터득해야 한다.
> - 모든 값은 타입을 가지지만, 타입은 값을 가지지 않는다.
> - `type`, `interface`같은 키워드는 타입 공간에만 존재한다.
> - `class`나 `enum` 키워드는 타입과 값 두 가지 모두로 사용 된다.
> - `typeof`, `this` 그리고 많은 다른 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 수 있다.

타입의 속성을 얻을 때에는 `[]`대괄호 접근법을 사용해야 한다.

```ts
interface Person {
  firstName: string;
  lastName: string;
  age: number;
}

const p: Person = {
  firstName: 'Son',
  lastName: 'Wonjae',
  age: 27,
};

const name: Person['firstName' | 'lastName'] = p.firstName;
const age: Person['age'] = p.age;
```

<br>

### 값과 타입 공간에서 다른 의미를 가지는 코드 패턴

**this**

값: JS의 `this`키워드

타입: 일명 다형성(polymorphic) `this`라고 불리는 `this`의 TS 타입

`**&` | `|`\*\*

값: `AND`, `OR`의 비트 연산자

타입: `Intersection`, `Union`

<br>

**extends**

값: 확장자

타입: `class A extends B`, `interface A extends B`, `Generic<T extends number>`

### TS 구조분해 할당

```ts
interface Email {
  person: Person;
  subject: string;
  body: string;
}

function email({ person, subject, body }: Email) {}
```

<br>

## 9 | 타입 단언보다는 타입 선언을 사용하기

> - 타입 단언(`as Type`)보다 타입 선언(`: Type`)을 사용해야 한다.
> - 화살표 함수의 반환 타입을 명시하는 방법을 터득해야 한다.
> - TS보다 타입 정보를 더 잘 알고 있는 상황에서는 타입 단언문과 null이 아님 단언문(`접미사 !`)을 사용하면 된다.

<br>

TS에서 변수에 값을 할당하고 타입을 부여하는 방법은 두가지이다.

```tsx
interface Person {
  name: string;
}

const alice: Person = { name: 'Alice' };
const bob = { name: 'Bob' } as Person;
```

1. 타입 선언

   - 변수에 타입 선언을 붙여 선언된 타입임을 명시

2. 타입 단언
   - TS가 추론한 타입이 있더라도 타입 단언을 따름
   - 즉 타입 체커에게 오류를 무시하라고 하는 것

타입 단언이 꼭 필요한 경우가 아니라면, 안전성 체크도 가능한 타입 선언을 사용하는 것이 좋다.

<br>

### 타입 단언이 필요한 경우

타입 단언은 타입체커가 추론한 타입보다 개발자가 판단하는 타입이 더 정확할 때 의미가 있다.

- DOM 엘리먼트에 대해서는 TS보다 개발자가 더 정확히 알고 있다.

```ts
document.querySelector('#myButton').addEventListener('click', (e) => {
  e.currentTarget; // 타입은 EventTarget
  const button = e.currentTarget as HTMLButtonElement;
  button; // 타입은 HTMLButtonElement
});
```

- TS는 DOM에 접근할 수 없기 때문에 `#mybutton`이 버튼 엘리먼트인지 알지 못한다.
- 이벤트의 `currentTarget`이 같은 버튼이어야 하는 것도 알지 못한다.

<br>

## 10 | 객체 래퍼 타입 피하기

> - 기본형 값에 메서드를 제공하기 위해 객체 래퍼 타입이 어떻게 쓰이는지 이해해야한다.
> - 직접 사용하거나 인스턴스를 생성하는 것은 피해야 한다.
> - TS 객체 래퍼 타입은 지양하고 기본형 타입을 사용해야 한다.
