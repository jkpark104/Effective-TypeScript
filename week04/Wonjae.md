
## 31 | 타입 주변에 `null`값 배치하기

- 한 값의 `null`여부가 다른 값의 `null`여부에 암시적으로 관련되도록 설계하면 안된다.
- API 작성 시 반환 타입을 하나의 큰 객체로 만들고 반환 타입 전체가 `null`이거나 `null`이 아니도록 설계해야 한다.
- 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하도록 하여 `null`이 존재하지 않도록 해야 한다.
- `strictNullChecks`를 설정하면 코드에 많은 오류가 표시되겠지만, `null`값과 관련된 문제점을 찾아낼 수 있기 때문에 반드시 설정해야 한다.

<br>

---

## 32 | 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

- 만약 인터페이스 내에서 유니온을 사용하고 있다면 인터페이스끼리 유니온으로 묶는 것이 더 용이하지 않을지 고려해야 한다.
- 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 TS가 이해하기도 좋다.
- TS가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 한다.

```ts
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}

// 아래 형태로 작성하면 잘못된 조합으로 섞이는 경우를 방지할 수 있다.
// 또한 태그드 유니온도 사용하기 용이해진다.
interface FillLayer {
  type: 'fill';
  layout: FillLayout;
  paint: FillPaint;
}

interface LineLayer {
  type: 'line';
  layout: LineLayout;
  paint: LinePaint;
}

interface PointLayer {
  type: 'paint';
  layout: PointLayout;
  paint: PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer;
```

<br>

---

## 33 | `string`타입보다 더 구체적인 타입 사용하기

- `string`을 남발하여 선언된 코드는 `any`와 다를바가 없다.
- 때문에 가능하다면 `string`보다 더 구체적인 타입을 사용하는 것이 좋다.
  - 문자열 리터럴 타입 유니온
  - 객체 속성이름을 받을 때는 `keyof T`

<br>

---

## 34 | 부정확한 타입보다는 미완성 타입을 사용하기

- 타입 안정성을 높이기 위해 완벽을 추구하다가 부정확한 타입을 사용하게 되는 경우가 발생하지 않도록 해야한다.
- 타입이 없는 것보다 잘못된 타입이 더 안 좋다.
- 타입을 정확하게 모델링할 수 없다면 차라리 범위를 넓게 지정하는 형식으로 모델링하는 것이 낫다.
- `any`와 `unknown`을 구별해서 사용해야 한다.

<br>

---

## 35 | 데이터가 아닌, API와 명세를 보고 타입 만들기

- 데이터에 드러나지 않는 예외적인 경우들이 문제가 될 수 있으므로 데이터보다는 명세를 기반으로 모델링하는 것이 낫다.
- 코드의 타입 안전성을 위해 `API`또는 데이터 형식에 대한 타입 생성을 고려해야 한다.