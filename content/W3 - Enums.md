# TS Study(W3) - Enums

## Enums?

이름이 있는 상수 들의 집합. TS에서는 `숫자 / 문자열` 기반 Enum을 제공.

### Numeric enums / 숫자 열거형

숫자 열거형은 다음과 같다.

```tsx
enum EnumOne {
    FirstIndex, // 0
    SecondIndex,// 1
    ThirdIndex  // 2
}
```

컴파일된 숫자 열거형은 String Index와 Number Index를 모두 갖는 형태가 된다.

```jsx
// Compiled

"use strict";
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
		ThirdIndex: 2,
	}
	**/

console.log(EnumOne[0]) // FirstIndex
console.log(EnumOne.FirstIndex) // 0
```

숫자 열거형을 사용할 때, Computed Value와 Constants를 섞어서 사용할 수 있다.

다만 다음과 같은 경우에만 사용이 가능하다.

- `초기화 되지 않은 열거형이 먼저` 나옴
- `숫자 상수` 또는 `다른 상수 열거형 멤버와 함께 초기화된 숫자 열거형 이후`에 사용

```tsx
// 초기화 되지 않은 열거형이 먼저 나오는 케이스
enum Sample {
	ValueOne, // 0
	ValueTwo  // 1
}

// 숫자 상수를 이용해 초기화한 이후 사용하는 케이스
enum SampleTwo {
    ValueOne = 5, // 5
    ValueTwo,     // 6 (초기화된 값 이후로 자동으로 증가된 값을 갖음)
    ValueThird    // 7
}

// 다른 상수 열거형 멤버 + 숫자 초기화한 이후 사용하는 케이스
enum SampleThird {
    ValueOne = "TEST", // "TEST"
    ValueTwo = 7,      // 7
    ValueThird         // 8
}

// [ERR] - 숫자 상수를 이용하여 초기화 하지 않은 경우
enum EnumWrong {
		FirstIndex = 'First',// 'First'
    SecondIndex, // Err: Enum member must have initializer.(1061)
    ThirdIndex = 'Third'
}
```

열거형을 참조하여 사용하는 예제

```tsx
enum EnumNumeric {
	FirstIndex,
	SecondIndex
}

function testFunc( enumValue: EnumNumeric ): void {
	// ...
}

testFunc(EnumNumeric.SecondIndex)
```

### String enums / 문자열 열거형

문자열 열거형은 각 멤버들이 문자열 리터럴 또는 다른 문자열 열거형의 멤버로 상수 초기된 열거형.

- 숫자 열거형처럼 값이 자동 증가하는 기능은 없지만 Human readable한 값을 이용할 수 있는 장점이 있음.

```tsx
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
		// 다른 문자열 열거형의 멤버로 상수 초기화
		Default = Up   // "UP"
}
```

컴파일된 문자열 열거형은 String Index만을 갖는다.

- 숫자 열거형이 Number index를 갖는 것과 대조적.

```jsx
"use strict";
var Direction;
(function (Direction) {
    Direction["Up"] = "UP";
    Direction["Down"] = "DOWN";
    Direction["Left"] = "LEFT";
    Direction["Right"] = "RIGHT";
    Direction["Default"] = "UP";
})(Direction || (Direction = {}));
```

### Heterogeneous enums / 이종 열거형

숫자와 문자를 섞어놓은 열거형.

- 별로 추천하지 않음.

```tsx
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

```jsx
// Compiled
"use strict";
var BooleanLikeHeterogeneousEnum;
(function (BooleanLikeHeterogeneousEnum) {
    BooleanLikeHeterogeneousEnum[BooleanLikeHeterogeneousEnum["No"] = 0] = "No";
    BooleanLikeHeterogeneousEnum["Yes"] = "YES";
})(BooleanLikeHeterogeneousEnum || (BooleanLikeHeterogeneousEnum = {}));
```

## Computed and constant members / 계산된 멤버와 상수 멤버

각 열거형의 멤버는 상수(Constant)이거나 계산된 값(Computed)일 수 있다.

열거형의 멤버는 `상수 열거형 표현식(constant enum expression)`으로 초기화 된다.

- 상수 열거형 표현식: 컴파일 시 연산이 일어나는 TS 표현식의 일부 (A constant enum expression is a subset of TypeScript expressions that can be fully evaluated at compile time)

다음과 같은 경우를 상수 열거형 표현식이라고 한다.

- 리터럴 열거형 표현식. (문자 / 숫자 리터럴)
- 이전에 정의된 다른 상수 열거형 멤버에 대한 참조.
    - 숫자 상수를 이용해 초기화한 이후 사용하는 케이스
- 괄호로 묶인 상수 열거형 표현식
- 상수 열거형 표현식에 단항 연산자를 사용하는 경우
    - `+, -, ~`
- 상수 열거형 표현식을 이중 연산자의 피연산자로 사용하는 경우
    - `+, -, *, /, %, <<, >>, >>>, &, |, ^`
    - 이 경우, 상수 열거형 표현식의 값이 `NaN` 이거나 `Infinity` 인 경우, 에러가 발생.

```tsx
enum FileAccess {
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // Fully evaluated at compile time / Computed value
    G = "123".length
}
```

## Union enums and enum member types / 유니언 열거형과 열거형 멤버 타입

열거형 멤버는 상수이거나 계산된 값이 와야하지만 계산되지 않은 특수한 케이스가 있음.

### Literal enum members

리터럴 열거형 멤버는 `초기화 값이 존재하지 않거나`, 아래와 같이 초기화 되는 멤버를 의미한다.

- 문자 리터럴
- 숫자 리터럴
- 숫자 리터럴에 `-` 단항 연산자가 적용된 경우 (e.g `-1`, `-100`)

열거형의 모든 멤버가 리터럴 열거형 값을 가지게 되면 특별한 의미로 사용되는데

1. `열거형 멤버를 타입`처럼 사용할 수 있다.

    ```tsx
    enum ShapeKind {
        Circle,
        Square,
    }

    interface Circle {
        kind: ShapeKind.Circle;
        radius: number;
    }

    interface Square {
        kind: ShapeKind.Square;
        sideLength: number;
    }

    let c: Circle = {
    		// 오류! 'ShapeKind.Circle' 타입에 'ShapeKind.Square' 타입을 할당할 수 없습니다.
        kind: ShapeKind.Square,
        radius: 100,
    }
    ```

2. 열거형 타입 자체가 효율적인 `유니언`이 될 수 있다.

    ```tsx
    enum E {
        Foo,
        Bar,
    }

    function f(x: E) {
        // 에러! E 타입은 Foo, Bar 둘 중 하나이기 때문에 이 조건은 항상 true를 반환합니다.
        // This condition will always return 'true' 
    		// since the types 'E.Foo' and 'E.Bar' have no overlap.
        if (x !== E.Foo || x !== E.Bar) {
    			...
        }
    }
    ```

## Enums at runtime / 런타임에서 열거형

열거형은 런터임에 존재하는 실제 객체.

```jsx
"use strict";
var EnumOne;
(function (EnumOne) {
    EnumOne[EnumOne["FirstIndex"] = 0] = "FirstIndex";
    EnumOne[EnumOne["SecondIndex"] = 1] = "SecondIndex";
    EnumOne[EnumOne["ThirdIndex"] = 2] = "ThirdIndex"; // 2
})(EnumOne || (EnumOne = {}));

