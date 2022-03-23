## 아이템 11 잉여 속성 체크의 한계 인지하기

### 잉여 속성 체크와 할당 가능 검사와의 차이점 구분하기

✅ 잉여 속성 체크

```jsx
interface Room {
  numbDoors: number;
  ceilingHeightFt: number;
}

const r: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // error
};
```

✅ 할당 가능 검사

구조적 타이핑 관점에서 obj는 Room의 부분 집합을 포함하여 타입 체커가 통과 됨

```jsx
interface Room {
  numbDoors: number;
  ceilingHeightFt: number;
}

const obj = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // error
};

const r: Room = obj; // 정상
```

- 이처럼 타스크립트는 구조적 타입핑에 의해서 타입의 범주가 굉장히 넓어질 수 있으며, 이는 우리가 의도한데로 동작하지 않도록 할 수 있는 위험이 있습니다.
- 그러나 잉여 속성 체크를 이용하면 기본적으로 타입 시스템의 구조적 본질을 해치지 않으면서 객체 리터럴에 알 수 없는 속성을 허용하지 않으므로써 위의 문제를 해결할 수 있습니다(엄격한 객체 리터럴 체크)

### 잉여 속성이 체크되는 시점과 안 되는 시점

✅ 체크되는 시점 - 객체 리터럴 형태일 경우

```jsx
const o: Room = {
  numDoors: 1,
  ceilingHeightFt: 10,
  elephant: 'present', // error
};

// 할당하는 값이 객체 리터럴
```

✅ 체크가 안 되는 시점 - 비 객체 리터럴, 타입 단언문

```jsx
interface Options{
	title : string;
	darkMode : boolean;
}

const o = { darkmode : true, title : 'ski free' } as Options
```

- 따라서 단언문보다 선언문을 사용을 지향해야 한다

### 잉여 속성을 원하지 않을 경우 - 인덱스 시그니처, 약한(weak) 타입

✅ 인덱스 시그니처

```jsx
interface Options {
  darkMode?: boolean;
  [otherOption: string]: unknown;
}

const o: Options = { darkmode: true }; // 정상
```

✅ 약한 타입 - 선택적 속성만 가지고 있는 타입

```jsx
interface LineCharOptions {
  logscale?: boolean;
  invertedYAxis?: boolean;
}

const opts = { logScale: true };
const o: LineCharOptions = opts; // LineCharOptions 유형과 공통적인 속성이 없습니다.
```

- 약한 타입의 경우 타입스크립트는 값 타입과 선언 타입에 공통 속성이 있는지 확인하는 별도의 체크를 수행
- 그러나 구조적으로 엄격하지 않고, 약한 타입과 관련된 할당문 마다 공통 속성 체크가 수행된다.

### 잉어 속성 체크의 장점

구조적 타이핑 시스템에서 속성 이름의 오타를 검수하는데 유용하다

## 아이템 12: 함수 표현식에 타입 적용하기

### 함수 선언문(statement)와 함수 표현식(expression)

타입스크립트에서는 함수 표현식을 사용하는 것이 좋다. 왜냐하면 함수 표현식은 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 `함수 표현식에 재사용`할 수 있는 장점이 있기 때문이다.

```jsx
type DiceRollFn = (slides : number) => number

const rollDice : DiceRollFn = slides => { ... }
```

- `rollDice` 함수의 매개변수에 마우스를 올리면, 이미 해당 매개 변수의 타입을 number로 인식한다
- 이는 불필요한 코드의 반복을 줄일 수 있습니다.

✅ fetch 함수 타입을 정의한 후 사용하기

```jsx
declare function fetch(
  input: RequestInfo,
  init?: RequestInit
): Promise<Response>;

const checkedFetch: typeof fetch = async (input, init) => {
  const response = await fetch(input, init);
  if (!response.ok) {
    // 만약 throw 대신 return을 사용할 경우 타입스크립트에서 해당 함수를 호출할 때 에러를 발생
    // Promise<Response | Error> 형식은 Promise<Response>에 할당될 수 없다
    throw new Error('error message');
  }

  return response;
};
```

