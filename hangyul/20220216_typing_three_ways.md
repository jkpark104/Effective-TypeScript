# 타입을 정의하는 방법

## 타입 별칭(Type Alias)
- 똑같은 타입을 한 번 이상 재사용하거나 또 다른 이름으로 부르고 싶은 경우 정의
- 새로운 프로퍼티(필드) 추가 불가능, `&`로 확장된 새로운 별칭을 정의할 수는 있음
    
```tsx
type Square = { width: number };
type Square = { height: number };
// 에러메시지: Duplicate identifier 'Square'.
    
type Rectangle = { height: number } & Square;
    
const box: Rectangle = { width: 16, height: 24 }
```
    
## 인터페이스(Interface) 선언
- 객체 타입의 shape을 선언하는 것으로, 원시 타입에 별칭을 부여하는 용도로는 사용할 수 없음
- 확장이 가능하며 동일한 인터페이스 이름으로 재정의하면 해당 인터페이스의 새로운 필드로 추가
    
```tsx
interface Square { 
  width: number;  
}
    
interface Square { 
  height: number;
} 
// { width: number, height: number } 인터페이스로 확장됨
    
// 위 height 타입 정의 코드가 없다 치고!
interface Rectangle extends Square { 
  height: number; 
}

const box: Rectangle = { width: 16, height: 24 }
```
    
## 클래스(Class)

- 위의 두 경우는 런타임에 참조할 수 없는 erasable(제거가능한) 타입
    
```tsx
type Shape = Square | Rectangle;
    
function calculateArea(shape: Shape){
  if (shape instanceOf Rectangle){ // Rectangle이 값으로 사용될 수 없음
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```
    
- 런타임에도 타입 정보를 유지해야 할 경우 취할 수 있는 방법
   - ‘height’ 속성이 매개변수인 shape 객체에 있는지 확인하는 코드를 작성한다

        ```tsx
        function caculateArea(shape: Shape){
          if ('height' in shape){ 
            return shape.width * shape.height;
          } else {
            return shape.width * shape.width;
          }
        }
        ```
        
  - Tag 기법으로 런타임에 접근 가능한 정보를 명시적으로 저장한다.      
  
        ```tsx
        interface Square { 
          kind: 'square';
          width: number;
        } 
        // 위 height 타입 정의 코드가 없다 치고!
        interface Rectangle { 
          kind: 'square';
          width: number;
          height: number;
        } 
        
        function caculateArea(shape: Shape){
          if (shape.kind === 'rectangle'){ 
            return shape.width * shape.height;
          } else {
            return shape.width * shape.width;
          }
        }
        ```
        
- 타입을 클래스로 만들면 타입(런타임 접근불가)과 값(런타임 접근가능)을 둘다 사용할 수 있다!       

        ```tsx
        class Square {
          constructor(public width: number) {}
        }
        
        class Rectangle extends Square {
          constructor(public width: number, public height: number){
            super(width);
          }
        }
        
        type Shape = Square | Rectangle; // 타입으로 참조
        
        function calculateArea(shape: Shape) {
          if (shape instanceof Rectangle) { // Rectangle 클래스 값으로 참조
            return shape.width * shape.height;
          } else {
            return shape.width * shape.width;
          }
        }
        ```
        
  - 인터페이스는 타입으로만 사용가능하지만 클래스는 타입과 값으로 모두 사용할 수 있으며, 어떻게 참조되는지 구분하는 것은 매우 중요한데 이는 아이템 8에서 다룬다고 합니다. 
        
## 결론
- `interface`가 가지는 대부분의 기능은 `type`에서도 동일하게 사용이 가능하다.
- 우선 `interface`를 사용하고 이후 문제가 발생하였을 때 `type`을 사용하는 것이 권유된다.
- 구조적으로 클래스의 인스턴스와 타입 선언된 객체에는 차이가 없다. Duck Typing에 의해서 객체 안에 해당 타입이 되기 위해 필요한 모든 속성이 존재한다면, 추가 속성에 관계없이 일치하는 타입으로 간주한다.
- 클래스로 정의한 값을 해당 인스턴스의 객체 형태를 가진 타입의 이름으로 사용 가능하므로 클래스를 활용하여 타입과 값으로 모두 사용할 수 있다.
