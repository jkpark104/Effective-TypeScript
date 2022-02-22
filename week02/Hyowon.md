# 2주차

## 아이템 11 잉여 속성 체크의 한계 인지하기

타입이 명시된 변수에 객체 리터럴을 할당할 때 타입스크립트는 해당 타입의 속성이 있는지, 그리고 **그 외의 속성이 없는지** 확인합니다.

```jsx
interface Room {
  numDoors: number;
  ceilingHeightFt: number;
}
const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present',
// ~~~~~~~~~~~~~~~~~~ Object literal may only specify known properties,
//                    and 'elephant' does not exist in type 'Room'
};
```

구조적 타이핑 관점으로 생각해 보면 오류가 발생하지 않아야 한다.

임시 변수를 도입해 보면 알 수 있는데, obj 객체는 Room타입에 할당이 가능하다.

```jsx
const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present',
};
const r: Room = obj;  // OK
```

obj의 타입은 `{ numDoors: number; ceilingHeightFt: number; elephant: string}`으로 추론된다. 

obj 타입은 Room 타입의 부분 집합을 포함하므로 Room에 할당 가능하며 타입체커도 통과한다.

두 예제의 차이점

첫 번째 예제는 구조적 타입 시스템에서 발생할 수 있는 중요한 종류의 오류를 잡을 수 있도록

**잉여 속성 체크** 라는 과정이 수행되었다.

 

그러나 잉여속성체크 역시 조건에 따라 동작하지 않는다는 한계가있다.

통상적인 할당 가능 검사와 함께 쓰이면 구조적 타이핑이 무엇인지 혼란스러워질 수 있다.

잉여 속성 체크가 할당 가능 검사와는 별도의 과정이라는 것을 알아야 타입스크립트 타입시스템에 대한 개념을 정확히 잡을 수 있다.

```jsx
interface Options {
  title: string;
  darkMode?: boolean;
}
function createWindow(options: Options) {
  if (options.darkMode) {
    setDarkMode();
  }
  // ...
}
createWindow({
  title: 'Spider Solitaire',
  darkmode: true
// ~~~~~~~~~~~~~ Object literal may only specify known properties, but
//               'darkmode' does not exist in type 'Options'.
//               Did you mean to write 'darkMode'?
});
```

```jsx
const o1: Options = document;  // OK
const o2: Options = new HTMLAnchorElement;  // OK
```

Options 타입은 범위가 매우 넓기 때문에, 순수한 구조건 타입 체커는 이런 종류의 오류를 찾아내지 못한다.

darkMode 속성에 boolean타입이 아닌 다른 타입의 값이 지정된 경우를 제외하면

string 타입인 title 속성과 또 다른 어떤 속성을 가지는 모든 객체는 Options 타입의 범위에 속한다.

document 와 HTMLAnchorElement 의 인스턴스 모두  string 타입인 title 속성을 가지고 있기 때문에 할당문은 정상이다.

잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서도 객체 리터럴에 알 수 없는 속성을 허용하지 않음으로써, 앞에서 다룬 Room이나 Options 예제 같은 문제점을 방지할 수 있습니다. → 그래서 **엄격한 객체 리터럴 체크** 라고도 불립니다.

document 나 HTMLAnchorElement는 객체 리터럴이 아니기 때문에 잉여속성체크가 되지 않습니다.

```jsx
const intermediate = { darkmode: true, title: 'Ski Free' };
const o: Options = intermediate;  // OK
```

첫번째 줄의 오른쪽은 객체 리터럴이지만, 두 번째 줄의 오른쪽은 객체리터럴이 아닙니다.

따라서 잉여 속성 체크가 적용되지 않고 오류는 사라진다.

잉여속성체크는 타입 단언문을 사용할 때에도 적용되지 않는다.

```jsx
const o = { darkmode: true, title: 'Ski Free' } as Options;  // OK
```

이 예제가 단언문보다 선언문을 사용해야하는 단적인 이유 중 하나이다.(아이템9)

잉여 속성 체크를 원치 않는다면, **인덱스 시그니처**를 사용해서 타입스크립트가 추가적인 속성을 예상하도록 할 수 있다. → 이런 방법이 데이터를 모델링하는데 적절한지 아닌지에 대해서는 아이템 15에서 다룬다.

