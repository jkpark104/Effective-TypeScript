## 아이템 36 해당 분야의 용어로 타입 이름 짓기

- 이름 짓기는 타입설계에서 중요한 부분이다
- 엄선된 타입, 속성, 변수의 이름은 의도를 명확히하고 코드와 타입의 추상화 수준을 높여준다.

```tsx
interface Animal {
  name: string;
  endangered: boolean;
  habitat: string;
}

const leopard: Animal = {
  name: 'Snow Leopard',
  endangered: false,
  habitat: 'tundra',
};
```

- name은 매우 일반적인 용어 → 동물의 학명인지 일반적인 명칭인지 알 수 없다
- endangered → 이미 멸종된 동물을 true로 해야하는지 판단할 수 없음
- habitat → 범위가 넓은 string 타입임, 서식지라는 뜻도 불분명
- 객체의 변수명은 leopard이지만 , name 속성의 값은 Snow Leopard → 객체의 이름과 속성의 name이 다른 의도로 사용된 것인지 불분명하다

```tsx
interface Animal {
  commonName: string;
  genus: string;
  species: string;
  status: ConservationStatus;
  climates: KoppenClimate[];
}
type ConservationStatus = 'EX' | 'EW' | 'CR' | 'EN' | 'VU' | 'NT' | 'LC';
type KoppenClimate = |
  'Af' | 'Am' | 'As' | 'Aw' |
  'BSh' | 'BSk' | 'BWh' | 'BWk' |
  'Cfa' | 'Cfb' | 'Cfc' | 'Csa' | 'Csb' | 'Csc' | 'Cwa' | 'Cwb' | 'Cwc' |
  'Dfa' | 'Dfb' | 'Dfc' | 'Dfd' |
  'Dsa' | 'Dsb' | 'Dsc' | 'Dwa' | 'Dwb' | 'Dwc' | 'Dwd' |
  'EF' | 'ET';
const snowLeopard: Animal = {
  commonName: 'Snow Leopard',
  genus: 'Panthera',
  species: 'Uncia',
  status: 'VU',  // vulnerable
  climates: ['ET', 'EF', 'Dfd'],  // alpine or subalpine
};
```

- name을 더 구체적인 용어로 대체
- endangered는 표준 분류 체계인 ConservationStatus 로 변경
- habitat 는 기후를 뜻하는 climates로 변경

- 정보를 찾기위해서 사람에 의존할 필요 없이 정보를 찾을 수 있다.
- 코드를 표현하고자 하는 모든 분야에는 전문 용어들이 있으므로 자체적인 용어 대신 이미 존재하는 용어를 사용해야 한다 → 소통에 유리하고 타입의 명확성을 올릴 수 있다.

타입, 속성, 변수에 이름을 붙일 때 명심해야 할 세가지 규칙

- 동일한 의미를 나타낼 때에는 같은 용어 사용하기
- data, info, thing, item, object, entity 같은 모호하고 의미 없는 이름은 피하기
- 이름을 지을 때는 포함된 내용이나 계산 방식이 아니라 데이터 자체가 무엇인지 고려하기

## 아이템 37 공식 명칭에는 상표를 붙이기

- 구조적 타이핑의 특성 때문에 코드가 이상한 결과를 낼 수 있다
    
    ```tsx
    interface Vector2D {
      x: number;
      y: number;
    }
    function calculateNorm(p: Vector2D) {
      return Math.sqrt(p.x * p.x + p.y * p.y);
    }
    
    calculateNorm({x: 3, y: 4});  // OK, result is 5
    const vec3D = {x: 3, y: 4, z: 1};
    calculateNorm(vec3D);  // OK! result is also 5
    ```
    
- calculateNorm함수가 3차원 벡터를 허용하지 않게 하려면 공식명칭(nominal typing)을 사용하면 된다
- 공식 명칭 개념을 타입스크립트에서 흉내 내려면 brand를 붙이면 된다.
    
    ```tsx
    interface Vector2D {
      _brand: '2d';
      x: number;
      y: number;
    }
    function vec2D(x: number, y: number): Vector2D {
      return {x, y, _brand: '2d'};
    }
    function calculateNorm(p: Vector2D) {
      return Math.sqrt(p.x * p.x + p.y * p.y);  // Same as before
    }
    
    calculateNorm(vec2D(3, 4)); // OK, returns 5
    const vec3D = {x: 3, y: 4, z: 1};
    calculateNorm(vec3D);
               // ~~~~~ Property '_brand' is missing in type...
    ```
    
    brand를 사용해서 calculateNorm함수가 Vector2D 타입만을 받는 것을 보장한다
    
    vec3D에 `_brand: '2d'`를 추가하는 것 같은 악의적인 사용을 막을 수는 없지만 실수를 방지할 수 있다.
    
    상표기법은 런타임에서 상표를 검사하는 것과같은 효과를 얻을 수 있다.
    
    타입시스템이기 때문에 런타임 오버헤드를 없앨 수 있고
    
    추가 속성을 붙일 수 없는 string이나 number 같은 내장 타입도 상표화할 수 있다.
    
    타입 시스템에서 절대 경로 판단하기
    
    ```tsx
    type AbsolutePath = string & {_brand: 'abs'};
    function listAbsolutePath(path: AbsolutePath) {
      // ...
    }
    function isAbsolutePath(path: string): path is AbsolutePath {
      return path.startsWith('/');
    }
    function f(path: string) {
      if (isAbsolutePath(path)) {
        listAbsolutePath(path);
      }
      listAbsolutePath(path);
                    // ~~~~ Argument of type 'string' is not assignable
                    //      to parameter of type 'AbsolutePath'
    }
    ```
    
    타입 시스템에서 목록이 정렬되어 있다는 의도를 표현하기
    
    ```tsx
    type SortedList<T> = T[] & {_brand: 'sorted'};
    
    function isSorted<T>(xs: T[]): xs is SortedList<T> {
      for (let i = 1; i < xs.length; i++) {
        if (xs[i] > xs[i - 1]) {
          return false;
        }
      }
      return true;
    }
    
    function binarySearch<T>(xs: SortedList<T>, x: T): boolean {
      // COMPRESS
      return true;
      // END
    }
    ```
    
    number 타입에 상표 붙이기
    
    ```tsx
    type Meters = number & {_brand: 'meters'};
    type Seconds = number & {_brand: 'seconds'};
    
    const meters = (m: number) => m as Meters;
    const seconds = (s: number) => s as Seconds;
    
    const oneKm = meters(1000);  // Type is Meters
    const oneMin = seconds(60);  // Type is Seconds
    const tenKm = oneKm * 10;  // Type is number
    const v = oneKm / oneMin;  // Type is number
    ```
    
    산순 연산 후에는 상표가 없어 진다.
    
    하지만 숫자의 단위를 문서화하는 괜찮은 방법일 수 있다.
    

