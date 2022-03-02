# 1장 타입스크립트 알아보기

## 타입스크립트와 자바스크립트의 관계

- 타입스크립트는 고수준언어인 자바스크립트로 컴파일되며, 실행 역시 자바스크립트로 이루어진다.
  
    → 자바스크립트와 타입스크립트의 관계는 필연적이다
    
- 타입스크립트는 타입이 정의된 자바스크립트의 상위집합(superset)이다.
  
    → 자바스크립트는 타입스크립트의 부분집합이다
    
- 자바스크립트 프로그램에 문법 오류가 없다면, 유효한 타입스크립트 프로그램이라고 할 수 있다.
- 자바스크립트 파일은 `.js`(또는 `.jsx`) 확장자를 사용, 타입스크립트 파일은 `.ts`(또는 `.tsx`) 확장자를 사용
  
    → `main.js` 파일명을 `main.ts` 로 바꾼다해도 달라지는 것은 없다. 
    
    → 이러한 특성은 기존에 존재하는 자바스크립트 코드를 타입스크립트로 `마이그레이션(migration)`하는데 엄청난 이점이 된다. 일부분에만 타입스크립트 적용이 가능하기 때문
    
- 모든 자바스크립트 프로그램이 타입스크립트다 (O)
  
    → 반대는 성립하지않음
    
    → 타입스크립트 프로그램이지만 자바스크립트가 아닌 프로그램이 존재한다.
    
    → 타입스크립트가 타입을 명시하는 추가적인 문법을 가지기 때문이다.
    
    ```tsx
    // 이 코드를 자바스크립트로 실행하면 오류가 난다.
    function greet(who: string){
    	console.log('Hello',who);
    }
    ```
    
    → `:string` 은 타입스크립트에서 쓰이는 타입 구문이다.
    
    → 타입 구문을 사용하는 순간 자바스크립트는 타입스크립트의 영역으로 들어가게된다.
    
- 타입스크립트 컴파일러는 일반 자바스크립트 프로그램에도 유용하다
  
    ```tsx
    let city = 'new york city';
    console.log(city.toUppercase()); // TypeError: city.toUppercase is not a function
    ```
    
    타입구문이 없어도 타입스크립트의 타입 체커는 문제점을 찾아낸다.
    
- 타입시스템의 목표 중 하나는 런타임에 오류를 발생시킬 코드를 미리 찾아내는 것이다.
- 타입체커는 의도와 다르게 동작하는 코드도 찾아낸다.
  
    ```tsx
    const states = [
      { name: 'a', capital: '1' },
      { name: 'b', capital: '2' },
      { name: 'c', capital: '3' },
    ];
    
    for (const state of states) {
      console.log(state.capitol);
      //'capitol' 속성이 '{ name: string; capital: string; }' 형식에 없습니다.
      //'capital'을(를) 사용하시겠습니까?
    }
    ```
    
- 타입 구문이 있으면 코드의 의도가 무엇인지 타입스크립트에게 알려줄 수 있기때문에 훨씬 더 많은 오류를 찾아낼 수 있다.
  
    ```tsx
    interface State {
      name: string;
      capital: string;
    }
    const states: State[] = [
      { name: 'a', capital: '1' },
      { name: 'b', capital: '2' },
      { name: 'c', capitol: '3' },
    ];
    // 개체 리터럴은 알려진 속성만 지정할 수 있지만 
    // 'State' 형식에 'capitol'이(가) 없습니다.
    // 'capital'을(를) 쓰려고 했습니까?
    
    for (const state of states) {
      console.log(state.capital);
    }
    ```
    
- 타입스크립트 타입 시스템은 자바스크립트의 런타임 동작을 모델링합니다.
  
    ```tsx
    const x = 2 + '3'; // 정상, string 타입입니다.
    const y = '2' + 3; // 정상, string 타입입니다.
    ```
    
- 런타임에 오류가 발생하지 않는 코드인데 문제점을 표시하는 경우도 있다.
  
    ```tsx
    const a = null + 7;
    const b = [] + 12;
    alert('Hello', 'TypeScript');
    ```
    
- 타입스크립트의 도움을 받으면 오류가 적은 코드를 작성할 수 있다.

### 요약

