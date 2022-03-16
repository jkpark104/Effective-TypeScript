# [아이템30] 문서에 타입 정보를 쓰지 않기

> ✅ 요약
> 1. 주석과 변수명에 타입 정보를 적는 것은 피해야 한다. 타입 선언이 중복되는 것으로 끝나면 다행이지만 최악의 경우는 타입 정보에 모순이 발생하게 된다.
> 2. 타입이 명확하지 않은 경우는 변수명에 단위 정보를 포함하는 것을 고려하는 것이 좋다.
	> ex) timeMs, temperatureC 등

</br>

타입스크립트의 타입 구문 시스템은 간결, 구체적이며 쉽게 읽을 수 있도록 설계됨.
함수의 입력과 출력의 타입을 코드로 표현하는 것이 주석보다 더 나은 방법이라는 것은 분명.
강제하지 않는 이상 주석과 코드는 동기화 되지 않는다.
하지만 타입 구문은 타입스크립트 타입 체커가 타입 정보를 동기화하도록 강제한다.
때문에 주석 대신 타입 정보를 작성한다면 코드가 변경된다 하더라도 정보가 정확히 동기화 된다.

특정 매개변수를 설명하고 싶을 때는 JSDoc의 `@param` 구문을 사용하면 된다.

값이나 매개변수를 변경한다고 설명하는 주석 대신, `readonly`로 선언하여 타입스크립트가 규칙을 강제할 수 있게하는 것이 좋다.

변수명에 타입 정보를 넣지 않는 것이 좋다. 
_ex) ageNum 보다는 age의 타입이 number임을 명시하는 것이 좋다._

단위가 있는 숫자들은 예외로, 단위가 무엇인지 확실하지 않다면 변수명 또는 속성 이름에 단위를 포함할 수 있다.
_ ex) timeMs는 time보다 명확하다._

</br>

---

# [아이템32] 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

> ✅ 요약
> 1. 유니온 타입의 속성을 여러 개 가지는 인터페이스에서는 속성 간의 관계가 분명하지 않기 때문에 실수가 자주 발생하니 주의해야 함.
> 2. 유니온의 인터페이스보다 인터페이스의 유니온이 더 정확하고 타입스크립트가 이해하기 좋다.
> 3. 타입스크립트가 제어 흐름을 분석할 수 있도록 타입에 태그를 넣는 것을 고려해야 한다. tagged union은 타입스크립트와 매우 잘 맞기 때문에 자주 볼 수 있는 패턴이다.

</br>

유니온 타입의 속성을 가지는 인터페이스를 작성 중이라면, 혹시 인터페이스의 유니온 타입을 사용하는 게 더 알맞지 않을지 검토해 봐야 한다.

ex)
```typescript
///bad
interface Layer {
layout: FillLayout | LineLayout | PointLayout;
paing: FillPaint | LinePaint | PointPaint;
}
```

```typescript
//good
interface FillLayer {
layout: FillLayout;
paint:FillPaint;
}

interface LineLayer {
layout: LineLayout;
paint:LinePaint;
}

interface PointLayer {
layout: PointLayout;
paint:PointPaint;
}

type Layer = FillLayer | LineLayer | PointLayer;

```
-> 이런 형태로 Layer를 정의하면, layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있다.

위와 같은 패턴의 가장 일반적인 예시가 `tagged union`
Layer의 경우 속성 중의 하나는 문자열 리터럴 타입의 유니온이 된다.
```typescript
interface Layer {
type: 'fill' | 'line' | 'paint'
layout: FillLayout | LineLayout | PointLayout;
paing: FillPaint | LinePaint | PointPaint;
}
```
이 경우 fill 타입과 LineLayout, PointPaint 타입이 쓰이는 것은 말이 되지 않으므로,
Layer를 인터페이스의 유니온으로 변환하는 것이 좋다.

