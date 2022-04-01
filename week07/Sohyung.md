# Item 41 ~ 45

## 들어가며

이펙티브 타입스크립트의 5장 any 다루기와 6장 타입 선언과 @types에 포함된 Item 41~45 내용을 정리했습니다.

## Item 41 any의 진화를 이해하기

> **요약**
>
> - 일반적인 타입들은 변수를 선언할 때 결정된다. 반면, `암시적 any와 any[] 타입은 진화`할 수 있다.
>   any를 진화시키는 방법보다 명시적 타입 구문을 사용하는 것이 안전하다.

타입의 진화는

- noImplicitAny가 설정된 상태에서
- 변수의 타입이 암시적 any인 경우에
- 새로운 값을 할당하거나 배열에 요소를 넣은 후
  일어난다.

암시적 any 진화 예시

```ts
function range(start: number, limit: number) {
  const out = []; // any[]

  for (let i = start; i < limit; i++) {
    out.push(i); // any[]
  }
  return out; // number[]  진화!
}
```

명시적으로 any를 사용하였을 때

```ts
let val: any;
if (Math.random() < 0.5) {
  val = /hello/;
  val; // any
} else {
  val = 12;
  val; // any
}
val; // any
```

암시적 any 상태일 때 값을 읽으려 하면 오류 발생

```ts
function range(start: number, limit: number) {
  const out = []; // any[]
  // ~~~ 'out' 변수는 형식을 확인할 수 없는 경우
  // 일부 위치에서 암시적으로 'any[]' 형식입니다.
  if (start === limit) {
    return out;
    // ~~~ 'out' 변수에는 암시적으로 'any[]' 형식이 포함됩니다.
  }
  for (let i = start; i < limit; i++) {
    out.push(i);
  }
  return out;
}
```

암시적 any 타입은 함수 호출을 거쳐도 진화하지 않는다.

```ts
function makeSquares(start: number, limit: number) {
  const out = []; // ~~~ 'out' 변수는 일부 위치에서 암시적으로 'any[]'형식입니다.
  range(start, limit).forEach(i => {
    out.push(i * i);
  });
  // map, filter 등 단일 구문으로 배열을 생성하여 any 전체를 진화시키는 방법은 가능
  return out; // ~~~ 'out'변수에는 암시적으로 'any[]' 형식이 포함됩니다.
}
```

## Item 42 모르는 타입의 값에는 any 대신 unknown을 사용하기

> **요약**
>
> - unknown은 any 대신 사용할 수 있는 안전한 타입이다. 어떠한 값이 있지만 그 타입을 알지 못하는 경우에 unknown 사용
> - 사용자가 타입 단언문이나 타입 체크를 사용하도록 강제하려면 unknown을 사용하면 된다.
> - {}, object, unknown의 차이점을 이해해야 한다.

**any vs unknown (vs never)**

- 어떠한 타입이든 any 타입에 할당 가능하다. (any, unknown)
- any 타입은 어떠한 타입으로도 할당 가능하다. (any)

**권장사항**
unknown을 반환하고 사용자가 직접 단언문을 사용하거나 원하는 대로 타입을 좁히도록 강제하는 것이 좋다.

타입을 모르는 경우 any를 사용하지 말고 unknown을 사용하라.

**unknown과 {}**

- {} 타입은 null과 undefined를 제외한 모든 값을 포함한다.
- object 타입은 모든 비기본형 타입으로 이루어진다. 여기에는 true 또는 12 또는 "foo"가 포함되지 않지만 객체와 배열은 포함된다.

## Item 43 몽키 패치보다는 안전한 타입을 사용하기

> **요약**
>
> - 전역 변수나 DOM에 데이터를 저장하지 말고, 데이터를 분리하여 사용해야 한다.
> - 내장 타입에 데이터를 저장해야 하는 경우, 안전한 타입 접근법 중 하나(보강, 사용자 정의 인터페이스로 단언)를 사용해야 한다.
> - 보강의 모듈 영역 문제를 이해해야 한다.

window, document에 값을 할당해 전역 변수를 만들면 다양한 문제가 있다.

- 멀리 떨어진 변수들 간에 의존성을 만들어 부수 효과를 고려해야 한다.
- TS 사용시 타입 체커는 Document, HTMLElement의 내장 속성만 알고 있다.

가장 좋은 방법은 DOM으로부터 데이터를 분리하는 것!

하지만 불가피하다면

- interface 보강(augmentation) 사용

```ts
interface Document {
  monkey: string;
}
document.monkey = 'Tamarin'; // 정상
```

보강시 주의사항
전역 스코프이기 때문에 코드의 다른 부분이나 라이브러리로부터 분리할 수 없다.
실행되는 동안 속성을 할당하면 실행 시점에서 보강을 적용할 방법이 없다.

- 구체적인 타입 단언문 사용

```ts
interface MonkeyDocument extends Document {
  monkey: string;
}
(document as MonkeyDocument).monkey = 'Macaque';
```

## Item 44 타입 커버리지를 추적하여 타입 안전성 유지하기

> **요약**
>
> - noImplicitAny가 설정되어 있어도, 명시적 any 또는 서드파티 타입 선언(@types)을 통해 any 타입은 코드 내에 옂ㄴ히 존재할 수 있다는 점을 주의해야 한다.
> - 작성한 프로그램의 타입이 얼마나 잘 선언되었는지 추적해야 한다. 이를 통해 any의 사용을 줄여 나갈 수 있고 타입 안전성을 꾸준히 높일 수 있다.

# 6장 타입 선언과 @types

타입스크립트의 의존성에 관한 개념을 잡을 수 있는 단원

## Item 45 devDependencies에 typescript와 @types 추가하기

> **요약**
>
> - 타입스크립트를 시스템 레벨로 설치하면 안 됩니다. 타입스크립트를 프로젝트의 devDependencies에 포함시키고 팀원 모두가 동일한 버전을 사용하도록 해야 합니다.
> - @types 의존성은 dependencies가 아니라 devDependencies에 포함시켜야 합니다. 런타임에 @types가 필요한 경우라면 별도의 작업이 필요할 수 있습니다.

### npm

npm은 자바스크립트 라이브러리 저장소와 프로젝트가 의존하고 있는 라이브러리들의 버전을 지정하는 방법(package.json)을 제공한다.

#### 구분해 관리되는 npm의 세 가지 의존성

- dependencies : 프로젝트를 실행해는데 필요
- devDependencies : 개발하고 테스트하는데 사용되자만 런타임에는 필요 없는 라이브러리들. npm에 공개하여 다른 사용자가 프로젝트를 설치하면 이 곳에 포함된 라이브러리들은 제외된다.
- peerDependencies : 런타임에 필요하지만 의존성을 직접 관리하지 않는 라이브러리들

### TS 프로젝트에서 고려할 의존성 두 가지

- devDependencies에 ts 넣기 : 팀원들의 동일한 버전 보장
- 타입 의존성(@types)고려