```jsx
interface Options {
  darkMode?: boolean;
  [otherOptions: string]: unknown;
}
const o: Options = { darkmode: true };  // OK
```

선택적 속성만 가지는 `weak타입`에도 비슷한 체크가 동작한다.

```jsx
interface LineChartOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
  areaChart?: boolean;
}
const opts = { logScale: true };
const o: LineChartOptions = opts;
   // ~ Type '{ logScale: boolean; }' has no properties in common
   //   with type 'LineChartOptions'
```

구조적 관점에서 LineChartOptions 타입은 모든 속성이 선택적이므로 모든 객체를 포함할 수 있다.

이런 약한 타입에 대해서는 타입스크립트는 값 타입과 선언 타입에 공통된 속성이 있는지 확인하는 별도의 체크를 수행한다.

공통속성체크는 잉여속성체크와 마찬가지로 오타를 잡는 데 효과적이며 **구조적으로 엄격하지 않다(p64)**

그러나 잉여 속성 체크와 다르게 약한 타입과 관련된 할당문마다 수행된다.

임시 변수를 제거하더라도 공통 속성 체크는 여전히 동작한다.

잉여 속성 체크는 구조적 타이핑 시스템에서 허용되는 속성 이름의 오타 같은 실수를 잡는 데 효과적인 방법이다.

선택적 필드를 포함하는 Options 같은 타입에 특히 유용한 반면, 적용 범위도 매우 제한적이며 오직 객체 리터럴에만 적용된다.

요약

- 객체 리터럴을 변수에 할당하거나 함수에 매개변수로 전달할 때 잉여 속성 체크가 수행된다.
- 잉여 속성 체크는 오류를 찾는 효과적인 방법이지만, 타입스크립트 타입 체커가 수행하는 일반적인 구조적 할당 가능성 체크와 역할이 다르다. 할당의 개념을 정확히 알아야 잉여속성체크와 일반적인 구조적 할당 가능성 체크를 구분할 수 있다.
- 잉여속성체크에는 한계가 있다. 임시 변수를 도입하면 잉여 속성 체크를 건너 뛸 수 있다.

## 아이템 12 함수 표현식에 타입 적용하기

자바스크립트에서는 함수 문장(함수선언문)과 함수표현식을 다르게 인식한다.

```jsx
function rollDice1(sides: number): number { /* COMPRESS */ return 0; /* END */ }  // Statement
const rollDice2 = function(sides: number): number { /* COMPRESS */ return 0; /* END */ };  // Expression
const rollDice3 = (sides: number): number => { /* COMPRESS */ return 0; /* END */ };  // Also expression
```

타입스크립트에서는 함수 표현식을 사용하는 것이 좋다.

함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용할 수 있다는 장점이 있기 때문이다.

```jsx
type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => { /* COMPRESS */ return 0; /* END */ };
```

함수 타입 선언의 장점

- 불필요한 코드의 반복을 줄인다.
    
    ```jsx
    function add(a: number, b: number) { return a + b; }
    function sub(a: number, b: number) { return a - b; }
    function mul(a: number, b: number) { return a * b; }
    function div(a: number, b: number) { return a / b; }
    ```
    
    ```jsx
    type BinaryFn = (a: number, b: number) => number;
    const add: BinaryFn = (a, b) => a + b;
    const sub: BinaryFn = (a, b) => a - b;
    const mul: BinaryFn = (a, b) => a * b;
    const div: BinaryFn = (a, b) => a / b;
    ```
    
    이 예제는 함수 타입 선언을 이용했던 예제보다 타입 구문이 적다.
    
    함수 구현부도 분리 되어 있어 로직이 분명해진다.
    
    모든 함수 표현식의 반환타입까지  number로 선언한 셈이다.
    
    라이브러리에서는 공통함수시그니처를 타입으로 제공하기도한다
    
- 코드가 간결하고 안전하다.
    
    

요약

- 매개변수나 반환 값에 타입을 명시하기보다는 함수 표현식 전체에 타입구문을 적용하는 것이 좋다
- 만약 같은 타입 시그니처를 반복적으로 작성한 코드가 있다면 함수 타입을 분리해 내거나 이미 존재하는 타입을 찾아보도록 한다. 라이브러리를 직접 만든다면 공통 콜백에 타입을 제공해야한다.
- 다른 함수 시그니처를 참조하려면  typeof fn을 사용하면 된다.

