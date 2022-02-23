# 타입스크립트와 자바스크립트의 관계

타입스크립트는 자바스크립트의 상위집합(superset)이다.  
그렇기에 자바스크립트로 작성한 main.js 파일명을 main.ts로 바꾼다고 해도 달라지는 것은 없다.

이러한 특성은 기존에 존재하는 자바스크립트 코드를 타입스크립트로 마이그레이션하는 데 엄청난 이점이 된다.  
기존 코드를 그대로 유지하면서 일부분에만 타입스크립트 적용이 가능하기 때문이다.

모든 자바스크립트 프로그램은 타입스크립트이지만, 타입스크립트 프로그램이지만 자바스크립트가 아닌 프로그램이 존재한다.  
이는 타입스크립트가 타입을 명시하는 추가적인 문법을 가지기 때문이다.

타입스크립트 컴파일러는 타입스크립트뿐만 아니라 일반 자바스크립트 프로그램에도 유용하다.

```javascript
let city = "new york city";
console.log(city.toUppercase());
// 'toUppercase' 속성이 'string' 형식에 없습니다.
// 'toUpperCase'를 사용하시겠습니까?
```

- city 변수가 문자열이라는 것을 알려 주지 않아도 타입스크립트는 초깃값으로부터 타입을 추론한다.
- 타입 시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다.
- 타입스크립트가 정적 타입 시스템이라는 것은 바로 이런 특징을 말하는 것이다.

타입스크립트는 자바스크립트 런타임 동작을 모델링하는 타입 시스템을 가지고 있기 때문에 런타임 오류를 발생시키는 코드를 찾아내려고 한다.  
그러나 타입 체커를 통과하면서도 런타임 오류를 발생시키는 코드는 충분히 존재할 수 있기에 모든 오류를 찾아내리라 기대해서는 안 된다.  
잘못된 매개변수 개수로 함수를 호출하는 경우처럼, 자바스크립트에서는 허용되지만 타입스크립트에서는 문제가 되는 경우도 존재한다. 이러한 문법의 엄격함은 온전히 취향의 차이이며 우열을 가릴 수 없는 문제이다.

# 타입스크립트 설정 이해하기

다음 코드가 오류 없이 타입 체커를 통과할 수 있을까?

```typescript
function add(a, b) {
  return a + b;
}
add(10, null);
```

정답은 '타입스크립트 컴파일러 설정에 따라 다르다'이다.  
타입스크립트는 어떻게 설정하느냐에 따라 완전히 다른 언어처럼 느껴질 수 있다.

설정을 제대로 사용하기 위해서는 noImplicitAny와 strictNullChecks를 이해해야 한다.

### noImplicitAny

변수들이 미리 정의된 타입을 가져야 하는지 여부를 제어한다.  
매개변수에 타입을 미리 정의하지 않으면 타입스크립트는 암묵적으로 any 타입으로 추론한다.  
any 타입을 매개변수에 사용하면 타입 체커는 무력해진다. any는 유용하지만 주의해서 사용해야 한다.

타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에, 되도록이면 noImplicitAny를 설정해야 한다.  
그러면 타입스크립트가 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지며, 개발자의 생산성이 향상된다.  
noImplicitAny 설정 해제는, 자바스크립트로 되어 있는 기존 프로젝트를 타입스크립트로 전환하는 상황에만 필요하다.

### strictNullChecks

null과 undefined가 모든 타입에서 허용되는지 확인하는 설정이다.  
이 설정을 추가하면 null과 undefined를 사용하는 경우 오류를 발생시킨다.

null을 허용하려고 한다면 의도를 명시적으로 드러냄으로써 오류를 고칠 수 있다.

```typescript
const x: number | null = null;
```

만약 null을 허용하지 않으려면 이 값이 어디서부터 왔는지 찾아야 하고, null을 체크하는 코드나 단언문을 추가해야 한다.

```typescript
const el = document.getElementById("status");

if (el) {
  el.textContent = "Ready";
}
el!.textContent = "Ready";
```

이 옵션을 설정한다면 "undefined는 객체가 아닙니다"와 같은 런타임 오류를 방지할 수 있다.

