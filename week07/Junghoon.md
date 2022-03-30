# any의 진화를 이해하기

타입스크립트에서 일반적으로 변수의 타입은 변수를 선언할 때 결정된다.  
그 후에 정제될 수 있지만, 새로운 값이 추가되도록 확장할 수는 없다. 그러나 any 타입과 관련해서 예외인 경우가 존재한다.

```typescript
function range(start: number, limit: number) {
  const out = []; // 타입이 any[]

  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out; // 타입이 number[]
}
```

- 처음에는 any 타입 배열인 `[]`로 초기화되었는데, 마지막에는 `number[]`로 추론되고 있다.
- out의 타입은 `any[]`로 선언되었지만 number 타입의 값을 넣는 순간부터 타입은 `number[]`로 진화한다.
- 타입의 진화는 타입 좁히기와 다른다. 배열에 다양한 타입의 요소를 넣으면 배열의 타입이 확장되며 진화한다.
- 또한 조건문에서는 분기에 따라 타입이 변할 수도 있다.

변수의 초깃값이 null인 경우도 any의 진화가 일어난다. 보통은 try/catch 블록 안에서 변수를 할당하는 경우에 나타난다.

```typescript
const result = [];
result.push("a");
result.push(1);

let val = null; // 타입이 any
try {
  val = 12; // 타입이 number
} catch (e) {
  console.warn("alas");
}
val; // 타입이 number | null
```

- any 타입의 진화는 noImplicitAny가 설정된 상태에서 변수의 타입이 암시적 any인 경우에만 일어난다. 그러나 명시적으로 any를 선언하면 타입이 그대로 유지된다.
- 암시적 any 상태인 변수에 어떠한 할당도 하지 않고 사용하려고 하면 암시적 any 오류가 발생하게 된다.
- any 타입의 진화는 암시적 any 타입에 어떤 값을 할당할 때만 발생한다. 그리고 어떤 변수가 암시적 any 상태일 때 값을 읽으려고 하면 오류가 발생한다.
- 암시적 any 타입은 함수 호출을 거쳐도 진화하지 않는다.
- any를 진화시키는 방식보다 명시적인 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

# 모르는 타입의 값에는 any 대신 unknown을 사용하기

## 함수의 반환값과 관련된 unknown

YAML 파서인 parseYAML 함수를 작성한다고 가정해보자

```typescript
function parseYAML(yaml: string): any {
  // ...
}

interface Book {
  name: string;
  author: string;
}

const book = parseYAML(`
name: aksje
author: emeifmn
`);

alert(book.title);
book("read");
```

- 이 경우 함수의 반환값에 타입 선언을 강제할 수 없기 때문에, 호출한 곳에서 타입 선언을 생략하게 되면 book 변수는 암시적 any 타입이 된다.
- 그렇기에 book 변수의 잘못된 프로퍼티에 접근하거나 했을 때, 적절한 경고를 내주지 못하고 런타임에 에러를 발생시킨다.
- 대신 parseYAML이 unknown 타입을 반환하게 만드는 것이 더 안전하다.

타입 체커는 집합 기반이기 때문에 any를 사용하면 타입 체커가 무용지물이 된다는 것을 주의해야 한다.  
unknown은 any 대신 쓸 수 있는 타입 시스템에 부합하는 타입이다.

any는 다음과 같은 두 가지 속성을 가지고 있다.

1. 어떠한 타입이든 any 타입에 할당 가능하다.
2. any 타입은 어떠한 타입으로도 할당 가능하다.

- unknown 타입은 any의 첫 번째 속성을 만족하지만, 두 번째 속성(unknown은 오직 unknown과 any에만 할당 가능)은 만족하지 않는다.
- 반면 never 타입은 첫 번째 속성은 만족하지 않지만, 두 번째 속성은 만족한다.
- unknown 타입인 채로 값을 사용하면 오류가 발생한다. unknown인 값에 함수 호출을 하거나 연산을 하려고 해도 마찬가지이다. unknown 상태로 사용하려고 하면 오류가 발생하기 때문에, 적절한 타입으로 변환하도록 강제할 수 있다.