```typescript
//good
interface FillLayer {
type: 'fill';
layout: FillLayout;
paint:FillPaint;
}

interface LineLayer {
type: 'line';
layout: LineLayout;
paint:LinePaint;
}

interface PointLayer {
type: 'paint';
layout: PointLayout;
paint:PointPaint;
}
```
이제 type 속성이 `태그` 가 되며, 런타임에 어떤 타입의 Layer가 사용되는지 판단하는데 쓰일 수 있다.
또한 타입스크립트는 태그를 참고하여 Layer의 타입 범위를 좁힐 수도 있다.

이처럼 각 타입의 속성들 간의 관계를 제대로 모델링하면, 타입스크립트가 코드의 정확성을 체크하는 데 도움이 된다. 
`tagged union`은 타입스크립트 타입 체커와 잘 맞기 때문에 타입스크립트 코드 어디에서나 찾을 수 있다. 어떤 데이터 타입을 `tagged union`으로 표현할 수 있다면, 보통 그렇게 하는 것이 좋다.

또는 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 undefined인 경우도 `tagged union` 패턴이 잘 맞는다.
ex)
```typescript
interface Person {
name: string;
//다음 두 속성은 둘 다 동시에 있거나 동시에 없다.
placeOfBirth?: string;
dateOfBirth?: Date;
}
```

위 처럼 타입 정보를 담고 있는 주석은 문제가 될 확률이 높다.(아이템30 참고)
현재 placeOfBirth와 dateOfBirth는 실제로 관련되어 있지만, 타입 정보에는 어떤 관계도 표현되지 않았다.
이럴 경우에는 두 개의 속성을 하나의 객체로 모으는 것이 더 나은 설게이다.
```typescript
interface Person {
name: string;
birth?: {
	place: string;
	date: Date;
  }
}
```
이렇게 해주면 place, date 중 하나만 있는 경우에 오류가 발생한다.

타입 구조를 손 댈 수 없는 상황(ex. API 결과)일 때는 인터페이스 유니온을 사용해 속성 사이의 관계를 모델링 할 수 있다.
ex)
```typescript

interface Name{
	name: string;
}

interface PersonWithBirth extends Name {
	placeOfBirth: string;
	dateOfBirth: Date;
}

type Person = Name | PersonWithBirth;
```

</br>

---

# [아이템 33] string 타입보다 더 구체적인 타입 사용하기
> ✅ 요약
> 1. '문자열을 남발하여 선언된' 코드를 피하자. 모든 문자열을 할당할 수 있는 string 타입보다는 구체적인 타입을 사용하는 것이 좋다.
> 2. 변수의 범위를 보다 정확하게 표현하고 싶다면 string 타입보다는 문자열 리터럴 타입의 유니온을 사용하는 것이 좋다. 타입체크를 더 엄격히 할 수 있고, 생산성을 향상 시킬 수 있다.
> 3. 객체의 속성 이름을 함수 매개변수로 받을 때는 string보다 keyof T를 사용하는 것이 좋다.

</br>

`string` 타입의 범위는 매우 넓다.
'x','y' 같은 한 글자도, 약 120만 글자의 전체 내용도 모두 `string` 타입이다.
때문에, `string` 타입으로 변수를 선언하려 한다면, 그보다 더 좁은 타입이 적절하지 않을지
생각해봐야 한다.
</br>

**🔸string 타입의 문제점들**

**1. 의도와 다른 타입 설정**

ex)
```typescript
interface Album {
	artist: string;
	title: string;
	releaseDate: string; 		// YYYY-MM-DD;
	recordingType: string; 		// 'live' or 'studio'
}
```
위 코드는 `string` 코드가 남발되고 있다.
주석에 타입 정보를 적어 둔 걸 보면 현재 인터페이스가 잘못되었다는 것을 알 수 있다.