# 코드 생성과 타입은 관계없음

타입스크립트 컴파일러는 두 가지 역할을 수행한다.

1. 최신 자바스크립트/타입스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일한다.
2. 코드의 타입 오류를 체크한다.

이 두 가지는 서로 독립적으로 실행한다. 즉, 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 주지 않는다. 또한 그 자바스크립트의 실행 시점에도 타입은 영향을 미치지 않는다.

## 타입 오류가 있는 코드도 컴파일이 가능하다

컴파일은 타입 체크와 독립적으로 동작하기 때문에, 타입 오류가 있는 코드도 컴파일이 가능하다.  
타입스크립트 오류는 언어들의 경고와 비슷하다. 문제가 될 만한 부분을 알려 주지만, 그렇다고 빌드를 멈추지는 않는다.  
-> 엄밀히 말하면 코드 생성만이 컴파일이라고 할 수 있기 때문에 코드에 오류가 있을 때 '타입 체크에 문제가 있다'라고 말하는 것이 더 정확한 표현이다.

## 런타임에는 타입 체크가 불가능하다

```typescript
interface Square {
  width: number;
}

interface Rectangle extends Square {
  height: number;
}

type Shape = Square | Rectangle;

function caculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

- instance 체크는 런타임에 일어나지만, Rectangle은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다.
- 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 그냥 제거되어 버린다.
- 위 코드를 의도한대로 동작시키기 위해서는 런타임에 타입 정보를 유지하는 방법이 필요하다.

### 속성 체크 방법

```typescript
function caculateArea(shape: Shape) {
  if ("height" in shape) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

- 속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시 shape 타입을 Rectangle로 보정해 주기 때문에 오류가 사라진다.

### 태그 기법

```typescript
interface Square {
  kind: "square";
  width: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

type Shape = Square | Rectangle;

function caculateArea(shape: Shape) {
  if (shape.kind === "rectangle") {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

- Shape 타입은 태그된 유니온의 한 예이다. 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기 때문에, 타입스크립트에서 흔하게 볼 수 있다.

### 클래스 사용

```typescript
class Square {
  constructor(public width: number) {}
}

class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}

type Shape = Square | Rectangle;

function caculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

- 인터페이스는 타입으로만 사용 가능하지만, 타입을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없다.

## 타입 연산은 런타임에 영향을 주지 않는다

```typescript
function asNumber(val: number | string): number {
  return val as number;
}
```

- 이 코드는 타입 체커를 통과하지만 잘못된 방법이다.
- as number는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다.
- 값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 자바스크립트 연산을 통해 변환을 수행해야 한다.

