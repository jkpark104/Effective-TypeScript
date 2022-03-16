# 2022년 3월 16일 한결 공부내용


## 아이템 31: 타입 주변에 null 값 배치하기
- null과 null이 아닌 값이 뒤섞여 있으면 문제가 발생하기 쉽다.
- strictNullCheck 설정하고 값이 전부 null이거나 전부 null이 아닌 경우로 분명히 구분되게 하자.
- 예컨대 함수의 반환값이 `[undefined, undefined]`가 아니라 `null`이 되도록 처리
- 클래스를 만들 때, 네트워크 요청을 통해 값을 갱신할 때에도 null로 초기화한 후 값을 가져오는 형태가 아니라 모든 값이 준비되었을 때 `MyClass.init` 메서드를 통해 인스턴스를 생성하도록.

## 아이템 32: 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기
- 서로 상응하는 조합으로 각 필드의 타입이 지정되어야 하는 경우, 각각 타입 계층을 분리된 인터페이스로 두고 이를 유니온으로 묶어 타입정의를 한다.
- 타입 속성들 간의 관계를 제대로 모델링하여 잘못된 조합으로 섞이는 경우를 방지한다.
- 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 없는 경우, 해당 필드들을 하나의 객체로 묶는다.
    ```tsx
    // Bad
    interface Person {
        name: string;
        placeOfBirth?: string;
        dateOfBirth?: Date;
    }
    
    // Good
    interface Person {
        name: string;
        birth ?: { 
            placeOfBirth: string;
            dateOfBirth: Date;
        }
    }
    ```
    
    - 타입의 구조를 손댈 수 없는 상황에서는 확장 후 확장된 타입을 인터페이스 유니온을 사용하여 모델링한다.
    
    ```tsx
    interface Name {
        name: string;
    }
    
    interface PersonWithBirth extends Name{
        birth: { 
            placeOfBirth: string;
            dateOfBirth: Date;
    }
    
    type Person = Name | PersonWithBirth;
    ```
    
## 아이템 33: string 타입보다 더 구체적인 타입 사용하기
- string 타입의 범위는 매우 넓으므로 string 타입을 남발하면 매개변수 순서가 잘못되거나 형식이 달라도 타입체커를 통과한다는 문제점이 있다.
- 적절한 형식의 타입을 지정하고, 타입의 범위를 좁혀 오류를 세밀하게 체크하도록 한다.
- 객체의 속성 이름을 함수 매개변수로 받을 때는 `keyof` 연산자를 통해, 해당 타입 안에 있는 필드로 제한하여 세밀하게 객체의 속성 체크를 할 수 있다.

## 아이템 34: 부정확한 타입보다는 미완성 타입을 사용하기
- 타입 선언의 정밀도를 높이다 보면 오히려 타입이 부정확해지고 오류 메시지가 부정확해져 개발경험을 해친다.

## 아이템 35: 데이터가 아닌, API와 명세를 보고 타입 만들기
- API 등에서 데이터를 받는 경우 자동으로 생성되는 타입 정보를 활용해야 정확히 타입을 정의할 수 있으며, 쿼리나 스키마가 바뀐다면 타입도 자동으로 바뀐다.
- 데이터 자체보다는 명세를 참고해서 타입을 정의해야 한다.