## 아이템 13 타입과 인터페이스의 차이점 알기

타입스크립트에서 명명된 타입을 정의하는 방법은 두가지가 있다.

```jsx
// 1. 타입을 사용하는 방법
type TState = {
  name: string;
  capital: string;
}

// 2. 인터페이스를 사용하는 방법
interface IState {
  name: string;
  capital: string;
}
```

클래스를 사용할 수도 있지만, 클래스는 값으로도 쓰일 수 있는 자바스크립트 런타임의 개념이다.

대부분의 경우에는 둘다 사용해도 되지만 두개의 차이를 분명하게 알고 같은 상황에서는 동일한 방법으로 명명된 타입을 정의해 일관성을 유지해야한다.

비슷한 점

- 추가속성과 함께 할당한다면 동일한 오류가 발생한다
- 인덱스 시그니처를 사용할 수 있다.
- 함수 타입을 정의할 수 있다.
- 제너릭이 가능하다.
- 인터페이스는 타입을 확장할 수 있으며 타입은 인터페이스를 확장할 수 있다.
    
    → 인터페이스는 유니온 타입같은 복잡한 타입을 확장하지 못한다
    
    → 복잡한 타입을 확장하고 싶다면 타입과 &를 사용해야 한다.
    
- 클래스를 구현(implements)할때 사용가능하다.

다른점

- 유니온 타입은 있지만 유니온 인터페이스라는 개념은 없다
- 인터페이스는 타입을 확장할 수 있지만 유니온은 할 수 없다.
- 일반적으로 인터페이스보다 type 키워드가 쓰임새가 많다
    - type 키워드는 유니온이 될 수도 있고, 매핑된 타입 또는 조건부 타입같은 고급기능 활용
    - 튜플과 배열타입에도 type키워드를 이용해 더 간결하게 표현할 수 있다.
- 인터페이스로도 튜플과 비슷하게 구현가능하지만 concat같은 메서드는 사용할 수 없다.
    - 튜플은 type키워드로 구현하는 것이 낫다
- 인터페이스에는 보강(augment)이 가능하다
    
    ```jsx
    interface IState {
      name: string;
      capital: string;
    }
    interface IState {
      population: number;
    }
    const wyoming: IState = {
      name: 'Wyoming',
      capital: 'Cheyenne',
      population: 500_000
    };  // OK
    ```
    
    이 예제처럼 속성을 확장하는 것을 `선언병합(declaration merging)` 이라고한다.
    
    선언 병합을 지원하기 위해서는 반드시 인터페이스를 사용해야하며 표준을 따라야한다.
    

복잡한 타입 → 타입 별칭 사용

타입, 인터페이스 두가지 방법으로 모두 표현가능 →  일관성과 보강의 관점에서 고려

요약

- 타입과 인터페이스의 차이점과 비슷한 점을 이해해야한다.
- 한타입을 타입과 인터페이스 두가지 문법을 사용해서 작성하는 방법을 터득해야한다
- 프로젝트에서 어떤 문법을 사용할지 결정할때 한가지 일관된 스타일을 확립하고 보강기법이 필요한지 고려해야한다.

## 아이템 14 타입연산과 제너릭 사용으로 반복 줄이기

- 상수를 사용해 반복을 줄이는 기법
    
    ```jsx
    function distance(a: {x: number, y: number}, b: {x: number, y: number}) {
      return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
    }
    ```
    
    ```jsx
    interface Point2D {
      x: number;
      y: number;
    }
    function distance(a: Point2D, b: Point2D) { /* ... */ }
    ```
    
- 함수가 같은 타입 시그니처를 공유하고 있는 경우 명명된 타입으로 분리
    
    ```jsx
    // HIDE
    interface Options {}
    // END
    function get(url: string, opts: Options): Promise<Response> { /* COMPRESS */ return Promise.resolve(new Response()); /* END */ }
    function post(url: string, opts: Options): Promise<Response> { /* COMPRESS */ return Promise.resolve(new Response()); /* END */ }
    ```
    
    ```jsx
    // HIDE
    interface Options {}
    // END
    type HTTPFunction = (url: string, options: Options) => Promise<Response>;
    const get: HTTPFunction = (url, options) => { /* COMPRESS */ return Promise.resolve(new Response()); /* END */ };
    const post: HTTPFunction = (url, options) => { /* COMPRESS */ return Promise.resolve(new Response()); /* END */ };
    ```
    
