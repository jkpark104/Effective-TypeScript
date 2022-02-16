## 노션 링크

https://stump-breadfruit-e3d.notion.site/week01-706215c752e844c1becf585124ff51b2

| 노션으로 보는 것이 더 깔끔합니다! 😎

## 1장

### [아이템1] 타입스크립트와 자바스크립트 관계 이해하기

문법의 유효성과 동작의 이슈는 독립적인 문제이다.

> 즉, 문법이 맞지 않는다고해서 동작이 아예 안되는 것은 아니다.

타입스크립트는 타입 추론을 통해 타입을 알고, 타입 체커가 문제를 찾는다. (string을 추론을 통해 알고, toUpperCase메서드로 추론(소문자 → 대문자)

```tsx
// city의 타입을 string으로 추론하고
let city = 'new york city';
console.log(city.toUppercase()); // 에러 발생 -> toUpperCase()를 의미하는 것인지?
```

타입스크립트에서 타입 구문을 추가한다면 훨씬 더 많은 오류를 찾을 수 있다. → `states` 예시(state의 인터페이스나 타입을 정의함으로써 개발자의 의도대로 사용하게 만듦)

타입스크립트 타입 시스템은 자바스크립트의 런타임 동작을 ‘모델링' 하지만, 의도치 않은 이상한 코드를 감지하기도 함(타입 체커에 의해)

작성된 프로그램이 타입 체크를 통과하더라도 여전히 런타임에 오류가 발생할 수 있다.

```tsx
const names = ['Alice', 'Bob'];
console.log(names[2].toUpperCase());
```

> 앞에서 배열이 범위 내에서 사용될 것이라 가정했지만 그렇지 않아서 문제 발생!

오류의 근본원인은 타입스크립트가 이해하는 값의 타입과 실제 값에 차이가 있기 때문!!(이해하는 값의 타입: name의 문자열 vs 실제 값: undefined)

### [아이템2] 타입스크립트 설정 이해하기

타입스크립트를 제대로 사용하려면 아래의 2개 설정을 최소한이라도 설정하자.

```tsx
"noImplicitAny": true, // 암묵적 any 금지(명시적으로 써줘야함)
"strictNullChecks": true, // null, undefined 타입 허용X
```

→ 쉽게하려면 그냥 `strict` 설정 ㄱㄱ

```tsx
"strict": true,
```

### [아이템3] 코드 생성과 타입이 관계 없음을 이해하기

타입스크립트 컴파일러의 2가지 역할

- 브라우저 동작을 위한, 구버전의 JS로 트랜스파일
- 코드의 타입 오류 체크

> 독립적으로 이루어짐!

타입스크립트가 할 수 있는 일과 할 수 없는 일

- 타입 오류가 있는 코드도 컴파일이 가능(막고 싶다면 noEmitOnError 설정)
- 런타임에 타입 체크 불가(안에 속성확인, ‘태그’ 기법, class로 선언하여 사용)
- 타입 연산은 런타임에 영향 X → 런타임에 타입을 체크해주는 것이 필요(값 정제를 원한다면)
- 런타임 타입과 선언된 타입이 다를 수 있고 이를 명심!
- 타입스크립트 타입으로 함수 오버로드 불가
- 타입스크립트 타입은 런타임 성능에 영향을 주지X → 빌드타임 오버헤드 존재

> **증분 빌드:** 컴파일과 관련한 기록파일을 남겨둠으로써 `tsc`가 필요한 것만 효율적으로 컴파일할 수 있도록 하는 기능

[타입스크립트(typescript) 프로젝트 세팅하기](https://elvanov.com/2524)

### [아이템4] 구조적 타이핑에 익숙해지기

JS가 덕 타이핑 기반(객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주!) 이라 TS또한 이를 그대로 모델링함

→ 그러나 이 때문에 문제가 발생하기도함

1. 정규화 진행시 다른 인터페이스 객체가 들어와도 인터페이스에서 벗어나지 않고 확장된 경우 문제 발생할 수 있음
2. 마찬가지로 Vector3D 인터페이스에 string이 들어간 인터페이스를 확장시 number만 계산하는 함수에서 문제 발생 → 차라리 루프보다 모든 속성을 각각 구하는 방법으로 해결
3. 생성자 또한 클래스 생성자가 반환하는 인스턴스의 객체 타입을 인터페이스로 가진 객체가 있다면 이를 클래스를 타입으로 붙여도 에러가 발생하지 않는다.
   `const d: C = { foo: ‘object literal’ } // OK!`
   `console.log(d(일반객체) instanceof C) // false`
4. 오히려 테스트에선 구조적 타이핑이 유리! → 추상화(DB라는 인터페이스)를 통해 로직과 테스트를 특정 구현으로부터 분리

> TS의 타입은 항상 **열려** 있다.

### [아이템5] any 타입 지양하기

any타입을 사용하면 타입 체커, 자동완성 서비스를 무력화하며 개발경험을 나쁘게하고 신뢰도를 떨어트리므로 사용하는 것을 지양하자!

any타입의 문제

- 타입 안전성 없음
- 함수 시그니처 무시(호출하는 쪽에서 약속된 타입의 입력을 제공해야하는데 전달되는 인수가 any타입이면 무시하게 됨)
- 자동완성과 같은 언어 서비스 적용 X
- 코드 리팩토링시 버그 감춤(함수(타입: number)를 인수로 받아 사용하는 함수(타입:any)의 콜백함수 문제)
- 타입설계 감춤
- 타입 시스템의 신뢰도 하락

## 2장

### [아이템6] 편집기를 사용하여 타입 시스템 탐색하기

타입스크립트 설치를 하면 다음 두가지 실행이 가능

- tsc
- 타입스크립트 서버

타입스크립트 서버가 타입 추론을 어떻게 하는지 커서를 갖다대어 확인하는 것은 타입스크립트를 연마하기에 좋은 방법!

→ 이를 통해 타입들을 보면서 라이브러리가 어떻게 모델링 되었는지, 어떻게 오류를 찾아낼지 살펴볼 수 있다. 그러므로 타입 선언 파일을 찾아보면서 공부!

### [아이템7] 타입이 값들의 집합이라고 생각하기

가장 작은 집합: never ⇒ 유닛타입(하나) ⇒ 유니온 타입(여러개)

타입 연산자는 **값의 집합에 적용된다.**

```tsx
interface Person {
  name: string;
}
interface Lifespan {
  birth: Date;
  death?: Date;
}
type PersonSpan = Person & Lifespan; -> name, birth, death 속성을 가지고있음

// 객체타입에서
keyof (A&B) = (keyof A) | (keyof B)
keyof (A|B) = (keyof A) & (keyof B)
```

extends는 부분집합에 해당한다.

Exclude는 일부타입을 제외하는데 사용되지만 그 결과가 적절한 타입스크립트 타입일때 유효!

### [아이템8] 타입 공간과 값 공간의 심벌 구분하기

타입스크립트의 심벌은 **타입 공간이나 값 공간** 중의 한 곳에 존재한다!

심벌이 타입으로 쓰이는지, 값으로 쓰이는지 구분하는 것이 중요!

```tsx
class Cylinder {
  radius = 1;
  height = 1;
}

const calculateVolume = (shape: unknown) => {
  // 값으로 사용된 부분
  if (shape instanceof Cylinder) {
    // 타입으로 사용된 부분
    shape;
    shape.radius;
  }
};
```

위 예제에서 클래스와 이넘같은 경우 타입 혹은 값으로 사용될 수 있음

InstanceType<typeof C>: 생성자 함수의 인스턴스 타입을 타입으로 구성하는 것

[Documentation - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html#instancetypetype)

객체의 프로퍼티 타입에 접근시 대괄호 표기법으로 접근!

✅ p: Person[’first’]

❌ p: Person.first

속성접근자

```tsx
type Tuple = [string, number, Date];
// TupleEl의 타입은 string | number | Date
type TupleEl = Tuple[number]; // number가 0, 1, 2 중 하나만 올 수 있으므로(그 의미로 쓰임) 어려움..
```

타입 공간과 값 공간의 혼동

```tsx
function email({ person: Person, subject: string, body: string }) {}
```

> 이렇게 작성하면 JS의 alias(별칭)으로 판단하여 타입이 값으로 쓰이기 때문에 오류가 발생한다. 따라서 아래와 같이 변경!

```tsx
function email({
  person,
  subject,
  body,
}: {
  person: PresentationStyle;
  subject: string;
  body: string;
}) {}
```

### [아이템9] 타입 단언보다는 타입 선언을 사용하기

타입 단언은 강제로 타입을 지정해서 타입체커에게 오류를 무시하라고 하는 것이기에 타입 선언 사용을 더 지향하자.

잉여 속성 체크(지정된 프로퍼티 이외에 다른 프로퍼티가 있는지 검사) 또한 타입 단언문에선 적용 X

타입 단언은 보통 DOM 엘리먼트를 가져올때 적용한다. (개발자가 더 잘 알고 있는 경우)

> 캡틴 판교는 null 단언문보다 dom에 단언문을 사용하는 것을 권장

```tsx
// like this
function $<T extends HTMLElement>(selector: string) {
  const element = document.querySelector(selector);
  return element as T;
}

const confirmedTotal = $<HTMLSpanElement>('.confirmed-total');
```

### [아이템10] 객체 래퍼 타입 피하기

타입스크립트는 기본형(원시타입)과 객체 래퍼타입을 별도로 모델링하기 때문에 값으로써 사용할 때와 타입으로 사용할 때를 구분하고, 타입으로 사용시 주의해야 한다!

```tsx
// 이렇게 쓰지 말고
const s: String = 'primitive';
x;
// 이렇게 쓰자! (기본형으로)
const s: string = 'prmitivie';
o;
```