## 변수 선언과 관련된 unknown

어떠한 값이 있지만 그 타입을 모르는 경우 unknown을 사용한다.  
예를 들어, GeoJSON 사양에서 Feature의 properties 속성은 JSON 직렬화가 가능한 모든 것을 담는 잡동사니 주머니 같은 존재이다.  
그래서 타입을 예상할 수 없기 때문에 unknown을 사용한다.

```typescript
interface Feature {
  id?: string | number;
  geometry: Geometry;
  properties: unknown;
}
```

- 타입 단언문이 unknown에서 원하는 타입으로 변환하는 유일한 방법은 아니다.
- instanceof를 체크한 후 unknown에서 원하는 타입으로 변환할 수 있다.
- 또한 사용자 정의 타입 가드도 unknown에서 원하는 타입으로 변환할 수 있다.

```typescript
function isBook(val: unknown): val is Book {
  return (
    typeof val === "object" && val !== null && "name" in val && "author" in val
  );
}
```

- unknown 타입의 범위를 좁히기 위해서는 많은 노력이 필요하다.
- in 연산자에서 오류를 피하기 위해 먼저 val이 객체임을 확인해야 하고, null이 아님을 확인해야 한다.
- 제너릭을 사용한 스타일은 타입 단언문과 달라 보이지만 기능적으로는 동일하다. 제너릭보다는 unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.

## 단언문과 관련된 unknown

이중 단언문에서 any 대신 unknown을 사용할 수 있다.

```typescript
declare const foo: Foo;
let barAny = foo as any as Bar;
let barUnk = foo as unknown as Bar;
```

- barAny와 barUnk는 기능적으로 동일하지만, 나중에 두 개의 단언문을 분리하는 리팩터링을 한다면 unknown 형태가 더 안전하다.
- any의 경우는 분리되는 순간 그 영향력이 퍼지게 되지만, unknown의 경우는 분리되는 즉시 오류를 발생하게 되므로 더 안전하다.

## unknown, object, {}

unknown과 비슷한 방식으로 object 또는 {}를 사용하는 코드들이 존재한다.  
object 또는 {}를 사용하는 방법 역시 unknown만큼 범위가 넓은 타입이지만, unknown보다는 범위가 약간 좁다.

- {}타입은 null과 undefined를 제외한 모든 값을 포함한다.
- object 타입은 non-primitive 타입으로 이루어진다.
- 정말로 null과 undefined가 불가능하다고 판단되는 경우만 unknown 대신 {}를 사용하면 된다.

# 몽키 패치보다는 안전한 타입 사용하기

자바스크립트 특징 중 하나는, 객체와 클래스에 임의의 속성을 추가할 수 있다는 것이다.  
객체의 속성을 추가할 수 있는 기능은 종종 웹 페이지에서 window나 document에 값을 할당하여 전역 변수를 만드는 데 사용된다.  
또는 DOM 엘리먼트에 데이터를 추가하기 위해서도 사용된다.

사실 객체에 임의의 속성을 추가하는 것은 좋은 설계가 아니다. 예를 들어 window에 데이터를 추가하면 그 데이터는 기본적으로 전역 변수가 된다.  
전역 변수를 사용하면 의도치 않은 부분들 간에 의존성을 만들게 되어 사이드 이펙트를 고려해야 한다.  
타입스크립트를 더하면 또 다른 문제가 발생하는데, 임의로 추가한 속성에 대해서 타입스크립트는 알지 못한다.

이 오류를 해결하는 가장 간단한 방법은 any 단언문을 사용하는 것인데, 이는 타입 안정성을 상실하고, 언어 서비스를 사용할 수 없게 된다는 단점이 존재한다.  
최선의 해결책은 document 또는 DOM으로부터 데이터를 분리하는 것이다.

