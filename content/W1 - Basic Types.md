# TS Study(W1) - 기본 타입

## Boolean, Number, String

```ts
// Boolean
const boolType: boolean = false;

// Number
const decimal: number = 6;     // 10진수
const hex: number = 0xf00d;    // 16진수
const binary: number = 0b1010; // 2진수
const octal: number = 0o744;   // 8진수

// Number - bigint
// ES2020 - https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/BigInt
const big: bigint = 100n;

// String
const str: string = "TEST";
```

## Array

```ts
const numArr: number[] = [1,2,3,4];

// 제네릭 배열 타입
const numArr2: Array<number> = [1,2,3,4];
```

### (+) Generic array type

```ts
/// <reference no-default-lib="true"/>

interface Array<T> {
    /**
     * Returns the value of the first element in the array where predicate is true, and undefined
     * otherwise.
     * @param predicate find calls predicate once for each element of the array, in ascending
     * order, until it finds one where predicate returns true. If such an element is found, find
     * immediately returns that element value. Otherwise, find returns undefined.
     * @param thisArg If provided, it will be used as the this value for each invocation of
     * predicate. If it is not provided, undefined is used instead.
     */
    find<S extends T>(predicate: (this: void, value: T, index: number, obj: T[]) => value is S, thisArg?: any): S | undefined;
    find(predicate: (value: T, index: number, obj: T[]) => unknown, thisArg?: any): T | undefined;
		...
}
```

## Tuple

요소의 타입과 개수가 고정된 배열의 표현.

```ts
const sample: [string, number, boolean] = ['String', 123, false]

// Index를 이용한 요소 접근.
console.log(sample[0].substring(1));

// Err: Property 'substring' does not exist on type 'number'.
console.log(sample[1].substring(1)); 
```

## Enum

이름이 있는 상수들의 집합.

- TS에서는 숫자 + 문자열 기반 Enum을 제공.

```ts
// 숫자 열거형
enum EnumOne {
    FirstIndex, // 0
    SecondIndex,// 1
    ThirdIndex  // 2
}

// 문자열 열거형
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

- 컴파일된 열거형

```js
"use strict";

// 숫자 열거형은 String + Number indexed
var EnumOne;
(function (EnumOne) {
    EnumOne[EnumOne["FirstIndex"] = 0] = "FirstIndex";
    EnumOne[EnumOne["SecondIndex"] = 1] = "SecondIndex";
    EnumOne[EnumOne["ThirdIndex"] = 2] = "ThirdIndex"; // 2
})(EnumOne || (EnumOne = {}));

/**
EnumOne = {
	0: "FirstIndex",
	1: "SecondIndex",
	2: "ThirdIndex",
	FirstIndex: 0,
	SecondIndex: 1,
	ThirdIndex: 2
}
*/

// 문자열 열거형은 String indexed
var Direction;
(function (Direction) {
    Direction["Up"] = "UP";
    Direction["Down"] = "DOWN";
    Direction["Left"] = "LEFT";
    Direction["Right"] = "RIGHT";
    Direction["Default"] = "UP";
})(Direction || (Direction = {}));
```

## Unknown

애플리케이션을 만드는 당시에 알 수 없는 타입을 표현할 때 사용.

- User나 API로 부터 들어오는 동적인 값들

```ts
let notSure: unknown = 4;
notSure = "maybe a string instead";
notSure = false;
```

만약 `unknown` 타입인 변수가 있을 때, `typeof` 를 이용해서 더 고급 타입 비교를 할 수 있다.

```ts
// declare는 TSC에 해당 변수가 어딘가에 선언되어있음을 알려주는 선언. 
// 전역 변수나 d.ts 파일을 만들 때, 사용한다.
declare const maybe: unknown;

if (typeof maybe === "string") {
  // TypeScript가 maybe의 타입이 string임을 알고 있음.
  const aString: string = maybe;

  // maybe의 타입이 string임을 알기 때문에 boolean 타입에는 할당할 수 없음.
	// ERR - Type 'string' is not assignable to type 'boolean'.
  const aBoolean: boolean = maybe;
}
```

## Any

경우에 따라 Type 정보를 모두 알 수 없는 경우에 사용하는 타입.

- 3rd-party-library 혹은 TS로 작성되지 않은 코드를 사용
- 이미 JS로 작성된 코드가 존재할 때, 점진적으로 Type checking을 할 수 있도록 해줌.
- 모든 타입들은 `any` 타입의 서브 타입들이다.

```ts
declare function getValue(key: string): any;

// OK, return value of 'getValue' is not checked
const str: string = getValue("myString");
```

### any vs unknown

`any` 타입의 경우 `unknown` 타입과 다르게 임의의 프로퍼티에 접근할 수 있도록 한다.

- `any` 타입은 꼭 필요한 경우에만 사용하도록 하며 **Type safety를 해치는 것을 잊으면 안된다**.

```ts
let looselyTyped: any = 4;
looselyTyped.ifItExists();
looselyTyped.toFixed();

let strictlyTyped: unknown = 4;
// ERR - Object is of type 'unknown'.
strictlyTyped.toFixed();

```

## Void

`any` 타입의 반대. 어떤 타입도 갖지 않는다는 의미.

Return value가 없는 함수의 return 타입에 자주 사용됨.

- 변수를 `void` 타입으로 지정하면 null 혹은 undefined 밖에 할당할 수 없으므로 별 의미가 없다.

```ts
function warnUser(): void {
  console.log("This is my warning message");
}
```

## Null & Undefined

값이 `null`이거나 `undefined`일 때 사용. 이들 타입 자체로는 큰 의미는 없음.

- null & undefined는 모든 다른 타입의 `Subtypes`이므로 이들을 다른 타입에 할당하는 것이 가능하다.  (strictNullChecks를 사용하지 않는 경우 한정)

## Never

절대 발생할 수 없는 타입.

- 항상 오류를 발생시키거나 절대 값을 반환하지 않는 함수의 반환 타입
- 모든 타입에 할당 가능한 하위 타입이지만 어떤 타입도 `never` 타입에 할당할 수 없음.

```ts
// 항상 Error가 발생하기에 어떤 값도 반환하지 않는 함수
function testFunc () : never {
    throw new Error('Never')
}

// number 타입에 never를 할당.
let numberValue: number = testFunc()

// never에는 어떤 타입도 할당할 수 없음.
// ERR - Type 'number' is not assignable to type 'never'.
let neverValue: never = 1234
```

## Object

Primitive type이 아닌 타입. `object` 타입 사용시, `Object.create` 같은 API의 참조가 더 잘 일어난다.

- Primitive type : Number, String, Boolean, undefined, null, symbol

## Type assertions / 타입 단언

현재 타입보다 더 세부적인 타입을 알고 있는 경우 사용하는 기법.

다른 언어의 형변환과 유사하지만 특별히 검사를 하거나 데이터를 재구성 하지 않는다.

```ts
// as 문법을 사용. - JSX에서는 이 방법만을 허용함.
let someValue: unknown = "this is a string";
let strLength: number = (someValue as string).length;

// angle-bracket 문법을 사용.
let someValue: unknown = "this is a string";
let strLength: number = (<string>someValue).length;
```

## About Number, String, Boolean, Symbol and Object

Wrapper 객체는 매우 타입스럽게 생겼지만 절대 사용하지 않도록 한다.

## References

[Handbook - Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)

[TypeScript 한글 문서](https://typescript-kr.github.io/pages/basic-types.html)