## 아이템 13 : 타입과 인터페이스의 차이점 알기

### 타입 선언 방법 (`type` vs `interface`)

두 방법의 차이를 분명히 알고, 각 상황에서 동일한 방식의 타입 선언 방법을 사용하는 일관성을 유지해야 한다

**✅ 비슷한 점**

1. (잉여 속성 체크) 두 방식으로 선언된 타입의 경우, 추가 속성을 할당하는 방식을 사용하면 오류가 발생

```jsx
interface Room {
	numbDoors : number ,
	ceilingHeightFt : number
}

const r : Room = {
	numDoors : 1 ,
	ceilingHeightFt : 10,
	elephant : 'present' // error
}

----
type Options = {
	title : string
}

const o : Options = { title : 'lion king', categories : "movie" } // error
```

2. 두 타입 모두 인덱스 시그니처를 사용할 수 있음

3. 함수 타입도 모두 선언 가능하다 (아이템 12)

4. 타입 별칭과 인터페이스는 모두 제너릭이 가능함

5. 타입은 인터페이스를 확장할 수 있으며, 인터페이스는 타입을 확장할 수 있다(물론 주의사항 존재)

→ 클래스를 사용하는 경우, 클래스는 타입과 인터페이스 모두 확장 가능하다

```jsx
interface IStateWithPop extends Tstate {
  population: number;
}

type TstateWithPop = Istate & { population: number }; // intersection
```

- interface 로 구현된 Istate의 경우, 유니온 타입과 같은 복잡한 타입을 확장하지 못한다
- 따라서 복잡한 타입을 확장하고 싶다면 타입과 `&` 를 사용해야 한다

✅ 다른 점

1. 유니온 타입은 있지만, 유니온 인터페이스는 없다

- 인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없음
- 유니온 타입의 확장이 필요한 경우, 일반적으로 타입 별칭(type) 방법을 사용해야 합니다.
  ```jsx
  type Input = { ... };
  type Output = { ... };

  // 아래와 같이 유니온 타입의 확장이 필요한 경우 인터페이스를 사용할 수 없습니다.
  type NamedVariable = (Input | Output) & { name: string };
  ```

2. 튜플의 경우 타입 키워드를 이용해야 합니다.

- 인터페이스로 튜플을 구현할 수 있지만, 튜플에서 사용하는 concat과 같은 메서드를 사용할 수 없습니다.
  ```jsx
  // 인터페이스 방식보다 더 간결함
  type Pair = [number, number]

  ---

  interface Pair {
  	0 : number,
  	1: number ,
  	length : 2;
  }

  const t : Pair = [10, 20] // 정상
  ```

3. 인터페이스는 선언 병합 시 사용하는 보강(augment)이 가능합니다.

- 타입 선언 파일을 작성 할 때는 선언 병합을 지원하기 위해서 반드시 인터페이스를 사용해야 합니다.
  ```jsx
  interface Istate{
  	name : string;
  	capital : string
  }

  interface Istate{
  	population : number;
  }

  const korea: Istate{
  	name : 'Korea',
  	capital : 'Seoul',
  	population : 5000 // 정상
  }
  ```
- 자바스크립트는 여러 버전의 자바스크립트 표준 라이브러리에서 타입을 모아 병합한다.
- 따라서 `type` 은 기존 타입에서 추가적인 보강이 없는 경우에만 사용해야 한다.

### 결국 뭘 써야 하는가

- 해당 프로젝트의 일관된 방식을 따른 것이 우선이다
- 신규 프로젝트의 경우, 향후의 보강 가능성을 생각한다면 인터페이스를 사용하는 것이 좋다 (API 타입 설계 등)

## 아이템 14 : 타입 연산과 제너릭 사용으로 반복 줄이기

### 타입에 이름 붙히기

```jsx
function distance (a : { x:number, y:number}, b: {x:number, y:number}) { ... }
```

```jsx
interface Point2D {
	x : number;
	y : number;
}

function distance (a: Point2D, b: Point2D) { ... }
```

### 함수 시그니처 공유하기

```jsx
function get (url : string, opts: Options): Promise<Response> { ... }
function post (url : string, opts: Options): Promise<Response> { ... }
```

