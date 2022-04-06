## 아이템 53 타입스크립트 기능보다는 ECMAScript 기능을 사용하기

- 타입스크립트 초기버전에는 자바스크립트에 없었던 클래스, 열거형, 모듈시스템이 포함될 수 밖에 없었다
- 자바스크립트에도 신규로 추가되면서 타입스크립트는 초기버전과 호환성을 포기했다
- 타입스크립트의 역할을 명확하게 하려면, 열거형, 매개변수 송성, 트리플 슬래시 임포트, 데코레이터는 사용하지 않는 것이 좋다.

## 아이템 54 객체를 순회하는 노하우

```tsx
const obj = {
  one: 'uno',
  two: 'dos',
  three: 'tres',
};
for (const k in obj) {
  const v = obj[k];
         // ~~~~~~ Element implicitly has an 'any' type
         //        because type ... has no index signature
}
let k: keyof typeof obj;  // Type is "one" | "two" | "three"
for (k in obj) {
  const v = obj[k];  // OK
}
```

- k는 string으로 추론
- obj 객체의 키 타입과 서로 다르게 추론되어서 오류
- k의 타입을 더욱 구체적으로 명시하면 오류는 사라진다

```tsx
interface ABC {
  a: string;
  b: string;
  c: number;
}

function foo(abc: ABC) {
  for (const k in abc) {  // const k: string
    const v = abc[k];
           // ~~~~~~ Element implicitly has an 'any' type
           //        because type 'ABC' has no index signature
  }
}
const x = {a: 'a', b: 'b', c: 2, d: new Date()};
foo(x);  // OK
```

- k를 string으로 추론하는 이유
    - foo 함수의 경우 ABC타입에 할당 가능한 어떠한 값이든 매개변수로 허용하기 때문에
    - 타입스크립트는 ABC 타입에 할당 가능한 객체에는 a,b,c 외에 다른 속성이 존재할 수 있기때문에 키를 string 타입으로 선택해야한다

- 타입문제 없이 객체의 키와 값을 순회하고 싶다면 `Object.entries`를  사용하면된다.
