## 21 | 타입 넓히기

> TS가 작성된 코드를 체크하는 정적 분석 시점에, **변수**는 `가능한` 값들의 집합인 타입을 가진다.

> 만약 **상수**를 사용하여 변수를 초기화할 때 타입을 명시하지 않으면 타입 체커는 지정된 단일 값으로 할당 가능한 값들의 집합을 유추한다.

> 이렇게 할당 가능한 값들의 집합을 유추하는 과정을 타입 넓히기라고 한다.

> ```ts
> // Type 'x'
> const x = 'x';
>
> // Type string
> let x = 'x';
> ```

### 넓히기 과정 제어하기

1. `const`

`const` 키워드는 재할당을 허용하지 않기 때문에 더욱 좁은 타입으로 추론할 수 있다.

하지만 객체나 배열의 경우에는 프로퍼티가 재할당 가능하기 때문에 `const`만으로는 프로퍼티의 타입 넓히기를 제어할 수 없다.

2. 명시적 타입 구문 제공

```ts
const v: { x: 1 | 3 | 5 } = {
  x: 1,
};
```

3. 타입 체커에 추가적인 문맥 제공

아이템 26에서 상세하게 다룰 예정

4. `const` 단언문 사용

```ts
  const v1 = {
    x: 1,
    y: 2
  } // { x: number; y: number; }

  const v2 = {
    x: 1 as const,
    y: 2
  } // { x: 1; y: number; }

  const v3 = {
    x: 1,
    y: 2
  } as const; // { readonly x: 1; readonly y: 2; }

  const a1 = [1, 2, 3]; // number[]
  const a2 = [1, 2, 3]; as const // readonly [1, 2, 3]
```

<br>

---

## 22 타입 좁히기

1. `null` 체크

```ts
const el = document.getElementById('foo'); // 타입이 HTMLElement | null

if (el) {
  el; // HTMLElement
} else {
  // null
}
```

2. `instanceof`
3. 속성 체크
4. `Array.isArray`같은 내장 함수
5. `typeof`
6. 태그드 유니온(`tagged union`)
7. 커스텀 함수(`사용자 정의 타입 가드`)

```ts
function isInputElement(el: HTMLElement): el is HTMLInputElement {
  return 'value' in el;
}

function getElementContent(el: HTMLElement) {
  if (isInputElement(el)) {
    el; // HTMLInputElement
    return el.value;
  }
  el; // HTMLElement
  return el.textContent;
}
```

<br>

---

## 23 | 한꺼번에 객체 생성하기

- 속성을 제각각 추가하지 말고 선언할 때 한번에 객체로 만들어야 한다.
- 객체에 조건부로 속성을 추가할 때는 아래와 같이 사용하는 것이 좋다.

```ts
declare let hasDates: boolean;
const nameTitle = { name: 'Khufu', title: 'Pharaoh' };

// 아래의 방식은 유니온으로 추론되기 때문에 다루기 까다롭다.
// const pharaoh: {
//     start?: number | undefined;
//     end?: number | undefined;
//     name: string;
//     title: string;
// }
const pharaoh = {
  ...nameTitle,
  ...(hasDates ? { start: -2589, end: -2566 } : {}),
};

function addOptional<T extends object, U extends object>(
  a: T,
  b: U | null
): T & Partial<U> {
  return { ...a, ...b };
}

// 이렇게 사용하는 것이 훨씬 효율적이다.
// const pharaoh2: {
//     name: string;
//     title: string;
// } & Partial<{
//     start: number;
//     end: number;
// }>
const pharaoh2 = addOptional(
  nameTitle,
  hasDates ? { start: -2589, end: -2566 } : null
);
```

<br>

---

## 24 | 일관성 있는 별칭 사용하기

해당 장에서 말하는 부분은 쉽게 표현하면 아래와 같다.

```ts
// 이렇게 별칭을 사용하기 보다
const box = polygon.box;

// 이렇게 비구조화 문법을 사용하라
const { box } = polygon;
```

- 별칭은 TS가 타입을 좁히는 것을 방해하기 때문에 변수에 별칭을 사용할 때는 일관되게 사용해야 한다.
- 비구조화 문법을 사용하여 일관된 이름을 사용하는 것이 좋다.
- 타입을 정제할 때는 프로퍼티보다는 지역 변수로 분리하여 사용하면 신뢰성 있게 타입을 정제할 수 있다.

<br>

---

## 25 | 비동기 코드에는 콜백 대신 `async`함수 사용하기

> 콜백보다는 프로미스, 프로미스보다는 `async/await`을 사용하는 것이 `타입 추론`, `간결하고 직관적인 코드 작성`, `모든 종류의 오류 제거` 측면에서 유리하다.

### 콜백보다 프로미스나 `async/await`을 사용해야하는 이유

- 콜백보다 프로미스가 코드를 작성하기 쉽다.
- 콜백보다 프로미스가 타입을 추론하기 쉽다.