- 타입스크립트는 자바스크립트의 상위집합이다
- 모든 자바스크립트 프로그램은 타입스크립트 프로그램이다
- 타입스크립트는 별도의 문법을 가지고 있기 때문에 일반적으로는 유효한 자바스크립트 프로그램이 아니다.
- 타입스크립트는 자바스크립트 런타임 동작을 모델링하는 타입 시스템을 가지고있기 때문에 런타임 오류를 발생시키는 코드를 찾아내려고한다. 그러나 타입 체커를 통과하면서도 런타임 오류를 발생시키는 코드는 충분히 존재할 수 있다.
- 자바스크립트에서는 허용되지만 타입스크립트에서는 문제가 되는 경우도 있다.

## 타입스크립트 설정 이해하기

타입스크립트는 매우 많은 설정을 가지고 있다.

커맨드 라인으로 설정하거나 `tsconfig.json` 설정파일을 통해서도 가능하다.

설정파일은 타입스크립트를 어떻게 사용할 계획인지 동료들이나 다른 도구들이 알 수 있다.

`tsc —init`만 실행하면 간단히 생성할 수 있다.

### noImplicitAny

- 변수들이 미리 정의된 타입을 가져야하는지 여부
- `noImplicitAny` 가 해제되어 있는 경우 변수가 `any` 타입으로 간주된다.
  
    → 이를 `암시적any` 라고 부른다 
    
- 타입스크립트는 타입 정보를 가질 때 가장 효과적이기 때문에 되도록이면 `noImplicitAny` 를 설정해야한다.
  
    → 문제를 발견하기 수월해지고, 코드의 가독성이 좋아지고, 개발자의 생산성이 향상된다.
    

### strictNullChecks

- `null` 과 `undefined` 가 모든 타입에서 허용되는지 확인하는 설정
  
    ```tsx
    const x: number = null; // 'null' 형식은 'number' 형식에 할당할 수 없습니다.
    const y: number | null = null;
    ```
    
    - (p12) 만약 null을 허용하지 않으려면, 이 값이 어디서부터 왔는지 찾아야하고, null 을 체크하는 단언문을 추가해야합니다??
- `null` 과 `undefined` 관련된 오류를 잡아 내는데 많은 도움이 되지만 코드 작성을 어렵게한다.
- `strictNullChecks` 를 설정하려면 `noImplicitAny` 를 먼저 설정해야한다.

타입스크립트에 strict 설정을 하면 대부분의 오류를 잡아낸다.

### 요약

- 타입스크립트 컴파일러는 언어의 핵심 요소에 영향을 미치는 몇 가지 설정을 포함하고 있다.
- 타입스크립트 설정은 커맨드라인을 이용하기보다는 `tsconfig.json` 을 사용하는 것이 좋다.
- 되도록이면 `noImplicitAny` 를 설정하는 것이 좋다
- “undefined는 객체가 아닙니다” 같은 런타임 오류를 방지하기 위해서는 `strictNullChecks` 를 설정하는 것이 좋다.
- 타입스크립트에서 엄격한 체크를 하고 싶다면 strict 설정을 고려하면 된다.

## 코드 생성과 타입이 관계없음을 이해하기

타입스크립트 컴파일러는 두가지 역할을 수행한다

- 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 `트랜스파일` 합니다.
- 코드의 타입 오류를 체크합니다.

트랜스파일

번역(translate)과 컴파일(complie)이 합쳐져 트랜스파일

소스코드를 동일한 동작을 하는 다른 형태의 소스코드(다른 버전, 다른언어)로 변환하는 행위를 의미

결과물이 여전히 컴파일되어야하는 소스코드이기 때문에 컴파일과는 구분해서 부른다.

이 두 가지가 서로 완벽히 독립적이다.

→ 타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 주지 않는다.

→ 자바스크립트의 실행 시점에도 타입은 영향을 미치지 않는다

### 타입 오류가 있는 코드도 컴파일이 가능하다.

- 타입스크립트의 오류는 경고와 비슷하다.
- 문제가 될 만한 부분을 알려 주지만, 그렇다고 빌드를 멈추지는 않는다.
- 코드에 오류가 있을때 “컴파일에 문제가 있다" 고 말하는것보다 “타입체크에 문제가 있다"고 말하는 것이 더 정확한 표현이다.
- 오류가 있더라도 컴파일된 산출물이 나오는 것은 문제가된 부분을 수정하지 않더라도 애플리케이션의 다른 부분을 테스트할 수 있다는 것이 도움이된다.
- 오류가 있을 때 컴파일하지 않으려면 `noEmitOnError` 를 설정하면된다.