- 다른 인터페이스를 확장하게 해서 반복 제거
    
    ```jsx
    interface Person {
      firstName: string;
      lastName: string;
    }
    
    interface PersonWithBirthDate extends Person {
      birth: Date;
    }
    ```
    
- 일반적이지는 않지만 인터섹션 연산자를 사용하는 방법
    - 유니온타입(확장할 수 없는)에 속성을 추가하려고 할 때 유용하다.
- 매핑된 타입 사용
    
    ```jsx
    type TopNavState = {
      [k in 'userId' | 'pageTitle' | 'recentFiles']: State[k]
    };
    ```
    
- 유니온을 인덱싱하기
    
    ```jsx
    interface SaveAction {
      type: 'save';
      // ...
    }
    interface LoadAction {
      type: 'load';
      // ...
    }
    type Action = SaveAction | LoadAction;
    type ActionType = Action['type'];  // Type is "save" | "load"
    ```
    
    Action 유니온에 타입을 더 추가하면 ActionType은 자동적으로 그 타입을 포함한다.
    
    Pick을 사용하여 얻게되는 type 속성을 가지는 인터페이스와는 다르다.
    

```jsx
interface Name {
  first: string;
  last: string;
}
type DancingDuo<T extends Name> = [T, T];

const couple1: DancingDuo<Name> = [
  {first: 'Fred', last: 'Astaire'},
  {first: 'Ginger', last: 'Rogers'}
];  // OK
const couple2: DancingDuo<{first: string}> = [
                       // ~~~~~~~~~~~~~~~
                       // Property 'last' is missing in type
                       // '{ first: string; }' but required in type 'Name'
  {first: 'Sonny'},
  {first: 'Cher'}
];
```

제너릭 타입에서 매개변수를 제한할 수 있는 방법은  extends를 사용하는 것

{first: string}은 Name을 확장하지 않기 때문에 오류가 발생한다.

요약

- DRY(don’t repeat yourself) 원칙을 타입에도 최대한 적용해야 한다
- 타입에 이름을 붙여서 반복을 피해야한다. extends를 사용해서 인터페이스 필드의 반복을 피해야한다.
- 타입들 간의 매핑을 위해 타입스크립트가 제공한 도구들을 공부하면 좋다. 여기에는 keyof, typeof, 인덱싱, 매핑된 타입들이 포함된다
- 제너릭타입은 타입을 위한 함수와 같다. 타입을 반복하는 대신 제너릭 타입을 사용하여 타입들 간에 매핑을 하는 것이 좋다. 제너릭 타입을 제한하려면 extends를 사용하면 된다
- 표준 라이브러리에 정의된 **Pick, Partial, ReturnType** 같은 제너릭 타입에 익숙해져야한다.

## 아이템 15 동적 데이터에 인덱스 시그니처 사용하기

타입스크립트에서는 타입에 `인덱스 시그니처` 를 명시하여 유연하게 매핑을 표현할수 있다

```jsx
type Rocket = {[property: string]: string};
const rocket: Rocket = {
  name: 'Falcon 9',
  variant: 'v1.0',
  thrust: '4,940 kN',
};  // OK
```

`[property: string]` 이 인덱스 시그니처 이며, 다음 세가지 의미를 담고 있다.

- 키의 이름: 키의 위치만 표시. 타입체커에서는 사용하지 않음
- 키의 타입: string이나 number 또는 symbol 의 조합이어야 하지만, 보통은 string을 사용한다
- 값의 타입: 어떤 것이든 될 수 있다.

이렇게 타입 체크가 수행되면 네가지 단점이 드러난다

- 잘못된 키를 포함해 모든 키를 허용한다
- 특정 키가 필요하지 않다
- 키마다 다른 타입을 가질 수 없다
- 키는 무엇이든 가능하기 때문에 자동완성 기능이 동작하지 않는다

string 타입이 너무 광범위해서 인덱스 시그니처를 사용하는 데 문제가 있다면 두가지 대안이 있다

1. Record를 사용하는 방법
    
    Record는 키 타입에 유연성을 제공하는 제너릭 타입이다.
    
    string의 부분집합을 사용할 수 있다.
    
    ```jsx
    type Vec3D = Record<'x' | 'y' | 'z', number>;
    // Type Vec3D = {
    //   x: number;
    //   y: number;
    //   z: number;
    // }
    ```
    