```jsx
type HTMLFunction = (url : string, opts: Options) => Promise<Response>

const get : HTMLFunction = (url, opts) => { ... }
const post : HTMLFunction = (url, opts) => { ... }
```

### 인터페이스 확장

```jsx
interface Person {
	first : string;
	last : string;
}

interface PersonWithBirth extends Person {
	birth : Date;
}

----
// 자주 사용하는 방식은 아님
type PesonWithBirth = Person & { birth : Date }
```

### 전체 속성 중 일부 속성을 가져와 타입 선언

```jsx
interface State {
	userId : string;
	pageTitle : string;
	recentFile : string[];
	pageContents : string;
}

interface TopNav {
	userId : State['userId];
	pageTitle : State['pageTitle];
}
```

- 이 방식을 사용하면 State의 pageTitle 속성이 변경되어도 TopNav에 반영됨
- 하지만 중복을 더 줄이기 위해서 `‘매핑된 타입'`을 사용하면 된다

### 매핑된 타입

```jsx
type TopNav = {
	[k in 'userId' | 'pageTitle'] : State[k]
}
```

```jsx
type Pick<T, K> = { [k in K] : T[k] }

type TopNav = Pick<State, 'userId' | 'pageTitle'>;
```

- 해당 방식은 매핑된 타입이 배열의 필드 루프를 도는 것과 동일한 방식으로 작동한다.
- 이 패턴은 표준 라이브러리에서 찾을 수 있으며, `Pick`이라고 한다
- `Pick`은 제너릭 타입으로, 함수를 호출하는 것과 마찬가지로 동작한다.

### 태그된 유니온

<aside>
🕹️ 태그된 유니온이란 ? 상황에 따라서 타입을 인터페이스로 나누고 이를 `type` 을 이용해 인터페이스를 유니온으로 정의한 타입을 의미한다

</aside>

```jsx
interface SaveAction {
	type : 'save';
	...
}

interface LoadAction {
	type : 'load';
	...
}

type Action = SaveAction | LoadAction; // 태그된 유니온
type ActionType = 'save' | 'load' // 중복
```

```jsx
type ActionType = Action['type']; // 타입은 'save' | 'load' (유니온을 반환)
```

- 다만 해당 해당 타입은 Pick을 사용하여 얻게 되는 인터페이스와 다르다 (객체를 반환)

```jsx
type ActionType = Pick<Action, 'type'>; // 타입은 { type : 'save' | 'load' }
```

### Partial

- 다음 인터페이스를 keyof를 사용하여 다름과 같이 압축할 수 있습니다.
  ```jsx
  interface Option {
  	width: number;
  	height: number;
  	color : string;
  	label : string;
  }

  interface OptionUpdate {
  	width?: number;
  	height?: number;
  	color?: string;
  	label?: string;
  }

  ---

  type OptionsUpdate = { [ k in Options]?: Options[k] }
  ```
  ```jsx
  type OptionsKey = keyof Options // keyof는 타입을 받아서 "속성" 타입의 유니온을 반환
  // 타입이 "width" | "height" | "color" | "label"
  ```
- 이런 패턴 역시 표준 라이브러리에서 Partial 이라는 이름으로 정의되어 있음
  ```jsx
  class UIwidget {
  	constructor(init: Options) {}
  	update(options : Partial<Options>} { }
  }
  ```

### 값 형태의 타입 선언

```jsx
cosnt INIT_OPTS = {
	width: 120;
	height: 230;
	color : "red";
	label : "VGA";
}

type Options = typeof INIT_OPTS
```

- typeof 연산자는 타입스크립트 단계에서 연산되어 타입을 반환한다.
- 다만 선언 순서는 타입을 먼저 정의하고 타입을 할당하는것이 좋다. 그래야 타입이 더 명확하고 예상하기 어려운 타입 변동을 방지할 수 있다.

### 제너릭 타입에서 매개변수 제한하기

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