## 아이템 38 any 타입은 가능한 한 좁은 범위에서만 사용하기

```tsx
interface Foo { foo: string; }
interface Bar { bar: string; }
declare function expressionReturningFoo(): Foo;
function processBar(b: Bar) { /* ... */ }

function f() {
  const x = expressionReturningFoo();
  processBar(x);
  //         ~ Argument of type 'Foo' is not assignable to
  //           parameter of type 'Bar'
}
```

오류를 제거하는 방법 두가지

```tsx
function f1() {
  const x: any = expressionReturningFoo();  // Don't do this
  processBar(x);
}

function f2() {
  const x = expressionReturningFoo();
  processBar(x as any);  // Prefer this
}
```

- f1에서는 함수의 마지막 까지 x의 타입이 any
- f2에서는 processBar 호출 이후에 x가 그대로 Foo타입

```tsx
function f1() {
  const x: any = expressionReturningFoo();
  processBar(x);
  return x;
}

function g() {
  const foo = f1();  // Type is any
  foo.fooMethod();  // This call is unchecked!
}
```

- f1함수가 x를 반환한다면 g함수 내에서 f1의 반환타입인 any가 foo의 타입에 영향을 미친다
- 이렇게 함수에서 any를 반환하면 그 영향력은 프로젝트 전반에 전염병처럼 퍼진다

- @ts-ignore 를 사용한 다음 줄의 오류가 무시되지만 근본적인 원인을 해결한 것은 아님
- 타입 체커가 알려 주는 오류는 문제가 될 가능성이 높은 부분이므로 근본적인 원인을 찾아 해결하는 것이 좋다
- 어떤 큰 객체 안의 한개 속성이 타입 오류를 가지는 상황에서는 객체 전체를 any로 단언하지 말고 최소한의 범위에서만 any를 사용하는 것이 좋다.

## 아이템 39 any를 구체적으로 변형해서 사용하기

```tsx
function getLengthBad(array: any) {  // Don't do this!
  return array.length;
}

function getLength(array: any[]) {
  return array.length;
}
```

- 함수 내의 array.length 타입이 체크된다
- 함수의 반환 타입이 any대신 number로 추론된다
- 함수 호출될 때 매개변수가 배열인지 체크한다

매개변수를 구체화할 때

배열의 배열 형태 인 경우 → any[][]

객체이긴 하지만 값을 알 수 없는 경우 → {[key:string]:any} 또는 object

object 타입은 객체의 키를 열거할 수는 있지만 속성에 접근할 수 없다는 점에서  {[key:string]:any} 와 다르다.

```tsx
function hasTwelveLetterKey(o: object) {
  for (const key in o) {
    if (key.length === 12) {
      console.log(key, o[key]);
                   //  ~~~~~~ Element implicitly has an 'any' type
                   //         because type '{}' has no index signature
      return true;
    }
  }
```

객체지만 속성에 접근할 수 없어야한다면 unknown 타입이 필요

요약: any를 사용할 때는 정말로 모든 값이 허용되어야한 하는지 면밀히 검토하기

## 아이템 40 함수 안으로 타입 단언문 감추기

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  return function(...args: any[]) {
      // ~~~~~~~~~~~~~~~~~~~~~~~~~~
      //          Type '(...args: any[]) => any' is not assignable to type 'T'
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  };
}
```

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function cacheLast<T extends Function>(fn: T): T {
  let lastArgs: any[]|null = null;
  let lastResult: any;
  return function(...args: any[]) {
    if (!lastArgs || !shallowEqual(lastArgs, args)) {
      lastResult = fn(...args);
      lastArgs = args;
    }
    return lastResult;
  } as unknown as T;
}
```

반환문에 있는 함수와 원본함수가 관련있는지 알지 못해서 오류

결과적으로는 원본함수 T타입과 동일한 매개변수로 호출되고 반환값 역시 예상한 결과가 되기 때문에

타입 단언문을 추가해서 오류를 제거하는 것이 큰 문제가 되지 않는다.

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== b[k]) {
                           // ~~~~ Element implicitly has an 'any' type
                           //      because type '{}' has no index signature
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

```tsx
declare function shallowEqual(a: any, b: any): boolean;
function shallowObjectEqual<T extends object>(a: T, b: T): boolean {
  for (const [k, aVal] of Object.entries(a)) {
    if (!(k in b) || aVal !== (b as any)[k]) {
      return false;
    }
  }
  return Object.keys(a).length === Object.keys(b).length;
}
```

k in b로 b객체에 k속성이 있는것을 확인했기때문에

실제 오류는 아니므로 any로 단언하는 수밖에 없다
