# TS Study(W1) - Interface

## Interface?

타입 체크를 위해 사용되는 새로운 타입.

- 코드 안의 구조를 정의함.
- 프로토콜과 유사.

## Interface sample

- 인터페이스를 적용하기 이전의 코드 예제.

```ts
// printLabel의 호출 시, 타입 검사를 수행.
// string 타입의 label 프로퍼티를 갖는 객체가 매개변수로 입력받는지 확인.
// 이때, 이 객체가 실제로 더 많은 프로퍼티를 가지고 있더라도 컴파일러에서는 최소한 필요한 프로퍼티만 확인.
function printLabel(labeledObj: { label: string }) {
    console.log(labeledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

- 인터페이스를 적용한 코드 예제.

```ts
interface LabeledValue {
    label: string;
}

// 이제 매개변수의 구조를 명시적으로 표현하지 않고 Interface를 통해 표현.
// Type checker는 Interface의 프로퍼티 순서를 확인하지 않으며 존재 여부와 타입만을 확인함.
function printLabel(labeledObj: LabeledValue) {
    console.log(labeledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

- Compiled

```js
// js
"use strict";
function printLabel(labeledObj) {
    console.log(labeledObj.label);
}
let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

```ts
// .d.ts
// (+) .d.ts 는 타입스크립트 코드의 타입 추론을 돕는 파일.
interface LabeledValue {
    label: string;
}
declare function printLabel(labeledObj: LabeledValue): void;
declare let myObj: {
    size: number;
    label: string;
};
```

## Optional Properties / 선택적 프로퍼티

Interface에서 `특정 조건에서만 존재`하거나 `아예 없는 프로퍼티를 사용`하는 경우 이를 처리하는 방법.

- 프로퍼티의 이름 끝에 `?` 를 붙여서 표현.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

## Readonly Properties / 읽기전용 프로퍼티

Interface를 따르는 객체의 생성 이후에 값을 수정할 수 없도록 하는 방법.

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}

let p1: Point = { x: 10, y: 20 };

// ERR - Cannot assign to 'x' because it is a read-only property.
p1.x = 5;
```

```ts
// .d.ts
interface Point {
    readonly x: number;
    readonly y: number;
}
declare let p1: Point;
```

### ReadonlyArray

생성 이후 수정할 수 없는 형태의 배열을 표현하는 방법.

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;

// ERR - Index signature in type 'readonly number[]' only permits reading.
ro[0] = 12;

// ERR - Property 'push' does not exist on type 'readonly number[]'.(2339)
ro.push(5);

// ERR - Gets the length of the array. This is a number one higher than the highest element defined in an array.
ro.length = 100;

// ERR - The type 'readonly number[]' is 'readonly' and cannot be assigned to the mutable type 'number[]'.(4104)
a = ro;

// Type assertion으로 오버라이딩하는 것은 가능!
a = ro as number[];
```

- `Array<T>` 에서 모든 변경 메서드가 제거된 타입
    - EX > push, shift, pop, unshift...

```ts
// lib.es5.d.ts

/////////////////////////////
/// ECMAScript Array API (specially handled by compiler)
/////////////////////////////

interface ReadonlyArray<T> {
    readonly length: number;
    toString(): string;
    toLocaleString(): string;
    concat(...items: ConcatArray<T>[]): T[];
    concat(...items: (T | ConcatArray<T>)[]): T[];
    join(separator?: string): string;
    slice(start?: number, end?: number): T[];
    indexOf(searchElement: T, fromIndex?: number): number;
    lastIndexOf(searchElement: T, fromIndex?: number): number;
    every<S extends T>(predicate: (value: T, index: number, array: readonly T[]) => value is S, thisArg?: any): this is readonly S[];
    every(predicate: (value: T, index: number, array: readonly T[]) => unknown, thisArg?: any): boolean;
    some(predicate: (value: T, index: number, array: readonly T[]) => unknown, thisArg?: any): boolean;
    forEach(callbackfn: (value: T, index: number, array: readonly T[]) => void, thisArg?: any): void;
    map<U>(callbackfn: (value: T, index: number, array: readonly T[]) => U, thisArg?: any): U[];
    filter<S extends T>(predicate: (value: T, index: number, array: readonly T[]) => value is S, thisArg?: any): S[];
    filter(predicate: (value: T, index: number, array: readonly T[]) => unknown, thisArg?: any): T[];
    reduce(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: readonly T[]) => T): T;
    reduce(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: readonly T[]) => T, initialValue: T): T;
    reduce<U>(callbackfn: (previousValue: U, currentValue: T, currentIndex: number, array: readonly T[]) => U, initialValue: U): U;
    reduceRight(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: readonly T[]) => T): T;
    reduceRight(callbackfn: (previousValue: T, currentValue: T, currentIndex: number, array: readonly T[]) => T, initialValue: T): T;
    reduceRight<U>(callbackfn: (previousValue: U, currentValue: T, currentIndex: number, array: readonly T[]) => U, initialValue: U): U;

    readonly [n: number]: T;
}
```

### `readonly` vs `const`

변수는 `const` 를 사용하고 프로퍼티는 `readonly`를 사용한다.

## Excess Property Checks / 초과 프로퍼티 검사

이전 설명에서 아래의 2가지를 배웠다.

- Interface를 이용한 타입 체크에서 최소한 필요한 프로퍼티만 검사함.
- Optional을 이용한 선택적 프로퍼티.

하지만 이 둘을 결합하는 경우에 문제가 생길 수 있다.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

위의 코드는 다음과 같은 이유로 문제가 없는 듯 보이며 실제로 JS로 같은 코드를 실행한다면 문제가 없다.

- createSquare의 매개변수 config는 interface SquareConfig를 이용해 타입 검사를 수행.
- `colour`는 타입이 명시된 프로퍼티는 아니지만 최소한의 프로퍼티를 검사하는 경우, 문제가 없어 보임.
    - 최소 조건: SquareConfig의 모든 프로퍼티는 Optional임.

타입 스크립트에서는 이 코드에 문제가 있다고 판단하는데 그 이유는  `객체 리터럴`이 다른 변수에 할당하거나 인수로 전달할 때, 특별한 처리를 받으며 `Excess Property Checking` 을 받기 때문이다.

- `Excess Property Checking` : 대상 타입이 갖고 있지 않은 프로퍼티를 갖는 경우, 에러가 발생.

```ts
/**
[ERR]

Argument of type '{ colour: string; width: number; }' is not assignable 
to parameter of type 'SquareConfig'.
Object literal may only specify known properties, 
but 'colour' does not exist in type 'SquareConfig'. 
Did you mean to write 'color'?
*/
let mySquare = createSquare({ colour: "red", width: 100 });
```

이 문제를 해결할 수 있는 방법은 다음과 같다.

- Type assertion

```ts
let mySquare = createSquare({ colour: "red", width: 100 } as SquareConfig);
```

- String index sginature(문자열 인덱스 서명) - 추가 프로퍼티가 있음을 확신하는 경우

```ts
interface SquareConfig {
    color?: string;
    width?: number;
		// String index signature
		// - color, width를 제외한 프로퍼티가 있을 수도 있으며, 그럴 경우 타입이 중요하지 않다.
    [propName: string]: any;
}
```

- 객체를 변수에 할당하여 Excess property checking을 피하는 방법

```ts
// 객체 리터럴이 아닌 경우 Excess property checking를 받지 않음.
const square = { colour: "red", width: 100 }
let mySquare = createSquare(square);
```

간단한 코드라면 위의 방법으로 검사를 피할 수 있지만, Excess property checking에서 발생하는 에러는 대부분 버그이기 때문에 Interface를 적절하게 수정하는 편이 더 좋은 방법이다.

## Function Types / 함수 타입

Interface는 프로퍼티를 이용하여 객체를 기술하는 것 이외에도 함수의 타입을 설명할 수 있다.

```ts
interface SearchFunc {
		// 함수 타입을 기술하기 위해 호출 서명(call signature)을 전달.
		// 이때, 각 매개변수는 이름+타입이 모두 필요하다.
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;

// 실제 함수 구현 시, 매개변수의 이름이 Interface와 같을 필요는 없다.
mySearch = function(src: string, sub: string) {
    let result = source.search(subString);
    return result > -1;
}
```

```js
// Compiled

"use strict";
let mySearch;
mySearch = function (source, subString) {
    let result = source.search(subString);
    return result > -1;
};
```

```ts
// .d.ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}
declare let mySearch: SearchFunc;
```

- Interface를 사용하지 않고 만든 함수의 케이스

```ts
function mySearch2 (source: string, subString: string) {
    let result = source.search(subString);
    return result > -1;
}

// .d.ts
declare function mySearch2(source: string, subString: string): boolean;
```

## Indexable Types / 인덱서블 타입

Interface로 함수 타입을 선언하는 것과 유사하게 `Index`로 타입을 선언할 수 있다. 

Indexable type은 string / number로 색인화할 때 반환한 유형을 기술할 수 있다. (Index signature)

```ts
// number로 색인화(indexed) 되면 string을 반환할 것을 표현
interface StringArray {
    [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];
```

string / number 두 가지 타입을 모두 지원하는 것은 가능하지만 number로 인덱싱한 반환 타입은 string으로 인덱싱한 반환 타입의 하위 타입이어야 한다.

- number로 인덱싱할 때, string으로 변환하기 때문 (100 → "100")

```ts
interface Animal {
    name: string;
}
interface Dog extends Animal {
    breed: string;
}

interface NotOkay {
		// ERR - Numeric index type 'Animal' is not assignable to string index type 'Dog'
    [x: number]: Animal;
    [x: string]: Dog;