- 원래 확장하지 않으면 couple2 예제도 타입 체크를 통과하게 된다
- 하지만, Name을 확장했기 때문에 last 속성이 없는 타입의 경우 문제가 된다.
- 즉, `extends`는 확장이 아니라 부분 집합의 개념이라고 이해할 필요가 있다
  ```jsx
  type Pick<T, K> = { [k in K] : T[k] } // error
  // ~K 타입은 'string | number | symbol' 타입에 할당할 수 없습니다

  ---

  type Pick<T, extends typeof K> = { ... } // 정상
  ```
  - 자바스크립트 객체의 속성 타입으로 string | number | symbol 만 올 수 있다
  - 따라서 속성 인덱스에 해당하는 K의 범위를 줄일 필요가 있다
  - 따라서 keyof T를 하면 T에 해당하는 key(인덱스)의 유니온 타입으로 K를 제한할 수 있다.

## 아이템 15 동적 데이터에 인덱스 시그니처 사용하기

인덱스 시그니처를 사용하여 객체의 타입을 유연하게 매핑할 수 있다

```jsx
typeof Rocket = { [property : string] : string }
const rocket = {
  name: 'Falcon 9',
  variant: 'Block 5',
  thrust: '7,607 kN',
};
```

- 인덱스 시그니처는 다음 세 가지 의미를 담고 있습니다
  - 키의 이름 : 키의 위치만 표시하는 용도. 타입 체커에서는 사용하지 않는다
  - 키의 타입 : string이나 number 또는 symbol의 조합이어야 하지만, 보통은 string만 사용한다
  - 값의 타입 : 어떤 것이든 될 수 있다
- 인덱스 시그니처의 4가지 단점
  - 잘못된 키를 포함한 모든 키를 허용한다
  - 특정 키가 필요하지 않는다 {} 도 Rocket 타입이 된다
  - 키마다 다른 타입을 가질 수 없다
  - 타입 스크립트 언어 서비스의 도움을 받지 못한다
- 따라서 인덱스 시그니처 보다 인터페이스를 사용하는 것이 더 정확한 방법이다
  하지만, 사전에 속성의 이름이나 타입을 알 수 없는 경우 인덱스 시그니처를 사용할 수 있다

### 타입에 가능한 필드가 제한된 경우

- 인덱스 시그니처를 사용할 경우, 타입의 필드가 너무 광범위한 경우 문제를 해결하기 위해 `Record`를 사용할 수 있습니다
  ```jsx
  interface Row1 {
    [column: string]: number;
  } // Too broad
  interface Row2 {
    a: number;
    b?: number;
    c?: number;
    d?: number;
  } // Better
  type Row3 =
    | { a: number }
    | { a: number, b: number }
    | { a: number, b: number, c: number }
    | { a: number, b: number, c: number, d: number }; // 번거로움
  ```
  ```jsx
  type Vec3D = Record<'x' | 'y' | 'z', number>;
  type Vec3D = { [k in 'x' | 'y' | 'z'] : number};
  type Vec3D = { [k in 'x' | 'y' | 'z']: k extends 'y' ? string : number};
  ```

## 아이템 16 number 인덱스 시그니처보다 Array, 튜플, ArrayLike 사용하기

- 자바스크립트에서 객체의 키값은 항상 문자열만 올 수 있다
- 하지만 배열은 분명히 객체이지만 숫자로 인덱싱 할 수 있다
  ```jsx
  const x = [1, 2, 3];

  console.log(x[0]); // 1
  ```
  ```jsx
  Object.keys(x); // ['0', '1', '2']
  ```
  - 하지만 실제로 배열의 키를 나열하면 키가 문자열로 출력된다.
- 타입스크립트는 위의 혼란을 바로잡기 위해, `숫자 키를 허용`하고 문자열 키와 다른 것으로 인식한다
  ```jsx
  interface Array<T> {
    [n: number]: T;
  }
  ```
- 그러나 일반적으로 key 자리에 string 대신 number를 타입으로 사용할 이유가 많지 않습니다.
  - 이는 옿려 숫자 속성이 어떤 특별한 의미를 가지고 있다는 오해를 불러 일으킬 수 있습니다.
  - 따라서 number를 인덱스 시그니처로 사용하기 보다, `Array`나 `튜플`, `ArrayLike` 타입을 사용하는 것이 좋습니다.