### 런타임에는 타입 체크가 불가능하다.

```tsx
interface Square {
  width: Number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
                    //'Rectangle'은(는) 형식만 참조하지만, 여기서는 값으로 사용되고 있습니다.
    return shape.width * shape.height;
                        //'Shape' 형식에 'height' 속성이 없습니다.
  } else {
    return shape.width * shape.width;
  }
}
```

- instanceof 체크는 런타임에 일어나지만 Rectangle은 타입이기 때문에 런타임 시점에서는 아무런 역할을 할 수 없다.
- 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거된다.
- 코드에서 다루고 있는 shape 타입을 명확하게 하려면 런타임에 타입정보를 유지하는 방법이 필요하다.
    - 첫번째 방법
        - (p16)속성 체크는 런타임에 접근 가능한 값에만 관련되지만, 타입 체커 역시도 shape의 타입을 Rectangle로 보정해 주기 때문에 오류가 사라집니다.
    
    ```tsx
    function calculateArea(shape: Shape) {
      if ('height' in shape) {
        shape; // 타입이 Rectangle
        return shape.width * shape.height;
      } else {
        shape; // 타입이 Square
        return shape.width * shape.width;
      }
    }
    ```
    
    - 두번째 방법 (’태그' 기법)
        - 태그된 유니온(tagged union)의 예시
        - 이 기법은 런타임에 타입 정보를 손쉽게 유지할 수 있기때문에, 타입스크립트에서 흔하게 볼 수 있다.
    
    ```tsx
    interface Square {
      kind: 'square';
      width: number;
    }
    interface Rectangle {
      kind: 'rectangle';
      height: number;
      width: number;
    }
    type Shape = Square | Rectangle;
    
    function calculateArea(shape: Shape) {
      if (shape.kind === 'rectangle') {
        shape; // 타입이 Rectangle
        return shape.width * shape.height;
      } else {
        shape; // 타입이 Square
        return shape.width * shape.width;
      }
    }
    ```
    
    - 세번째 방법
        - 타입(런타임 접근 불가)과 값(런타임 접근가능)을 둘다 사용하는 기법
        - 타입을 클래스로 만드는 방법
        - 인터페이스는 타입으로만 사용가능하지만, 클래스로 선언하면 타입과 값으로 모두 사용할 수 있으므로 오류가 없다.
        - `type Shape = Square | Rectangle;` 부분에서는 타입으로 참조된다.
        - `shape instanceof Rectangle` 부분에서는 값으로 참조된다.

### 타입연산은 런타임에 영향을 주지 않는다.

```tsx
function asNumber(val:number | string):number{
	return val as number; //타입 연산이므로 런타임에는 아무런 영향이 없다
}
```

```tsx
function asNumber(val:number | string):number{
	return typeof(val) === 'string' ? Number(val): val;
	// 자바스크립트의 연산을 통해 변환을 수행해야한다.
}
```

### 런타임 타입은 선언된 타입과 다를 수 있다.

- `:boolean` 은 런타임에 제거된다
- 자바스크립트였다면 실수로 setLightSwitch를 “ON”으로 호출할 수도 있었을 것이다.
- 네트워크 호출로 부터 받아온 값으로 함수를 실행하는 경우에도 마지막 코드가 실행될 수 있다.

```tsx
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      on();
      break;
    case false:
      off();
      break;
    default:
      console.log('default');
  }
}
```

```tsx
class Square {
  constructor(public width: number) {}
}

class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}

type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    shape; // 타입이 Rectangle
    return shape.width * shape.height;
  } else {
    shape; // 타입이 Square
    return shape.width * shape.width;
  }
}
```

### 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

- C++ 같은 언어는 동일한 이름에 매개변수만 다른 여러 버전의 함수를 허용한다
  
    → 이를 ‘함수 오버로딩' 이라고 한다.
    
- 타입스크립트에서는 타입과 런타임의 동작이 무관하기때문에 함수 오버로딩이 불가능하다.
- 하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만 구현체는 오직 하나다.

### 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

- 런타임 오버헤드가 없는 대신 빌드타임 오버헤드가 있다
- 오래된 런타임 환경을 지원하기 위해 호환성을 높이고 성능 오버헤드를 감안할지, 호환성을 포기하고 성능 중심의 네이티브 구현제를 선택할지 문제에 맞닥뜨릴 수도 있다.

### 요약

- 코드 생성은 타입 시스템과 무관하다.
- 타입스크립트 타입은 런타임 동작이나 성능에 영향을 주지 않는다.
- 타입 오류가 존재하더라도 코드 생성(컴파일)은 가능하다.
- 타입스크립트 타입은 런타임에 사용할 수 없다. 런타임에 타입을 지정하려면 타입 정보 유지를 위한 별도의 방법이 필요하다. 일반적으로는 태그된 유니온과 속성 체크 방법을 사용한다. 또는 클래스 같이 타입스크립트 타입과 런타임 값 둘다 제공하는 방법이 있다.

## 구조적 타이핑에 익숙해지기

- 자바스크립트는 본질적으로 덕 타이핑(duck typing) 기반이다.
- 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용한다.
- 타입스크립트는 이런 동작을 그래도 모델링한다.
  
    → 매개변수 값이 요구사항을 만족한다면 타입이 무엇인지 신경 쓰지 않는 동작
    

덕타이핑이란

객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는 것으로 간주하는 방식

덕테스트에서 유래되었는데, 다음과 같은 명제로 정의됩니다. “만약 어떤 새가 오리처럼 걷고, 헤엄치고, 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다"

```jsx
interface Vector2D {
  x: number;
  y: number;
}
function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
interface NamedVector {
  name: string;
  x: number;
  y: number;
}
const v: NamedVector = { x: 3, y: 4, name: 'Zee' };
calculateLength(v);  // OK, result is 5
```

```jsx
function calculateLength(v) {
    return Math.sqrt(v.x * v.x + v.y * v.y);
}
var v = { x: 3, y: 4, name: 'Zee' };
calculateLength(v); // OK, result is 5
```

- NamedVector는 number 타입의 x와 y 속성이 있기 때문에 calculateLength 함수로 호출 가능하다.
- Vector2D 와 NamedVector 의 관계를 전혀 선언하지 않았지만 타입스크립트가 이해할 수 있다.
  
    → 이유는 자바스크립트의 런타임 동작을 모델링하기때문에 
    
    → NamedVector의 구조가  Vector2D와 호환된다.
    
    → 여기서 `구조적 타이핑` 이라는 용어가 사용된다.
    

### 구조적 타이핑때문에 문제가 생기는 경우

```jsx
interface Vector2D {
  x: number;
  y: number;
}
function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
interface NamedVector {
  name: string;
  x: number;
  y: number;
}
interface Vector3D {
  x: number;
  y: number;
  z: number;
}
function normalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

```jsx
function calculateLength(v) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}
function normalize(v) {
  var length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}

