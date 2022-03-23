## 아이템 31 타입 주변에 null 값 배치하기

```tsx
function extent(nums: number[]) {
  let min, max;
  for (const num of nums) {
    if (!min) {
      min = num;
      max = num;
    } else {
      min = Math.min(min, num);
      max = Math.max(max, num);
    }
  }
  return [min, max];
}
```

- 최솟값이나 최댓값이 0인 경우 결과가 제대로 나오지 않음
    - extent([0,1,2])의 경우 결과가 [0,2]가 나와야하는데 [1,2]가 된다
- nums 배열이 비어 있다면 함수는 [undefined, undefined]를 반환한다.

```tsx
function extent(nums: number[]) {
  let result: [number, number] | null = null;
  for (const num of nums) {
    if (!result) {
      result = [num, num];
    } else {
      result = [Math.min(num, result[0]), Math.max(num, result[1])];
    }
  }
  return result;
}
```

- 반환타입이 `[number,number] | null` 이 되어서 사용하기가 더 수월해 졌다.

```tsx
interface UserInfo { name: string }
interface Post { post: string }
declare function fetchUser(userId: string): Promise<UserInfo>;
declare function fetchPostsForUser(userId: string): Promise<Post[]>;
class UserPosts {
  user: UserInfo | null;
  posts: Post[] | null;

  constructor() {
    this.user = null;
    this.posts = null;
  }

  async init(userId: string) {
    return Promise.all([
      async () => this.user = await fetchUser(userId),
      async () => this.posts = await fetchPostsForUser(userId)
    ]);
  }

  getUserName() {
    // ...?
  }
}
```

```tsx
interface UserInfo { name: string }
interface Post { post: string }
declare function fetchUser(userId: string): Promise<UserInfo>;
declare function fetchPostsForUser(userId: string): Promise<Post[]>;
class UserPosts {
  user: UserInfo;
  posts: Post[];

  constructor(user: UserInfo, posts: Post[]) {
    this.user = user;
    this.posts = posts;
  }

  static async init(userId: string): Promise<UserPosts> {
    const [user, posts] = await Promise.all([
      fetchUser(userId),
      fetchPostsForUser(userId)
    ]);
    return new UserPosts(user, posts);
  }

  getUserName() {
    return this.user.name;
  }
}
```

- 클래스를 만들 때는 필요한 모든 값이 준비되었을 때 생성하여 null이 존재하지 않도록 하는 것이 좋다.

## 아이템 32 유니온의 인터페이스보다는 인터페이스의 유니온을 사용하기

```tsx
interface Layer {
  layout: FillLayout | LineLayout | PointLayout;
  paint: FillPaint | LinePaint | PointPaint;
}
```

- layout이 LineLayout 타입이면서 paint 속성이 FillPaint 타입인것은 말이 되지 않는다
- 더 나은 모델링 방법은 각각 타입의 계층을 분리된 인터페이스로 둬야한다.

```tsx
interface FillLayer {
  layout: FillLayout;
  paint: FillPaint;
}
interface LineLayer {
  layout: LineLayout;
  paint: LinePaint;
}
interface PointLayer {
  layout: PointLayout;
  paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

- 이런 형태로 Layer를 정의하면 layout과 paint 속성이 잘못된 조합으로 섞이는 경우를 방지할 수 있다.

```tsx
interface Person {
  name: string;
  // These will either both be present or not be present
  placeOfBirth?: string;
  dateOfBirth?: Date;
}
```

```tsx
interface Person {
  name: string;
  birth?: {
    place: string;
    date: Date;
  }
}
```

- 여러 개의 선택적 필드가 동시에 값이 있거나 동시에 undefined인 경우도 태그된 유니온 패턴이 잘 맞다.

## 아이템 33 string 타입보다 더 구체적인 타입 사용하기

- string의 범위는 매우 넓다
- 더 좁은 타입이 적절하지 않을지 검토해야한다.

```tsx
interface Album {
  artist: string;
  title: string;
  releaseDate: string;  // YYYY-MM-DD
  recordingType: string;  // E.g., "live" or "studio"
}
const kindOfBlue: Album = {
  artist: 'Miles Davis',
  title: 'Kind of Blue',
  releaseDate: 'August 17th, 1959',  // Oops!
  recordingType: 'Studio',  // Oops!
};  // OK
```

- 날짜 형식이 다름
- recordingType 으로 "live" 또는 "studio" 를 받고싶었지만 S가 대문자인 Studio가 들어와도 오류가 나지 않는다.

```tsx
function recordRelease(title: string, date: string) { /* ... */ }
recordRelease(kindOfBlue.releaseDate, kindOfBlue.title);  // OK, should be error
```

- 매개변수 순서가 바뀐 경우도 정상으로 인식함

⇒ string 타입이 남용된 코드를 `stringly typed` 라고 표현되기도 한다.

```tsx
interface Album {
  artist: string;
  title: string;
  releaseDate: Date;
  recordingType: RecordingType;
}
```

이렇게 코드를 수정하면 오류를 더 세밀하게 체크한다.

세가지 장점이 더 있다.

1. 다른 곳으로 값이 전달되어도 타입 정보가 유지된다.
    
    함수를 사용하는 사람은 recordingType이 “studio” 또는 “live”여야한다는 것을 알 수 없다????
    
    ```tsx
    function getAlbumsOfType(recordingType: string): Album[] {
      // COMPRESS
      return [];
      // END
    }
    ```
    
2. 타입을 명시적으로 정의하고 해당 타입의 의미를 설명하는 주석을 붙여 넣을 수 있다.
3. keyof 연산자로 더욱 세밀하게 객체의 속성 체크가 가능해진다.
    
    
- string은 any와 비슷한 문제를 가지고 있다.
- 객체의 속성 이름을 함수 매개변수로 받을 때는 string보다 keyof T를 사용하는 것이 좋다.

## 아이템 34 부정확한 타입보다는 미완성 타입을 사용하기

```tsx
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
  coordinates: number[][][];
}
type Geometry = Point | LineString | Polygon;  // Also several others
```

```tsx
type GeoPosition = [number, number];
interface Point {
  type: 'Point';
  coordinates: GeoPosition;
}
```

GeoPosition 에는 세번째 요소인 고도가 있을 수 있고 또 다른 정보가 있을 수 도 있는데

타입 선언을 세밀하게 만들고자 했지만 시도가 너무 과했고 오히려 타입이 부정확해졌다.

- 타입 정보를 구체적으로 만들수록 오류 메시지와 자동 완성 기능에 주의를 기울여야한다.

## 아이템 35 데이터가 아닌, API와 명세를 보고 타입 만들기

- 우리가 다루는 타입 중 프로젝트 외부에서 비롯된 경우에는 타입을 직접 작성하지 않고 자동으로 생성할 수 있다.
- 여기서 핵심은, 예시 데이터가 아니라 명세를 참고해 타입을 생성한다는 것이다.