2. 매핑된 타입을 사용하는 방법
    
    매핑된 타입은 키마다 별도의 타입을 사용하게 해준다.
    
    ```jsx
    type ABC = {[k in 'a' | 'b' | 'c']: k extends 'b' ? string : number};
    // Type ABC = {
    //   a: number;
    //   b: string;
    //   c: number;
    // }
    ```
    

요약

- 런타임 때까지 객체의 속성을 알 수 없을 경우에만 (예를 들어  CSV 파일에서 로드하는 경우) 인덱스 시그니처를 사용하도록 한다.
- 안전한 접근을 위해 인덱스 시그니처의 값 타입에 undefined를 추가하는 것을 고려해야한다.
- 가능하다면 인터페이스, Record, 매핑된 타입 같은 인덱스 시그니처보다 정확한 타입을 사용하는 것이 좋다.

## 아이템 16 number 인덱스 시그니처 보다는 Array, 튜플, ArrayLike를 사용하기

자바스크립트에서 속성이름을 숫자로 사용하려고 하면 자바스크립트 런타임은 문자열로 변환한다.

Object.keys를 이용해 배열의 키를 나열해 보면, 키가 문자열로 출력된다.

타입스크립트는 이러한 혼란을 바로잡기 위해 숫자 키를 허용하고, 문자열 키와 다른 것으로 인식한다.

요약

- 배열은 객체이므로 키는 숫자가 아니라 문자열이다. 인덱스 시그니처로 사용된 number 타입은 버그를 잡기 위한 순수 타입스크립트 코드이다.
- 인덱스 시그니처에 number를 사용하기 보다 Array 나 튜플, 또는 ArrayLike 타입을 사용하는 것이 좋다.

## 아이템 17 변경 관련된 오류 방지를 위해 readonly

readyonly number[] 는 ‘타입’이고, number[] 와 구분되는 몇 가지 특징이 있다.

- 배열의 요소를 읽을 수 있지만, 쓸 수는 없다.
- length를 읽을 수 있지만, 바꿀 수는 없습니다.
- 배열을 변경하는 메서드를 호출할 수없다.

number[]는 readonly number[]의 서브타입이 된다.

따라서 변경가능한배열을 readonly 배열에 할당할 수 있다. 하지만 그 반대는 불가능하다.

```jsx
const a: number[] = [1, 2, 3];
const b: readonly number[] = a;
const c: number[] = b;
   // ~ Type 'readonly number[]' is 'readonly' and cannot be
   //   assigned to the mutable type 'number[]'
```

매개변수를 readonly로 선언하면 다음과 같은 일이 생깁니다.

- 타입스크립트는 매개변수가 함수 내에서 변경이 일어나는지 체크합니다.
- 호출하는 쪽에서는 함수가 매개변수를 변경하지 않는다는 보장을 받게된다
- 호출하는 쪽에서 함수에 readonly 배열을 매개변수로 넣을 수도 있다.

readonly는 얕게 동작한다 → 제너릭을 만들면 깊은 readonly타입을 사용할 수 있다

라이브러리를 사용하는것이 낫다. ts-essentials에 있는 DeepReadonly 제너릭을 사용하면된다.

 요약