console.log(normalize({ x: 3, y: 4, z: 5 }));
// { x: 0.6, y: 0.8, z: 1 }
```

`calculateLength` 는 2D벡터를 기반으로 연산하는데, 버그로 인해 normalize가 3D벡터로 연산되었다.

→ z가 무시됨

함수를 작성할 때, 호출에 사용되는 매개변수의 속성들이 매개변수의 타입에 선언된 속성만을 가질거라 생각하기 쉽다. 

이러한 타입을 `봉인된(sealed)타입` 또는 `정확한(precise)타입` 이라고 하는데 타입스크립트 타입시스템에서는 표현할 수 없다. 

타입은 열려(open)있다. → 타입의 확장에 열려있다는 의미

### 타입이 열려있기때문에 생기는 당황스러운 경우

```jsx
function calculateLengthL1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis];
               // ~~~~~~~ Element implicitly has an 'any' type because ...
               //         'string' can't be used to index type 'Vector3D'
    length += Math.abs(coord);
  }
  return length;
}
```

```jsx
const vec3D = {x: 3, y: 4, z: 1, address: '123 Broadway'};
calculateLengthL1(vec3D);  // OK, returns NaN
```

v는 어떤 속성이든 가질 수 있기 때문에, axis의 타입은 string이 될 수도 있다.

v[axis]가 어떤 속성이 될지 알 수 없기 때문에 number라고 확정할 수 없다.

정확한 타입으로 객체를 순회하는 것은 까다로운 문제이다. → 아이템 54에서 다룰예정

이런경우에는 루프보다는 모든 속성을 각각 더하는 구현이 더 낫다.

```jsx
function calculateLengthL1(v: Vector3D) {
  return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}
```

