## 41 | `any`의 진화를 이해하기

- TS에서 일반적으로 변수의 타입은 변수를 선언할 때 결정된다.
- 일반적인 타입들은 `정제`되지만 `any`는 진화한다.
- `any`의 진화는 값을 할당하거나 배열에 요소를 넣은 후에만 일어난다.
- 때문에 `any`타입이 진화했더라도 변수가 할당된 줄에 타입은 여전히 `any`이다.
- `any`를 진화시키는 방식보다 명시적 타입 구문을 사용하는 것이 안전한 타입을 유지하는 방법이다.

<br>

---

## 42 | 모르는 타입의 값에는 `any`대신 `unknown`을 사용하기

```ts
interface Book {
  name: string;
  author: string;
}

function parseYAML(yaml: string): unknown {
  // ...
}

const book = parseYAML(`
    name: Wuthering Heights
    author: Emily Bronte
`);

// 에러: unknown은 사용할 수 없음
book('');
```

- `unknown`은 `any`대신 사용할 수 있는 안전한 타입이다.

  - `any`는 어떠한 타입도 할당 가능하기 때문에 타입 체커가 의미가 없다.
  - 하지만 `unknown`은 어떠한 타입이든 `unknown`에 할당할 수 있지만 `unknown`은 오직 `any`에만 할당가능하기 때문에 타입시스템에서 보다 명확히 오류를 잡아낼 수 있다.

- `{}`과 `object`는 `null`과 `undefined`를 제외한 모든 값을 포함한다.
  - 정말 `null`과 `undefined`가 불가능하다고 판단되는 경우만 `unknown`대신 `{}`를 사용하라.

<br>

---

## 43 | 몽키 패치보다는 안전한 타입을 사용하기

> 몽키패치(Monkey Patch): 런타임 중인 프로그램 메모리의 소스 내용을 직접 바꾸는 것
>
> 몽키패치의 어원: `게릴라 패치`(guerrilla patch) -> `고릴라 패치`(gorilla patch) -> `원숭이 패치`(monkey Patch)

- 내장 타입에 데이터를 저장(몽키 패치)해야 하는 경우 아래와 같은 해결책이 있다.

  1. `interface` 보강(`argmentation`)

  ```ts
  interface Document {
    monkey: string;
  }

  document.monkey = 'Ta';
  ```

  2. 더 구체적인 타입 단언문

  ```ts
  interface MonkeyDocument extends Document {
    monkey: string;
  }

  (document as MonkeyDocument).monkey = 'Macaque';
  ```

<br>

---

## 44 | 타입 커버리지를 추적하여 타입 안전성 유지하기

- `noImplicitAny`가 설정되어 있어도, 명시적 `any` 또는 서드파티 타입 선언(`@types`)를 통해 `any`타입은 코드 내에 존재할 수 있다.

- 아래 명령어를 통해 프로젝트의 n개 심벌 중 m개가 `any`가 아니거나 `any`의 별칭이 아닌 타입을 가지고 있음을 알 수 있다.
  - `npx type-coverage`
  - `npx type-coverage --detail`

<br>

---

## 45 | `devDependencies`에 `typescript`와 `@types`추가하기

- TS를 사용할 때 전역으로 설치하기보다는 `devDependencies`에 추가하여 팀원들이 동일한 TS버전을 사용할 수 있도록 해야한다.
- `@types`의 의존성은 `devDependencies`에 포함시켜야 한다.