- 배열을 순회하는 방법에 있어 차이를 알아봅시다.
  ```jsx
  const xs = [1, 2, 3]
  console.log(xs['0']) // 타입 시스템에서 string이 number에 할당될 수 없음

  ---

  for(const key in xs) {
  	key // string
  	cosnt x = xs[key] // number
  }
  ```
  - 위의 상황은 실용적인 허용이라고 생각하는 것이 좋다
  - 그러나 인덱스가 필요하지 않는 경우 `for...of` 를 사용하는 것이 좋습니다.
  - 만약 인덱스가 필요한 경우, `forEach` 메서드를 사용하면 된다.
  - 순회 중간에 멈춰야 하는 경우 `for(;;)` 방법을 사용하는 것이 좋습니다.
    → 왜냐하면 `for...in` 의 경우 다른 배열 순회 방법보다 몇 배 느리기 때문이다.

## 아이템 17 변경 관련된 오류 방지를 위해 readonly 사용하기

- 매개변수로 객체를 전달할 경우, 해당 함수는 해당 객체의 값을 변경할 수 있다.
- 이런 경우를 방지하기 위해 readonly 접근 제어자를 사용하면 된다.
  ```jsx
  function arraySum(arr: readonly number[]) {
    let sum = 0, num;
    while ((num = arr.pop()) !== undefined) {
                   // ~~~ 'pop' does not exist on type 'readonly number[]'
      sum += num;
    }
    return sum;
  }
  ```
  - 배열 요소를 읽을 수 있지만, 쓸 수 없다
  - `length` 값을 읽을 수 있지만, 바꿀 수는 없다
  - 배열을 변경하는 pop 과 같은 메서드를 호출할 수 없다
- 일반 배열이 readonly 배열 보다 더 많은 기능이 있으므로, readonly 배열의 서브 타입이 된다.
  - 따라서 일반 배열은 readonly 배열에 할당할 수 있지만, 그 반대는 안 된다
  ```jsx
  const a: number[] = [1, 2, 3];
  const b: readonly number[] = a;
  const c: number[] = b;
     // ~ Type 'readonly number[]' is 'readonly' and cannot be
     //   assigned to the mutable type 'number[]'
  ```
- 이런 readonly를 사용하는 것은 명시적으로 컴파일러와 사람에게 객체의 불변성이 보장됨을 알려준다.
  - 함수의 매개변수를 readonly로 설정할 경우, 해당 함수를 호출하는 모든 함수도 readonly로 만들어야 한다
  - 그러나 외부 라이브러리는 이런 변경이 어려우므로 타입 단언문을 사용해야 한다
- readonly는 얕게 동작한다
  ```jsx
  const dates: readonly Date[] = [new Date()];
  dates.push(new Date());
     // ~~~~ Property 'push' does not exist on type 'readonly Date[]'
  dates[0].setFullYear(2037);  // OK
  ```
  - `Readonly` 제너릭을 사용할 수도 있습니다.
    ```jsx
    interface Outer {
      inner: {
        x: number,
      };
    }

    const o: Readonly<Outer> = { inner: { x: 0 } };

    o.inner = { x: 1 };
    // ~~~~ Cannot assign to 'inner' because it is a read-only property
    o.inner.x = 1; // OK

    type T = Readonly<Outer>;
    // Type T = {
    //   readonly inner: {
    //     x: number;
    //   };
    // }
    ```
  - 만약 깊은 동작을 원할 경우 ts-essentials에 있는 DeepReadonly 제너릭을 활용할 수 있다
- 인덱스 시그니처에 readonly를 사용할 경우 읽기만 허용하고 쓰기를 방지하여 객체 속성의 변경을 방지할 수 있습니다.
  ```jsx
  let obj: {readonly [k: string]: number} = {};
  // Or Readonly<{[k: string]: number}
  obj.hi = 45;
  //  ~~ Index signature in type ... only permits reading
  obj = {...obj, hi: 12};  // OK
  obj = {...obj, bye: 34};  // OK
  ```