### 프로미스보다 `async/await`을 사용해야하는 이유

- 일반적으로 더 간결하고 직관적인 코드가 된다.
- `async`함수는 항상 프로미스를 반환하도록 강제된다.

```ts
async function fetchPages() {
  const [res1, res2, res3] = await Promise.all([fetch(url1), fetch(url2), fetch(url3)])
  ...
}
```

비교가 안될정도로 차이가 심하다!

```ts
function fetchPagesCB() {
  let numDone = 0;
  const reses: string[] = [];
  const done = () => {
    const [res1, res2, res3] = reses;
    ...
  }

  urls.forEach((url, i) => {
    fetchURL(url, r => {
      reses[i] = url
      numDone++
      if (numDone === urls.length) done();
    })
  }
}
```

<br>

---

## 26 | 타입 추론에 문맥이 어떻게 사용되는지 이해하기

- 타입 추론에서 문맥이 어떻게 쓰이는지 주의하며 코드를 작성해야 한다.
- 변수를 뽑아서 별도로 선언했을 때 오류가 발생한다면 타입 선언을 추가해주면 된다.
- 변수가 상수임이 확실하다면 상수 단언(`as const`)을 사용하면 된다.
  - 하지만 상수 단언을 사용하면 정의한 곳이 아닌 사용한 곳에서 오류가 발생하므로 디버깅에 주의해야 한다.

<br>

---

## 27 | 함수형 기법과 라이브러리로 타입 흐름 유지하기

- 타입 흐름을 개선하고, 가독성을 높이고, 명시적인 타입 구문의 필요성을 줄이는 효율적인 방법은 함수형 기법을 사용하는 것이 좋다.
- 유틸리티 함수를 사용할 때 타입 흐름 제어나 명시적인 타입 구문을 줄이기 위해 함수의 로직을 이해하고 있다는 전제 하에 유틸리티 라이브러리를 사용하는 것도 좋다.

<br>

---

## 28 | 유효한 상태만 표현하는 타입을 지향하기

> 유효한 상태만 표현한다는 것은 특정 상태를 관리할 때 사용되는 속성만 갖도록 표현하라는 것이다.

- 유효한 상태와 무효한 상태 모두 표현하는 타입은 예측가능성이 떨어지고 오류를 초래하기 쉽다.
- 유효한 상태만 표현하는 타입을 지향하면 코드가 길어지거나 표현이 어려울 수는 있지만 결론적으로 유지보수를 용이하게 한다.

만약 응답에 대한 로딩상태, 에러, 성공을 관리하는 상태가 있다고 가정한다면 아래와 같은 형태일 것이다.

```ts
interface state {
  isLoading: boolean;
  error: boolean;
  data: string;
}
```

위와 같은 형태도 어색하지는 않다 말 그대로 상태를 관리하는 것이니 로딩 성공 여부와 에러 발생 여부 성공 데이터가 담겨있는 형태이기 때문에 추상화는 잘못되었다고 표현할 수 없다.

하지만 이렇게 되면 케이스 분기 처리가 복잡해질 수 있다.

때문에 아래와 같이 작성한다면 상태를 정의하는 코드는 길어지겠지만 분기 처리가 용이하다.

```ts
interface RequestPending {
  state: 'pending';
}

interface RequestError {
  state: 'error';
  error: string;
}

interface RequestSuccess {
  state: 'ok';
  data: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;

interface State {
  currentPage: string;
  requests: { [page: string]: RequestState };
}

function renderPage(state: State) {
  const { currentPage } = state;
  const requestState = state.requests[currentPage];

  switch (requestState.state) {
    case 'pending':
      return `Loading ${currentPage}...`;
    case 'error':
      return `Error! Unable to load ${currentPage}: ${requestState.error}`;
    case 'ok':
      return `<h1>${currentPage}</h1>\n${requestState.data}`;
  }
}
```

<br>

---

## 29 | 사용할 때는 너그럽게, 생성할 때는 엄격하게

- 보통 매개변수 타입은 반환 타입에 비해 범위가 넓은 경향이 있다.
  - 선택적 속성과 유니온 타입은 반환 타입보다 매개변수 타입에 더 많이 사용된다.
- 매개변수와 반환 타입의 재사용을 위해서 기본 형태(`반환 타입`)와 느슨한 형태(`매개변수 타입`)를 도입하는 것이 좋다.

<br>

---

## 30 | 문서에 타입 정보를 쓰지 않기

- 주석과 변수명에 타입 정보를 적는 것을 지양하고, 되도록이면 코드로 표현 가능하도록 하라
  - 타입 선언이 중복 되는 것으로 끝나면 다행이지만 타입 정보에 모순이 발생할 수 있다.
  - 주석을 작성할 때 타입에 대한 정보를 적게 되면 변경 사항에 대한 동기화가 힘들다.
- 타입이 명확하지 않은 경우에는 변수명에 단위정보를 포함하는 것도 고려하는 것이 좋을 수 있다.
