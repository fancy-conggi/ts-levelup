# literal

리터럴 타입은 집합 타입의 보다 `구체적인 하위 타입`입니다

이것이 의미하는 바는 타입 시스템 안에서 `"Hello World"`는 `string`이지만, `string`은 `"Hello World"`가 아니란 것입니다



## Constant vs Literal

### Constant

상수는 변하지 않는 변수를 뜻함

상수는 숫자, 문자만 올 수 있는 것이 아니라 클래스나 구조체와 같이 기본형에서 파생된 객체도 넣을 수 있음



상수는 데이터가 변하지 안한아야 한다는 뜻은 참조변수 메모리의 주소값이 변하지 않는다는 의미

그 주소가 가르키는 데이터들까지 상수라는 의미는 아님



### Literal

리터럴은 데이터 자체를 뜻한다. 변수에 넣은 변하지 않는 `데이터`를 의미

앞의 a가 상수라면 뒤의 1이 리터럴이 된다.

```javascript
const a = 1
```

즉, 1과 같이 변하지 않는 데이터 (boolean, char, int, long, double, ... )를 리터럴이라 부른다.



#### 객체리터럴

> 그럼 인스턴스가 리터럴이 될 수 있을까?
>
> => 아니요

하지만 객체 리터럴이란 표현이 있다 

데이터가 변하지 않도록 설계한 클래스를 불변 클래스(immutable class)라 칭하고 

해당 클래스는 한번 생성하면 객체 안의 데이터가 변하지 않음

변하는 상황이라면 새로운 객체가 생성된다. String, Color와 같은 객체가 이와 같다.

따라서 우리는 "abc"와 같은 문자열을 `객체 리터럴(리터럴)`이라 부름



## var, let vs const

`var` 또는 `let`으로 변수를 선언할 경우 이 변수의 값이 변경될 가능성이 있음을 컴파일러에게 알립니다. 

반면, `const`로 변수를 선언하게 되면 TypeScript에게 이 객체는 절대 변경되지 않음을 알립니다.

```ts
// 따라서, TypeScript는 문자열이 아닌 "Hello World"로 타입을 정합니다.
const helloWorld = "Hello World";

// 반면, let은 변경될 수 있으므로 컴파일러는 문자열이라고 선언할 것입니다.
let hiWorld = "Hi World";
```



## 문자열 리터럴 타입 (String Literal Types)

문자를 타입처럼...



실제로 문자열 리터럴 타입은 유니언 타입, 타입 가드 그리고 타입 별칭과 잘 결합됩니다.

이런 기능을 함께 사용하여 `문자열로 enum과 비슷한 형태를 갖출 수 있음`

```ts
// @errors: 2345
type Easing = "ease-in" | "ease-out" | "ease-in-out";

class UIElement {
  animate(dx: number, dy: number, easing: Easing) {
    if (easing === "ease-in") {
      // ...
    } else if (easing === "ease-out") {
    } else if (easing === "ease-in-out") {
    } else {
      // 하지만 누군가가 타입을 무시하게 된다면
      // 이곳에 도달하게 될 수 있습니다.
    }
  }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // Compile-Error
```



## 숫자형 리터럴 타입  (Numeric Literal Types)

숫자를 타입처럼... 



**CASE1**

컴파일 단계에서의 에러를 발생하기 위함일 뿐,

JavaScript내에서 제약은 없음

```ts
function rollDice(): 1 | 2 | 3 | 4 | 5 | 6 {
  return (Math.floor(Math.random() * 6) + 1) as 1 | 2 | 3 | 4 | 5 | 6;
}

const result = rollDice();
```

```javascript
"use strict";
function rollDice() {
    return (Math.floor(Math.random() * 6) + 1);
}
const result = rollDice();
```



**CASE2**

만약 tileSize에 8, 16, 32이 외의 숫자가 들어가면 컴파일 에러 발생

```ts
/** loc/lat 좌표에 지도를 생성합니다. */
declare function setupMap(config: MapConfig): void;

// ---생략---
interface MapConfig {
  lng: number;
  lat: number;
  tileSize: 8 | 16 | 32;
}

setupMap({ lng: -73.935242, lat: 40.73061, tileSize: 16 });
```