```typescript
function asNumber(val: number | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

## 런타임 타입은 선언된 타입과 다를 수 있다

- 예를 들어, 네트워크 호출로부터 받아온 값으로 함수를 실행하는 경우 값의 타입을 보장할 수 없다.
- 타입스크립트에서 런타임 타입과 선언된 타입은 맞지 않을 수 있기에 타입이 달라지는 혼란스러운 상황은 가능한 한 피해야 한다.
- 선언된 타입은 언제든지 달라질 수 있다!

## 타입스크립트 타입으로는 함수를 오버로드할 수 없다

- 동일한 이름에 매개변수만 다른 여러 버전의 함수를 함수 오버로딩이라고 한다.
- 타입스크립트에서는 타입과 런타임의 동작이 무관하기 때문에 함수 오버로딩은 불가능하다.
- 타입스크립트가 함수 오버로딩 기능을 지원하기는 하지만, 온전히 타입 수준에서만 동작한다. 즉, 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만, 구현체는 오직 하나뿐이다.

## 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다

- 타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에 런타임의 성능에 아무런 영향을 주지 않는다.

# 구조적 타이핑에 익숙해지기

자바스크립트는 본질적으로 덕 타이핑 기반이다.

**덕 타이핑**  
객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식

타입스크립트는 이런 동작, 즉 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 동작을 그대로 모델링한다.

```typescript
interface Vector2D {
  x: number;
  y: number;
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { name: "HI", x: 3, y: 4 };

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

calculateLength(v); // 정상 출력
```

- NamedVector의 구조가 Vector2D와 호환되기 때문에 함수 호출이 가능한데, 여기서 구조적 타이핑이라는 용어가 사용된다.
- 반대로 구조적 타이핑으로 인해 예상치 못한 오류가 발생할 수도 있다.
- 함수를 작성할 때, 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질 거라 생각하기 쉽다. 이러한 타입은 봉인된 또는 정확한 타입이라고 불리며, 타입스크립트 시스템에서는 표현할 수 없다. 즉, 타입은 열려있다.

```typescript
class C {
  foo: string;
  constructor(foo: string) {
    this.foo = foo;
  }
}

const c = new C("instance of C");
const d: C = { foo: "object literal" }; // 정상
```

- 클래스에서도 구조적 타이핑이 사용되는데, d는 string 타입의 foo 속성을 가지고 생성자(Object.prototype으로 비롯된)를 가지기에 타입이 호환된다.
- 만약 C의 생성자에 단순 할당이 아닌 연산 로직이 존재한다면, d의 경우는 생성자를 실행하지 않으므로 문제가 발생하게 된다.
- 구조적 타이핑을 사용하면 유닛 테스트를 손쉽게 할 수 있다.

# any 타입 지양하기

타입스크립트의 타입 시스템은 코드에 타입을 조금씩 추가할 수 있기 때문에 점진적이며, 언제든지 타입 체커를 해제할 수 있기 때문에 선택적이다.

일부 특별한 경우를 제외하고는 any를 사용하면 타입 스크립트의 수많은 장점을 누릴 수 없게 된다.

## any 타입의 위험성

1. any 타입에는 타입 안전성이 없다.
2. any는 함수 시그니처를 무시해 버린다.
3. any 타입에는 언어 서비스가 적용되지 않는다.
4. 코드 리팩터링 때 버그를 감춘다.
5. 타입 설계를 감춘다.
6. 타입시스템의 신뢰도를 떨어트린다.

# 편집기를 사용한 타입 시스템 탐색

타입스크립트를 설치하면, 두 가지를 실행할 수 있다.

1. 타입스크립트 컴파일러
2. 단독으로 실행할 수 있는 타입스크립트 서버

여기서 타입스크립트 서버는 언어 서비스를 제공한다.  
언어 서비스에서는 자동 완성, 명세 검사, 검색, 리팩터링 등의 기능을 제공한다.  
그리고 편집기는 타입스크립트가 언제 타입 추론을 수행할 수 있는지에 대한 개념을 잡게 해 준다.  
추론 정보는 디버깅하는 데에 큰 도움을 준다.

# 타입이 값들의 집합이라고 생각하기

타입스크립트가 오류를 체크하는 순간에는 타입을 가지고 있다.  
타입은 '할당 가능한 값들의 집합'이라고 말할 수 있다.  
예를 들어, 42와 36.5는 number 타입에 해당되고, 'hi'는 그렇지 않다.

```typescript
interface Person {
  name: string;
}

interface Lifespan {
  birth: Date;
  death?: Date;
}

type PersonSpan = Person & Lifespan;
```

- & 연산자는 두 타입의 인터섹션(교집합)을 계산한다.
- 언뜻 보기에는 공통 속성이 없기 때문에, PersonSpan 타입을 공집합으로 예상하기 쉽다.
- 그러나 타입 연산자는 인터페이스의 속성이 아닌, 값의 집합에 적용된다.
- 그리고 추가적인 속성을 가지는 값도 그 타입에 속하기에 Person과 Lifespan의 속성을 모두 구현한 값은 인터섹션 타입에 속한다.

조금 더 일반적으로 PersonSpan 타입을 선언하는 방법은 extends 키워드를 쓰는 것이다.

```typescript
interface Person {
  name: string;
}

interface PersonSpan extends Person {
  birth: Date;
  death?: Date;
}
```

- 타입이 집합이라는 관점에서 extends의 의미는 '...에 할당 가능한'과 비슷하게, '...의 부분집합'이라는 의미로 받아들일 수 있다.
- extends 키워드는 제네릭 타입에서 한정자로도 쓰이며, 이 문맥에서는 '...의 부분집합'을 의미하기도 한다.

## 타입스크립트 용어와 집합 이론 용어 사이 대응 관계

- never -> 공집합
- 리터럴 타입 -> 원소가 1개인 집합
- 값이 T에 할당 가능 -> 값이 T의 원소
- T1이 T2에 할당 가능 -> T1이 T2의 부분 집합
- T1이 T2를 상속 -> T1이 T2의 부분 집합
- `T1 | T2` -> T1과 T2의 합집합
- `T1 & T2` -> T1과 T2의 교집합
- unknown -> 전체 집합

### 추가

- 타입스크립트 타입이 되지 못하는 값의 집합들도 존재한다.
- 정수에 대한 타입, 또는 x와 y 속성 외에 다른 속성이 없는 객체는 타입스크립트 타입에 존재하지 않는다.
- 'A는 B를 상속', 'A는 B에 할당 가능', 'A는 B의 서브 타입'은 'A는 B의 부분 집합'과 같은 의미이다.

# 타입 공간과 값 공간의 심벌 구분

타입스크립트의 심벌은 타입 공간이나 값 공간 중 한 곳에 존재  
심벌 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있다.

```typescript
interface Cylinder {
  radius: number;
  height: number;
}

const Cylinder = (radius: number, height: number) => ({ radius, height }); // 정상
```

- interface Cylinder에서 Cylinder는 타입으로 쓰인다.
- const Cylinder에서 이름은 같지만 값으로 쓰이며, 서로 아무런 관련도 없다.
- 이런 점이 가끔 오류를 야기한다. 다만 어떤 형태로 쓰이는지 문맥을 살펴 알아내야 하고 언뜻 봐서 알 수 없다.

```typescript
type T1 = "string literal";
const v1 = "string literal";
```

- 일반적으로 type이나 interface 다음에 나오는 심벌은 타입인 반면, const나 let 선언에 쓰이는 것은 값이다.
- 컴파일 과정에서 타입 정보는 제거되기 때문에, 심벌이 사라진다면 그것은 타입에 해당될 것이다.

## 상황에 따라 타입과 값 두 가지 모두 가능한 경우

### class

클래스가 타입으로 쓰일 때는 형태(속성, 메서드)가 사용되는 반면, 값으로 쓰일 때는 생성자가 사용된다.

### typeof

```typescript
const p: Person = { ... };
type T1 = typeof p; // 타입은 Person
const v1 = typeof p; // 값은 'object'
```

- 타입의 관점에서, typeof는 값을 읽어서 타입스크립트 타입을 반환한다.
- 값의 관점에서 typeof는 자바스크립트 런타임의 typeof 연산자가 된다.

## 속성 접근자가 타입으로 쓰일 경우

- 속성 접근자인 []는 타입으로 쓰일 때에도 동일하게 동작한다.
- 그러나 대괄호 접근자와 마침표 접근자는 값이 동일하더라도 타입은 다를 수 있다.
- 따라서 타입의 속성을 얻을 때에는 반드시 대괄호 접근자를 사용해야 한다.

```typescript
const first: Person["first"] = p["first"]; // 값으로는 p.first도 가능
```

- `Person['first']`는 타입 맥락에 쓰였기 때문에 타입이다.
- 인덱스 위치에는 유니온 타입과 기본형 타입을 포함한 어떠한 타입이든 사용 가능하다.

```typescript
type PersonEl = Person["first" | "last"];
```

## 두 공간 사이에서 다른 의미를 가지는 코드 패턴

- 값으로 쓰이는 this는 자바스크립트의 this 키워드이다. 타입으로 쓰이는 this는, 일명 다형성 this라고 불리는 this의 타입스크립트 타입이다. 서브클래스의 메서드 체인을 구현할 때 유용하다.
- 값에서 `&`와 `|`는 비트 연산이다. 타입에서는 인터섹션과 유니온이다.
- const는 새 변수를 선언하지만, as const는 리터럴 또는 표현식의 추론된 타입을 바꾼다.
- extends는 서브클래스 또는 서브타입 또는 제너릭 타입의 한정자를 정의할 수 있다.
- in은 루프 또는 매핑된(mapped) 타입에 등장한다.

## 타입스크립트에서 구조 분해 할당

타입스크립트에서 구조 분해 할당을 하면 이상한 오류가 발생한다.

```typescript
function Email({ person: Person, subject: string, body: string });
// 'Person'에 암시적으로 'any' 형식이 있습니다.
// 'string' 식별자가 중복되었습니다.
// 'string'에 암시적으로 'any' 형식이 있습니다.
```

- 값의 관점에서 Person과 string이 해석되었기 때문에, 즉 구조 분해 할당 시 별칭 값으로 해석하여 오류가 발생한다.
- 문제를 해결하려면 타입과 값을 구분해야 한다.

```typescript
function email({
  person,
  subject,
  body,
}: {
  person: Person;
  subject: string;
  body: string;
}) {
  // ...
}
```

- 이 코드는 장황하지만 매개변수에 명명된 타입을 사용하거나 문맥에서 추론되도록 잘 동작한다.

# 타입 단언보다는 타입 선언 사용하기

타입 스크립트에서 변수에 값을 할당하고 타입을 부여하는 방법은 두 가지, 타입 선언과 타입 단언이다.  
타입 단언보다 타입 선언을 사용하는 것이 좋다.

그 이유는,

- 타입 선언은 할당되는 값이 해당 인터페이스를 만족하는지 검사한다.
- 반면 타입 단언은 강제로 타입을 지정했으니 타입 체커에게 오류를 무시하라고 하는 것과 같다.
- 타입 단언이 꼭 필요한 경우가 아니라면, 안정성 체크도 되는 타입 선언을 사용하는 것이 좋다.

## 화살표 함수에서의 타입선언

map API 결과로 Person[] 타입을 반환받고 싶다고 가정해보자

```typescript
interface Person {
  name: string;
}

const people = ["alice", "bob", "jan"].map((name) => ({ name }));
```

이와 같이 코드를 작성하면 타입 추론은 `{ name: string; }[]`로 된다.

이때 타입 단언을 사용하면 타입 체크를 제대로 하지 못해 의도치 않은 오류가 런타임에 발생할 수 있다.

```typescript
interface Person {
  name: string;
}

const people: Person[] = ["alice", "bob", "jan"].map(
  (name): Person => ({ name })
);
```

이와 같이 코드를 작성하면 의도한 대로 Person[] 타입을 추론하게 된다.  
주의할 점은 `(name: Person)`이 아닌 `(name): Person`으로 작성해야 반환 타입이 Person이라고 명시하는 것이다.

## 타입 단언을 사용해야 하는 경우

타입 단언은 타입 체커가 추론한 타입보다 개발자가 판단하는 타입이 더 정확할 때 의미가 있다.  
ex) DOM 엘리먼트 타입 선언 시

- 타입스크립트는 DOM에 접근할 수 없기 때문에 해당 태그가 어떤 엘리먼트인지 알지 못한다.
- 자주 쓰이는 문법 `!`를 사용하여 null이 아님을 단언할 수도 있다.
- 타입 단언문으로 임의의 타입 간에 변환을 할 수는 없다. A가 B의 부분 집합인 경우에 타입 단언문을 사용해 변환할 수 있다.
- 이를 강제로 실행하려면 unknown을 사용해야 한다. 이는 타입 간 변환을 가능케 하지만 위험한 동작을 발생시킬 수 있다.

```typescript
const el = document.getElementById("foo")!; // 타입은 HTMLElement
const el = document.body as unknown as Person;
```

# 객체 래퍼 타입 피하기

- 타입스크립트는 기본형과 객체 래퍼 타입을 별도로 모델링한다.
- 타입스크립트 객체 래퍼 타입은 지양하고, 대신 기본형 타입을 사용해야 한다.
  - string을 매개변수로 받는 메서드에 String 래퍼 객체를 전달하는 순간 문제가 발생한다.
  - string은 String에 할당할 수 있지만 String은 string에 할당할 수 없다.