- 만약 함수가 매개변수를 수정하지 않는다면 readonly로 선언하는게 좋다
- readonly를 사용하면 변경하면서 발생하는 오류를 방지할 수 있다.
- const와 readonly 의 차이를 이해해야 한다
    - `const`
    변수 참조를 위한 것
    변수에 다른 값을 할당/대입할 수 없음.
    - `readonly`
    속성을 위한 것
    속성을 앨리어싱을 통해 변경될 수 있음
    
    [읽기 전용(readonly)](https://radlohead.gitbook.io/typescript-deep-dive/type-system/readonly)
    
- readonly는 얕게 동작한다는 것을 명심해야한다.

## 아이템 18 매핑된 타입을 사용하여 값을 동기화하기

```jsx
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k]) {
      if (k !== 'onClick') return true;
    }
  }
  return false;
}
```

이 접근법을 이용하면 차트가 정확하지만 너무 자주 그려질 가능성이 있다

→ onClick 처럼 렌더링에 필요없는 속성이 추가된 경우를 말하는거 같음

```jsx
function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  return (
    oldProps.xs !== newProps.xs ||
    oldProps.ys !== newProps.ys ||
    oldProps.xRange !== newProps.xRange ||
    oldProps.yRange !== newProps.yRange ||
    oldProps.color !== newProps.color
    // (no check for onClick)
  );
}
```

이 코드는 차트를 불필요하게 다시 그리는 단점을 해결했습니다

→ 렌더링에 필요한 속성만 체크하므로 해결함 하지만 누락 가능성 있음

```jsx
interface ScatterProps {
  // The data
  xs: number[];
  ys: number[];

  // Display
  xRange: [number, number];
  yRange: [number, number];
  color: string;

  // Events
  onClick: (x: number, y: number, index: number) => void;
}
const REQUIRES_UPDATE: {[k in keyof ScatterProps]: boolean} = {
  xs: true,
  ys: true,
  xRange: true,
  yRange: true,
  color: true,
  onClick: false,
};

function shouldUpdate(
  oldProps: ScatterProps,
  newProps: ScatterProps
) {
  let k: keyof ScatterProps;
  for (k in oldProps) {
    if (oldProps[k] !== newProps[k] && REQUIRES_UPDATE[k]) {
      return true;
    }
  }
  return false;
}
```

타입체커가 대신할 수 있게 하는 코드

→ `[k in keyof ScatterProps]` 은 타입체커에게 REQUIRES_UPDATE가 ScatterProps과 동일한 속성을 가져야한다는 정보를 제공한다

요약

- 매핑된 타입을 사용해서 관련된 값과 타입을 동기화도록 합니다
- 인터페이스에 새로운 속성을 추가할 때, 선택을 강제하도록 매핑된 타입을 고려해야합니다.

## 아이템 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

코드의 모든 변수에 타입을 선언하는 것은 비생산적이며 형편없는 스타일로 여겨진다.

타입추론이 된다면 명시적 타입 구문은 필요하지 않다. 오히려 방해된다

```jsx
interface Product {
  id: string;
  name: string;
  price: number;
}

function logProduct(product: Product) {
  const id: number = product.id;
     // ~~ Type 'string' is not assignable to type 'number'
  const name: string = product.name;
  const price: number = product.price;
  console.log(id, name, price);
}
```

logProduct 는 비구조화 할당문을 사용해서 구현하는게 더 낫다

```jsx
function logProduct(product: Product) {
  const {id, name, price} = product;
  console.log(id, name, price);
}
```

비구조화 할당문은 모든 지역변수의 타입이 추론되도록한다.

타입스크립트에서 변수의 타입은 일반적으로 처음 등장할때 결정됩니다.

이상적인 타입스크립트 코드는 함수/메서드 시그니처에 타입 구문을 포함하지만, 함수 내에서 생성된 지역 변수에는 타입 구문을 넣지 않습니다.

함수 매개변수에 타입 구문을 생략하는 경우도 있다 → 기본값이 있는 경우

객체리터럴을 정의할때 타입을 명시하면 잉여속성체크(아이템11)가 동작한다.

반환타입을 명시해야하는 이유

1. 함수에 대해 더욱 명확하게 알 수 있다
2. 명명된 타입을 사용하기 위해서

요약

- 타입스크립트가 타입을 추론할 수 있다면 타입 구문을 작성하지 않는게 좋다
- 이상적인 경우 함수/메서드의 시그니처에는 타입 구문이 있지만, 함수 내의 지역 변수에는 타입 구문이 없다
- 추론될 수 있는 경우라도 객체 리터럴과 함수 반환에는 타입 명시를 고려해야한다. 이는 내부 구현의 오류가 사용자 코드 위치에 나타나는 것을 방지해준다.

## 아이템 20 다른 타입에는 다른 변수 사용하기

다른타입에 별도의 변수를 사용하는게 바람직한 이유는 다음과 같다

- 서로 관련이 없는 두 개의 값을 분리한다
- 변수명을 더 구체적으로 지을 수 있다
- 타입 추론을 향상시키며, 타입 구문이 불필요해진다
- 타입이 더 간결해진다
- let 대신 const로 변수를 선언하게 된다

요약

- 변수 의 값은 바뀔 수 있지만 타입은 일반적으로 바뀌지 않는다
- 혼란을 막기 위해 타입이 다른 값을 다룰 때에는 변수를 재사용하지 않도록 한다.