분리할 수 없는 경우(객체와 데이터가 붙어 있어야만 하는 라이브러리를 사용 중이거나 자바스크립트 애플리케이션을 마이그레이션 하는 과정 중이라면), 두 가지 차선책이 존재한다.

1. interface의 특수 기능 중 하나인 보강을 사용한다.
2. 더 구체적인 타입 단언문을 사용한다.

## 보강

보강을 사용한 방법이 any보다 나은 점은 다음과 같다.

- 타입이 더 안전하여 타입 체커는 오타나 잘못된 타입의 할당을 오류로 표시한다.
- 속성에 주석을 붙일 수 있다.
- 속성에 자동완성을 사용할 수 있다.
- 몽키 패치가 어떤 부분에 적용되었는지 정확한 기록이 남는다.

모듈 관점에서(타입스크립트 파일이 import/export를 사용하는 경우), 제대로 동작하게 하려면 global 선언을 추가해야 한다.

```typescript
export {};

declare global {
  interface Document {
    monkey: string;
  }
}
```

- 보강을 사용할 때 주의할 점은 모듈 영역과 관련이 있다.
- 보강은 전역적으로 적용되기 때문에, 코드의 다른 부분이나 라이브러리로부터 분리할 수 없다.

## 구체적인 타입 단언문

```typescript
interface MonkeyDocument extends Document {
  monkey: string;
}

(document as MonkeyDocument).monkey = "hi";
```

- MonkeyDocument는 Document를 확장하기 때문에 타입 단언문은 정상이며 할당문의 타입은 안전하다.
- 또한 Document 타입을 건드리지 않고 별도로 확장하는 새로운 타입을 도입했기 때문에 모듈 영역 문제도 해결할 수 있다.
- 따라서 몽키 패치된 속성을 참조하는 경우에만 단언문을 사용하거나 새로운 변수를 도입하면 된다.

# 타입 커버리지를 추적하여 타입 안정성 유지하기

noImplicitAny를 설정하고 모든 암시적 any 대신 명시적 타입 구문을 추가해도 any 타입과 관련된 문제들로부터 안전하다고 할 수 없다.  
any 타입이 여전히 프로그램 내에 존재할 수 있는 두 가지 경우가 있다.

- 명시적 any 타입
  - `any[]`와 `{ [key: string]: any] }` 같은 타입은 인덱스를 생성하면 단순 any가 되고 코드 전반에 영향을 미친다.
- 서드파티 타입 선언
  - @types 선언 파일로부터 전파되기 때문에 특별히 조심해야 한다.

any 타입은 타입 안정성과 생산성에 부정적 영향을 미칠 수 있으므로, 프로젝트에서 any의 개수를 추적하는 것이 좋다.  
npm의 `type-coverage` 패키지를 활용하여 any를 추적할 수 있다.

`npx type-coverage`  
`npm type-coverage --detail`

프로젝트에서 any가 아닌 타입의 백분율을 출력해준다. `--detail` 플래그를 붙이면, any 타입이 있는 곳을 모두 출력해 준다.

작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 한다. 추적함으로써 any의 사용을 줄여 나갈 수 있고, 타입 안정성을 꾸준히 높일 수 있다.

# devDependencies에 typescript와 @types 추가하기

## 모든 타입스크립트 프로젝트에서 공통적으로 고려해야 할 의존성 두 가지

### 타입 스크립트 자체 의존성

타입스크립트를 시스템 레벨로 설치할 수도 있지만, 다음 두 가지 이유 때문에 좋지 않다.

- 팀원들 모두가 항상 동일한 버전을 설치한다는 보장이 없다.
- 프로젝스틀 셋업할 때 별도의 단계가 추가된다.

따라서 타입스크립트를 시스템 레벨로 설치하기보다는 devDependencies에 넣는 것이 좋다.

### 타입 의존성(@types)

- 사용하려는 라이브러리에 타입 선언이 포함되어 있지 않더라도, DefinitelyTyped에서 타입 정보를 얻을 수 있다.
- DefinitelyTyped의 타입 정의들은 npm 레지스트리의 @types 스코프에 공개된다.