		// 이건 가능
    // [x: number]: Dog;
    // [x: string]: Animal;
}
```

문자열 인덱스 시그니처를 사용하는 경우, 모든 프로퍼티들이 반환 타입과 일치하도록 강제한다.

```ts
interface NumberDictionary {
    [index: string]: number;
    length: number;
    // Property 'name' of type 'string' is not assignable to string index type 'number'.
    name: string;      
}

// Resolved
interface NumberDictionary {
    [index: string]: number | string;
    length: number;
    name: string;      
}

```

인덱스 할당을 막기 위해서 `readonly` 를 사용하는 것도 가능하다.

```ts
interface ReadonlyStringArray {
    readonly [index: number]: string;
}

let myArray: ReadonlyStringArray = ["Alice", "Bob"];

// ERR - Index signature in type 'ReadonlyStringArray' only permits reading.
myArray[2] = "Mallory";
```

## Class Types / 클래스 타입

다른 언어들 처럼 클래스가 특정 규약을 지키도록 하기 위해서 Interface를 사용하는 것도 가능함.

- 단, `Interface는 Class의 public만을 기술`하기 때문에 private은 체크할 수 없다.

```ts
interface ClockInterface {
    currentTime: Date;
		setTime(d: Date): void;
}

class Clock implements ClockInterface {
    currentTime: Date = new Date();
		setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

```js
// Compiled
"use strict";
class Clock {
    constructor(h, m) {
        this.currentTime = new Date();
    }
    setTime(d) {
        this.currentTime = d;
    }
}
```

```ts
// .d.ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date): void;
}
declare class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date): void;
    constructor(h: number, m: number);
}
```

### Class's Interface에서 static과 instance 차이

클래스를 위한 인터페이스를 다룰 때, static과 instance 두가지 타입에 대해서 생각해봐야 함.

- Static 타입관련 샘플

```ts
interface ClockConstructor {
    new (hour: number, minute: number): Clock;
}

class Clock implements ClockConstructor {
    currentTime: Date;
		// Interface는 클래스의 인스턴스만 타입 체크함. -> 생성자는 static의 영역이므로 에러 발생.
		/**
			[ERR]
			Class 'Clock' incorrectly implements interface 'ClockConstructor'.
		  Type 'Clock' provides no match for the signature 
			'new (hour: number, minute: number): Clock'.
		*/
    constructor(h: number, m: number) { }
}
```

- 혼합 : 매개 변수로 생성자를 전달받아 새로운 Instance를 만들어 주는 대리 생성자 함수를 구현.

```ts
// 생성자의 Interface
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}

// Class의 Interface
interface ClockInterface {
    tick(): void;
}

// 전달된 타입의 인스턴스를 생성하는 생성자 함수를 구현
function createClock(
	ctor: ClockConstructor,
	hour: number, 
	minute: number
): ClockInterface {
		// 전달받은 생성자를 호출
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

- 클래스 표현을 이용하는 방법

```ts
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface;
}

interface ClockInterface {
  tick(): void
}

const Clock: ClockConstructor = class Clock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
      console.log("beep beep");
  }
}