## 아이템 18 : 매핑된 타입을 사용하여 값을 동기화하기

### 실패에 닫힌 접근법 vs 실패에 열린 접근법

- 다음과 같은 interface 속성 중, , events 데이터를 제외하고 data & display 속성이 변경되면 리랜더링하려고 할 경우 할 수 있는 두 가지 방법이 있다.
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
  ```
- 실패에 닫힌 접근법
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
  - 실패에 닫힌 접근법은 오류 발생 시 적극적으로 대처하는 방식으로 방어적, 보수적 접근법입니다
  - 따라서 보안이 중요한 곳의 경우 닫힌 방법을 사용해야 합니다
  - 하지만 보수적이기 때문에 너무 자주 그려질 가능성이 있습니다
- 실패에 열린 접근법
  ```jsx
  function shouldUpdate(oldProps: ScatterProps, newProps: ScatterProps) {
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
  - 실패에 열린 접근 법은 오류 발생 시 소극적으로 대처하는 방법입니다.
  - 따라서 사용성이 중요한 곳이라면 열린 방법이 적합합니다.
  - 그러나 누락될 위험이 있습니다.

### 매핑된 타입으로 문제 해결하기

```jsx
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

- `[k in keyof ScatterProps]` 은 `REQUIRES_UPDATE` 가 `ScatterProps` 와 매핑되도록 만든다
  (`ScatterProps` 와 동일한 속성을 갖게 된다)
- 이렇게 매핑을 할 경우, 어떤 속성에 따라 리랜더링 할지 여부를 미리 정의할 수 있다
- 또 `ScatterProps` 의 속성이 변경 될 경우, `REQUIRES_UPDATE` 에게 오류를 발생시켜 누락을 방지할 수 있다

## 아이템 19 추론 가능한 타입을 사용해 장황한 코드 방지하기

### 타입 추론이 불필요한 경우

- 일반적으로 변수를 선언할 때, 타입이 추론되므로 명시하는 것은 불필요하다
  ```jsx
  // 다음과 같이 선언해도 타입은 자동으로 추론된다
  let x = 12;

  const person = {
    name: 'Sojourner Truth',
    born: {
      where: 'Swartekill, NY',
      when: 'c.1797',
    },
    died: {
      where: 'Battle Creek, MI',
      when: 'Nov. 26, 1883',
    },
  };
  ```
- 타입스크립트는 우리가 생각하는 것보다 더 정교하게 타입을 추론합니다.
  ```jsx
  const axis1: string = 'a'; // 타입이 string
  const axis2 = 'y'; // 타입이 "y"
  ```
- 오히려 명시적으로 타입을 적는 것이 생산성 측면에서 비효율적입니다. 하지만 매개변수의 `product`와 같이 타입스크립트가 스스로 판단하기 어려운 상황에서는 명시적 타입 구문이 필요하다
  ```jsx
  interface Product {
    id: number; // 나중에 string으로 변경해야 하는 경우,
    name: string;
    price: number;
  }

  function logProduct(product: Product) {
    const id: number = product.id; // 이 코드에서 id의 타입을 명시적으로 작성하여 에러가 발생함
    const name: string = product.name;
    const price: number = product.price;
    console.log(id, name, price);
  }

  ---
  // 훨씬 더 간결함
  function logProduct(product: Product) {
    const {id, name, price} = product;
    console.log(id, name, price);
  }
  ```
- 일반적으로 타입스크립트 코드는 `함수/메서드의 시그니처`(매개변수 등)에 타입 구문을 포함하지만, 함수 내의 지역 변수에는 타입 구문을 넣지 않고 최소화하여 코드를 읽는 사람이 구현 로직에 집중할 수 있게 하는 것이 좋다
  - 다만 매개변수에 기본값이 있는 경우에는 타입을 생략해도 추론이 가능하다
  - express HTTP 라이브러리의 request와 response의 경우 타입 선언이 필요하지 않다

### 타입 추론이 필요한 경우

- `객체 리터럴`을 정의하는 경우에 잉여 속성 체크가 동작하여, **변수를 할당하는 시점**에 선택적 속성이 있는 타입의 오타 등을 표시하도록 도와준다.
  - 만약 타입을 체크하지 않으면, **변수를 사용하는 시점**에 오류 발생하기 때문에, 중요한 로직일 경우 중대한 에러가 발생할 수 있으므로 주의해야 함.
  ```jsx
  interface Product {
    id: string;
    name: string;
    price: number;
  }

  // 객체 리터럴로 정의
  const elmo: Product = {
    name: 'Tickle Me Elmo',
    id: '048188 627152',
    price: 28.99,
  };
  ```
- `함수의 반환`에 타입을 명시하여, 구현 상의 오류가 **함수를 호출한 곳까지 영향을 주지 않도록** 설계할 수 있다
  ```jsx
  const cache: { [ticker: string]: number } = {};
  function getQuote(ticker: string) {
    if (ticker in cache) {
      return cache[ticker];
    }
    return fetch(`https://quotes.example.com/?q=${ticker}`)
      .then((response) => response.json())
      .then((quote) => {
        cache[ticker] = quote;
        return quote;
      });
  }
  // getQuote는 항상 Promise를 반환하므로 이럴 경우 오류가 발생한다.
  function considerBuying(x: any) {}
  getQuote('MSFT').then(considerBuying);
  // ~~~~ Property 'then' does not exist on type
  //        'number | Promise<any>'
  //      Property 'then' does not exist on type 'number'
  ```
  ```jsx
  // 아이에 ReturnType을 명시하는 방법을 이용해, 구현 상의 로직이 호출 시점으로 이어지지 않도록 한다
  function getQuote(ticker: string): Promise<number> {
    if (ticker in cache) {
      return cache[ticker];
      // ~~~~~~~~~~~~~ Type 'number' is not assignable to 'Promise<number>'
    }
    // COMPRESS
    return Promise.resolve(0);
    // END
  }
  ```
  **[반환타입을 명시하면 좋은 장점 2가지]**
  1. 함수에 대해 더 명확하게 알 수 있다. 함수의 입력과 출력의 타입을 명시하는 것은 함수의 전체 로직을 일관성이 있게 설계할 수 있도록 만들어 주먹구구식으로 함수가 작성되는 것을 방지할 수 있다
  2. 반환된 타입이 복잡할 수록 **명명된(이름이 있는) 타입**을 제공한다면, 타입에 대한 주석을 작성할 수 있고 더욱 직관적인 표현이 가능합니다
    <aside>
    💡 `eslint` 규칙  `no-inferrable-types` 작성된 모든 타입 구문이 정말 필요한지 확인 가능
    
    </aside>


## 아이템 20 다른 타입에는 다른 변수 사용하기

- 변수의 값은 바뀔 수 있지만, 타입은 보통 바뀌지 않는다.
  ```jsx
  let id = '12-34-56';
  fetchProduct(id);

  id = 123456; // ~~ '123456' is not assignable to type 'string'.
  fetchProductBySerialNumber(id);
  ```
  - 따라서 다음과 같은 상황에서 `유니온` 타입을 이용해 타입을 확장해야 한다
    ```jsx
    let id: string | number = '12-34-56';

    id = 123456;
    ```
    → 그러나 유니온 타입은 일일히 변수를 사용할 때마다, 어떤 타입인지 확인해야 하는 단점이 있습니다
  - 따라서 이 경우에는 별도의 변수를 도입하는 것이 낫습니다.
    ```jsx
    const id = '12-34-56';

    const serial = 123456; // OK
    ```
    - 연관성 없는 두 값을 분리할 수 있습니다
    - 변수명을 더 구체적으로 지을 수 있습니다
    - 타입 추론을 향상시켜, 타입 구문이 불필요해지므로 코드가 더 간결해 집니다
    - let 대신 const로 선언하여, 타입 추론이 용이하고 의도하지 않은 값 변경을 방지할 수 있습니다
  - 물론 스코프가 다를 경우, 같은 이름의 변수를 사용해도 두 변수는 아무런 관계가 없습니다. 하지만 결국 코드를 읽는 사람 관점에서 혼란을 줄 수 있으므로 지양해야 합니다.
