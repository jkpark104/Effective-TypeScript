# 7장 Item 55 ~ 57, 8장 Item 58 ~ 62


## Item 55 DOM 계층 구조 이해하기

> DOM 계층 구조를 파악하여 타입을 지정해줘야 한다.
> Node, Element, HTMLElement, EventTarget 간의 차이점
> Event와 MouseEvent 의 차이점
> DOM엘리먼트와 이벤트에는 충분히 구체적인 타입 정보를 사용하거나, 추론할 수 있도록 문맥 정보를 활용해야 한다.

예를 들어 `p` 요소에 대한 DOM 계층 구조는 다음과 같다.

`EventTarget` > `Node` > `Element` > `HTMLElement` > `HTMLParagraphElement`

그럼 다음 예제를 통해 각각의 계층별 타입 특징을 살펴보자.

```js
function handleDrag(eDown: Event) { 
	const targetEl = eDown.currentTarget; targetEl.classList.add('dragging'); 
	const dragStart = [eDown.clientX, eDown.clientY]; 
	const handleUp = (eUp: Event) => { 
		targetEl.classList.remove('dragging'); 
		targetEl.removeEventListener('mouseup', handleUp); 
		const dragEnd = [eUp.clientX, eUp.clientY]; 
		console.log('dx, dy = ', [0, 1].map(i => dragEnd[i] - dragStart[i])); 
	} 
	targetEl.addEventListener('mouseup', handleUp); 
} 
const div = document.getElementById('surface'); div.addEventListener('mousedown', handleDrag);
```

### DOM 요소별 계층

1. EventTarget
	1. DOM 타입 중 가장 추상화된 타입
	2. ex) window, XMLHttpRequest
	3. 이벤트 리스너를 추가, 제거, 보내는 것 밖에 할 수 없음
2. Node
	1. `요소.childeNodes` 로 얻을 수 있는 `NodeList` 안의 요소들이 Node다.
		1. `요소.children`은 `HTMLCollection`
	2. 엘리먼트 뿐만 아니라 텍스트, 주석 포함
	3. ex) document, Text, Comment
3. Element, HTMLElement
	1. SVG 태그의 전체 계충 구조를 포함하면서 HTML이 아닌 엘리먼트인 SVGElement가 있다. 
4. HTMLxxxElement
	1. HTMLImageElement, HTMLInputElement 처럼 각각 다른 속성(src, value)을 가지고 있는 구체적인 엘리먼트 타입이 있다.

DOM에 관해서는 타입스크립트보다 우리가 더 정확히 알고 있으니, 단언문을 사용해도 좋다.
strictNullChecks 설정 상태라면 HTMLDivElement가 null인 경우를 체크해주자.


### Event 계층
`Event` 하위에도 MouseEvent, KeyboardEvent 등 Mozilla 문서에만 52개 이상의 Event 종류가 있다. 보다 구체적인 (하위 계층)의 타입에 대한 속성을 사용하면서 상위 계층인 Event로 표시되어 있다면 에러가 발생한다. 즉, 보다 구체적인 이벤트 타입을 지정해 주어야 한다.

```js

function addHandleDragger(el: HTMLElement) { 
	el.addEventListener('mousedown', eDown => { // 인라인 함수로 만들기???
		const dragStart = [eDown.clientX, eDown.clientY]; 
		const handleUp = (eUp: MouseEvent) => { // 구체적인 디벤트 타입
			targetEl.classList.remove('dragging'); 
			targetEl.removeEventListener('mouseup', handleUp); 
			const dragEnd = [eUp.clientX, eUp.clientY]; 
			console.log('dx, dy = ', [0, 1].map(i => dragEnd[i] - dragStart[i])); 
		} 
		el.addEventListener('mouseup', handleUp); 
	});
} 

const div = document.getElementById('surface');

if(div){
	addDragHandler(div);
	// 혹은 단언문 addDragHandler(div!);
}
```


## Item 56 정보를 감추는 목적으로 private 사용하지 않기

> public, protected, private 접근 제어자는 타입 시스템에서만 강제되고 런타임에는 소용이 없을 뿐더러 단언문을 통해 우회할 수 있다.
> 따라서 접근 제어자로 데이터를 감추려고 해서는 안된다!
> 확실히 데이터를 감추고 싶다면 클로저를 사용하자.