```typescript
const kindOfAlbum: Album {
	artist: 'aaaa';
	title: 'bbbb';
	releaseDate: 'August 17th, 1959'; 		
	recordingType: 'Studio'; 	
}
```
위 코드는 Album 타입의 객체에 엉뚱한 값을 설정하고 있다.
releaseDate 필드의 값은 주석에 나타난 형식과 다르고, recordingType 필드의 값은 대문자를 포함하고 있다. 하지만 이 두 값 모두 문자열이고, 해당 객체는 Album 타입에 할당 가능하며, 타입 체커를 통과한다.

</br>

**2. 매개변수 순서가 잘못된 오류가 드러나지 않음**


</br>

**🔸오류를 방지하기 위해 타입을 좁히는 방법** 

releaseDate 필드는 `Date 객체`를 사용해서 날짜 형식으로만 제한하는 것이 좋다.
recordingType 필드는 'live'와 'studio', 단 두 개의 값으로 유니온 타입을 정의할 수 있다.
```typescript
type RecordingType = 'live' | 'studio' 

interface Album {
	artist: string;
	title: string;
	releaseDate: Date;
	recordingType: RecordingType;
}
```
이렇게 변경하면 타입스크립트는 오류를 더 세밀하게 체크한다.

위와 같은 방식을 사용할 경우 3가지 장점이 있다.
1. 타입을 명시적으로 정의함으로써 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.
3. keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능하다.
	ex)
	```typescript
	function pluck<T, K exteds keyof T>(record: T[], key:K) { 
		return records.map(r => r[key]) 
	} 
		
	pluck(albums, 'releaseDate') // type Date[]
	pluck(albums, 'rr') // error 'rr'이 albums에 없다
	```

	keyof는 T타입의 모든 속성명을 타입으로 가지게 하며 위 코드처럼 별도의 제네릭을 추가하여 사용하지 않더라도 타입추론이 쉽게 된다.

</br>

---

# [아이템34] 부정확한 타입보다는 미완성 타입을 사용하기

> ✅ 요약
> 1.타입이 없는 것보다 잘못된 게 더 나쁘다.
> 2. 정확하게 타입을 모델링 할 수 없다면, 부정확하게 모델링하지 말아야한다.
	> -> 대략적으로 모델링하더라도 틀리게 모델링하지는 말아야 한다.	
> 3. `any` 와 `unknown`을 구별해서 사용해야 한다.
> 4. 타입 정보를 구체적으로 만들수록 오류 메시지와 자동 완성 기능에 주의를 기울여야 한다.

</br>

일반적으로 타입이 구체적일수록 버그를 더 많이 잡고, 타입스크립트가 제공하는 도구를 활용할 수 있다.
하지만 타입 선언의 정밀도를 높이는 일에는 주의를 기울여야 한다.
실수가 발생하기 쉽고, _**잘못된 타입은 차라리 타입이 없는 것보다 못할 수 있기 때문이다.**_


ex)
```typescript
interface Point {  
type: 'Point';  
coordinates: number[];  
}  
  
interface LineString {  
type: 'LineString';  
coordinates: number[][];  
}  
  
interface Polygon {  
type: 'Polygon';  
coordinates: number[][];  
}  
  
type Geometry = Point | LineString | Polygon; // 다른 것들도 추가될 수 있다

```

```typescript
type GeoPosition = [number, number];  
interface Point {  
 type: 'Point';  
 coordinates: GeoPosition;  
}
```
number[]가 추상적이라고 느껴 number[]를 경도와 위도를 나타내는 튜플 타입으로 선언하여 수정.
타입을 더 구체적으로 개선했기 때문에 더 나은 코드가 된 것 같지만 그렇제 않다.
코드에는 위도와 경도만을 나타냈지만, 위치 정보에는 고도나 다른 요소가 있을 수 있다.
이 코드를 사용하려면 사용자들은 as any를 추가해서 타입 체커를 무시하거나, 타입 단언문을 사용해야 한다.

-> 결과적으로 타입 선언을 세밀하게 만들고자 했지만 너무 과한 시도로 오히려 타입이 부정확해졌다.