const clock = new Clock(15, 30)
console.log(clock.tick) // "beep beep"
```

## Interface의 확장

클래스처럼 Interface의 멤버를 다른 Interface에 복사하는 것도 가능하다. 

이를 이용하여 재사용성이 높고 유연한 컴포넌트를 만들 수 있다.

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape {
    sideLength: number;
}

// Multi interfaces를 확장
interface Square2 extends Shape, PenStroke {
    sideLength: number;
}

let square = {} as Square;
square.color = "blue";
square.sideLength = 10;
```

## Hybrid Types / 하이브리드 타입

JS의 동적이고 유연한 특성을 처리하기 위해 몇가지 타입을 조합한 구조

- 3rd-party library를 대응하는 경우 이같은 패턴이 사용될 수 있음.

```ts
interface Counter {
		// 함수 (생성자)
    (start: number): string;
		// Property
    interval: number;
		// Public method
    reset(): void;
}

function getCounter(): Counter {
    let counter = (function (start: number) { 
			// start부터 시작해서 Interval마다 값을 증가시킴
		}) as Counter;
		
    counter.interval = 123;
    counter.reset = function () { 
			// Counter를 초기화
		};
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

## Interfaces Extending Classes / 클래스를 확장한 인터페이스

Interface가 Class를 상속할 때, Class의 멤버를 모두 상속하며 구현은 상속받지 않게 된다. 

- Private, Protected 포함

```js
class Control {
	private state: any;
}

// Interface SelectableControl을 구현하는 것은 Control을 상속하는 Class에서만 가능하다.
// SelectableControl는 Control class를 상속하기 때문에 private state 프로퍼티를 포함하기 때문.
interface SelectableControl extends Control {
    select(): void;
}

// Button은 Control을 상속하기 때문에 SelectableControl를 구현할 수 있다.
class Button extends Control implements SelectableControl {
    select() { }
}

// Control을 상속하지 않았기 때문에 private property를 직접 구현할 경우, 다음의 에러가 발생.
// ERR - Types have separate declarations of a private property 'state'.
// 구현하지 않는 경우, 다음의 에러가 발생.
// ERR - Property 'state' is missing in type 'SampleImage' but required in type 'SelectableControl'.
class SampleImage implements SelectableControl {
    // private state: any;
    select() { 
        console.log(this.state)
    }
}
```

## References

[Handbook - Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html)

[TypeScript 한글 문서](https://typescript-kr.github.io/pages/interfaces.html)
