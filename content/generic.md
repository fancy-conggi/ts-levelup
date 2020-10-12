# Generic

사용자는 제네릭을 통해 여러 타입의 컴포넌트나 자신만의 타입을 사용할 수 있습니다.



## Generic 선언

`any`를 쓰는 것은 함수의 `arg`가 어떤 타입이든 받을 수 있다는 점에서 제네릭이지만,

실제로 함수가 반환할 때 어떤 타입인지에 대한 정보는 잃음

만약 number 타입을 넘긴다고 해도 any 타입이 반환된다는 정보를 얻음

```ts
function identity(arg: number): number {
    return arg;
}

function identity(arg: any): any {
    return arg;
}
```



값이 아닌 타입에 적용되는 `타입변수(제네릭)`를 사용

T라는 타입 변수는 유저가 준 타입을 캡처하고 ( ex. number ) 이 정보를 나중에 사용

아래 예제에서는 반환타입으로 다시 사

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

PS. 컴파일후에는 제레릭에 대한 어떠한 검증도 없음

```javascript
"use strict";
function identity(arg) {
    return arg;
}
```



## Generic 호출

[방법1] `타입 인수 전달`

```ts
let output = identity<string>("myString"); // 출력 타입은 'string'입니다.
```

[방법 2] `타입 인수 추론`

`"myString"`를 보고 그것의 타입으로 `T`를 정함

인수 추론은 코드를 간결하고 가독성있게 하지만 

복잡한 예제에서 컴파일러가 타입을 유추할 수 없는 경우에 명시적인 타입 인수 전달이 필요할 수도 ..

```ts
let output = identity("myString"); //출력 타입은 'string'입니다.
```



## Generic 배열

함수 호출 시마다 인수 `arg`의 길이를 로그에 찍으려면 ?

```ts
function loggingIdentity<T>(arg: T): T {
  console.log(arg.length); // 오류: T에는 .length 가 없습니다. Compiler-Error
  return arg;
}
```

T라는 타입에 .length가 있다는 것을 보장할 수 없음

```typescript
function loggingIdentity<T>(arg: T[]): T[] {
  console.log(arg.length); // 배열은 .length를 가지고 있습니다. 따라서 오류는 없습니다.
  return arg;
}

function loggingIdentity<T>(arg: Array<T>): Array<T> {
  console.log(arg.length); // 배열은 .length를 가지고 있습니다. 따라서 오류는 없습니다.
  return arg;
}
```



## Generic 타입

### Generic 인터페이스

제네릭 함수의 타입은 함수 선언과 유사하게 타입 매개변수가 먼저 나열되는, 비-제네릭 함수의 타입과 비슷

```ts
function identity<T>(arg: T): T {
  return arg;
}

let myIdentity_A: <T>(arg: T) => T = identity;

// 제네릭 타입을 객체 리터럴 타입의 함수 호출 시그니처로 작성하는 병우
let myIdentity_B: { <T>(arg: T): T } = identity;
```



앞선 것들로 제네릭 인터페이스 작성하기

`myIdentity_B` : 

 제네릭 함수를 작성하는 것 대신 제네릭 타입의 일부인 비-제네릭 함수 시그니처를 가짐

함수를 사용할 때, 시그니처가 사용할 것을 효과적으로 제한할 특정한 타입 인수가 필요합니다 (여기서는 `number`)

```ts
interface GenericIdentityFn {
  <T>(arg: T): T;
}

function identity<T>(arg: T): T {
  return arg;
}

let myIdentity_A: GenericIdentityFn = identity;
let myIdentity_B: GenericIdentityFn<number> = identity;
```



### Generic 클래스

#### 1. 클래스가 Generic인 경우

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };

let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };
```



> 클래스는 static 측면과, instance 측면이 있는데
>
>  제네릭 클래스는 instance 측면에서만 제네릭임으로 클래스로 작업할 때 static 기능은 사용할 수 없음



#### 2. 클래스가 Generic의 타입변수인 경우

클래스를 넘겨서 인스턴스를 받는 (Factory 타입) 경우

다음과 같이 타입을 사용`{new(): T; }`

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```



## Generic 제약조건 

### 제약조건1. extends

특정 타입으로만 동작하는 제네릭 함수

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // 이제 .length 프로퍼티가 있는 것을 알기 때문에 더 이상 오류가 발생하지 않습니다.
    return arg;
}

loggingIdentity(3);  // 오류, number는 .length 프로퍼티가 없습니다.
loggingIdentity({length: 10, value: 3});
```



### 제약조건2. keyof

이름이 있는 객체에서 프로퍼티를 가져오고 싶은 경우

`keyof x`는 사실상 `a | b | c | d`와 같습니다. 

```ts
// K extends keyof T (K는 T의 하위 속성중 하나다)
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // 성공
getProperty(x, "m"); // 오류: 인수의 타입 'm' 은 'a' | 'b' | 'c' | 'd'에 해당되지 않음.
```



`keyof` 의 또 다른 활용

````ts
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
````