////////////////////////////////////////////////

function f(obj: { FirstIndex: number }) {
    return obj.FirstIndex;
}

// EnumOne가 FirstIndex라는 숫자 프로퍼티를 가지고 있기 때문에 동작하는 코드입니다.
f(EnumOne);
```

## Enums at compile time / 컴파일 시점에서 열거형

열거형이 런타임에 실존하는 객체라도 일반적인 객체에 `keyof` 로 기대하는 동작이 일어나지 않는다.

대신 `keyof typeof` 를 사용하면 모든 열거형의 키를 문자열로 나타내는 타입을 가져온다.

```tsx
enum LogLevel {
    ERROR, WARN, INFO, DEBUG
}

// ERR - Cannot find name 'keyof'.
console.log(keyof LogLevel)

/**
 * 이것은 아래와 동일합니다. :
 * type LogLevelStrings = 'ERROR' | 'WARN' | 'INFO' | 'DEBUG';
 */
type LogLevelStrings = keyof typeof LogLevel;
```

### Reverse mappings / 역 매핑

숫자 열거형의 경우, 멤버 프로퍼티 이름을 가진 객체를 생성하는 것 외에도 `열거형 값에서 열거형 이름으로 역 매핑`을 받는다.

- 문자열 열거형은 역매핑을 생성하지 않음.

```tsx
enum Enum {
    A
}

let a = Enum.A;        // 0
let nameOfA = Enum[a]; // "A"
```

```jsx
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));

var a = Enum.A;
var nameOfA = Enum[a];
```

### Const enums

간접 참조를 허용하지 않는 엄걱한 요구사항을 만족시킬 때, `const enums` 를 사용할 수 있다.

```tsx
const enum Enum {
    A = 1,
    B = A * 2
}

let test = [Enum.A, Enum.B]
```

```tsx
declare const enum Enum {
    A = 1,
    B = 2
}
declare let test: Enum[];
```

```jsx
// Enum이 객체화되지 않았음.
"use strict";
let test = [1 /* A */, 2 /* B */];
```

## Ambient enums

이미 존재하는 열거형 타입을 묘사하기 위해서 `Ambient enum`을 사용한다.

```tsx
// declare 키워드를 사용하였음을 잊지말자.
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

```tsx
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

```jsx
// Enum 객체가 생성되지 않았음을 확인할 수 있다.
"use strict";
```

이때, 코드 상에서 Enum 객체를 사용하는 경우 컴파일 과정에서는 문제가 없을 수 있지만 런타임에 실제로 Abmbient enum 객체가 존재하지 않으면 에러가 발생한다.

```tsx
declare enum Enum {
    A = 1,
    B,
    C = 2
}

const testValue = 5

function testFunc (value: Enum) {
    if (value === Enum.A) console.log('Matched with A')
    if (value === Enum.B) console.log('Matched with B')
    if (value === Enum.C) console.log('Matched with C')
    
    console.log('Not matched. :(')
}

testFunc(testValue)
```

```tsx
"use strict";
const testValue = 5;
function testFunc(value) {
    if (value === Enum.A)
        console.log('Matched with A');
    if (value === Enum.B)
        console.log('Matched with B');
    if (value === Enum.C)
        console.log('Matched with C');
    console.log('Not matched. :(');
}

// [ERR]: "Executed JavaScript Failed:" 
// [ERR]: Can't find variable: Enum
testFunc(testValue);
```

## References

[TypeScript 한글 문서](https://typescript-kr.github.io/pages/enums.html)

[Handbook - Enums](https://www.typescriptlang.org/docs/handbook/enums.html)

[Introduction to TypeScript Enums - Const and Ambient Enums](https://levelup.gitconnected.com/introduction-to-typescript-enums-const-and-ambient-enums-1fe686b67495)