TS에서는 접근 제어자(public, private, protected)를 사용할 수 있지만 컴파일 과정에서는 사라진다. 따라서 JS 코드에서는 접근할 수 있으며, TS의 단언문을 통해서도 우회할 수 있다.

따라서 타입체크와 런타임 모두에서 정보를 숨기기 위해서는
- 클로저
	- 생성자 외부에서 접근할 수 없기에 접근 메서드도 생성자 내부에 정의해야 한다.
	- 인스턴스 생성마다 접근 메서드가 복사되어 메모리를 낭비하게 된다.
	- 즉, 동일한 클래스 내에서도 서로의 비공개 데이터에 접근하는 것이 불가능하다.
- 접두사 `#`
	- Stage4 단계의 `#`를 이용한 비공개 필드 기능 [Class field declarations for JavaScript](https://github.com/tc39/proposal-class-fields)
	- 클래스 메서드나 동일한 클래스의 개별 인스턴스 끼리 접근이 가능하다.


## Item 57 소스맵을 사용하여 타입스크립트 디버깅하기
> 원본 코드가 아닌 변환된 JS 코드를 디버길하지 말자. 소스맵을 사용해서 런타임에 타입스크립트 코드를 디버깅하자.
> 소스맵이 최종적으로 변환된 코드에 완전히 매핑되었는지 확인하자.
> 소스맵에 원본 코드가 그대로 포함되지 않도록 설정하자.

```json
// tsconfig.json
{
	"compilerOptions": {
		"sourceMap": true
	}
}
```

tsconfig.json에 위의 설정을 추가하면 브라우저에서 index.ts 파일을 생성한다. 이를 통해 디버깅하면 된다.

# 8장 타입스트립트로 마이그레이션 하기

자바스크립트 프로젝트에서 발생하는 버그 중 상당 부분은 타입스크립트를 이용해 미연에 방지할 수 있었다는 다수의 참고 자료가 있다. (에어비앤비의 사후 분석 6개월치에서 발견된 버그의 38%가 TS에서 방지할 수 있는 것이었다고 함)

단, 한꺼번에 많은 코드를 타입스크립트로 전환할 수 없어 `점진적 마이그레이션`을 해야 한다.

또한 순수 자바스크립트에서 발생할 컴파일 오류 중 꺼둘 필요가 있는 것들이 있다.(noImplicitAny: off

점진적 마이그레이션을 하는 방법 다루기 start !

## Item 58 모던 자바스크립트로 작성하기
> TS 개발 환경은 모던 자바스크립트도 실행할 수 있으므로 최신 기능들을 적극적으로 사용하자 -> 코드 품질 향상, TS 추론 개선
> 

-   ECMAScript 모듈 사용
-   프로토타입 대신 클래스 사용
-   var대신 let/const 사용(호이스팅 관점에서)
-   for(;;)대신 for-of 또는 forEach 사용
-   함수 표현식보다 화살표 함수 사용( `noImplictThis` 또는 `strict`를 설정하면 오류 표시)
-   단축 객체 표현과 구조 분해 할당 사용
-   함수 매개변수 기본값 사용
-   저수준 프로미스나 콜백 대신 `async/await` 사용하기
-   연관 배열에 객체 대신 Map과 Set사용
-   `use strict` 넣지 않기 

## Item 59 타입스크립트 도입 전에 @ts-check와 JSDoc으로 시험해 보기
>  파일 상단에 // @ts-check를 추가하면 JS에서도 타입 체크를 수행할 수 있다.
>  전역 선언과 서드파티 라이브러리 타입 선언을 추가하는 방법을 익히자.
>  JSDoc 주석을 활용하면 JS상태에서도 타입 단언과 타입 추론을 할 수 있다.
>  JSDoc 주석은 중간 단계이므로 너무 공들이지는 말자. 
>  최종 목표는 `.ts`

## Item 60 allowJs 로 타입스크립트와 자바스크립트 같이 사용하기

## Item 61 의존성 관계에 따라 모듈 단위로 전환하기

## Item 62 마이그레이션의 완성을 위해 noImplicitAny 설정하기
> **마지막 단계!** 이를 설정하지 않으면 타입 선언과 관련된 실제 오류가 드러나지 않는다.
> 전면 적용하기 전 타입 오류를 점진적으로 수정해야 한다. 
> 엄격한 타입 체크 적용하기 전 팀원들이 TS에 익숙해질 수 있도록 시간을 줍시다.