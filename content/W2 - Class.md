## Classes

- 클래스를 사용하는 예제

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

- 상속을 사용하는 예제

```ts
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

## public, private and protected modifiers

### public modifier

- Default modifier

```ts
class Animal {
    // public name: string;
		name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

### private modifier

- 멤버를 포함하는 클래스 외부에서 이 멤버에 접근할 수 없도록 하는 방법.

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

const cat = new Animal("Cat")
// ERR - Property 'name' is private and only accessible within class 'Animal'.
console.log(cat.name)
```

```ts
// .d.ts
declare class Animal {
    private name;
    constructor(theName: string);
}
declare const cat: Animal;
```

```js
"use strict";
class Animal {
    constructor(theName) { this.name = theName; }
}
const cat = new Animal("Cat");

// Private으로 지정했지만 실제로는 그렇지 않다는 걸 알 수 있음.
console.log(cat.name);
```

### TS의 Pirvate, Protected

TS는 구조적인 타입 시스템이기 때문에 두개의 타입을 비교할 때, 모든 멤버의 타입이 호환된다고 하면 그 타입 자체가 호환 가능하다고 판단한다.

```ts
type TEST1 = {
    name: string
    age: number
}

type TEST2 = {
    name: string
    age: number
}

const testA: TEST1 = {name: 'K', age: 123}
const testB: TEST2 = testA
```

하지만 `private` 이나 `protected` 가 있는 타입을 비교하는 경우, 다르게 처리된다.

- 호환된다고 판단하는 두 개의 타입 중, 한쪽에 Private / Protected가 있다면 다른 한쪽에도 동일한 선언의 Private/ Protected 멤버가 있어야 함.

    ```ts
    class Animal {
        private name: string;
        constructor(theName: string) { this.name = theName; }
    }

    class Rhino extends Animal {
        constructor() { super("Rhino"); }
    }

    class Employee {
        private name: string;
        constructor(theName: string) { this.name = theName; }
    }

    let animal = new Animal("Goat");
    let rhino = new Rhino();
    let employee = new Employee("Bob");

    // Rhino가 상속하고 있는 private property는 Animal의 그것과 같음.
    animal = rhino;

    // ERR - Types have separate declarations of a private property 'name'.
    // Animal과 Employee의 구조는 같지만 private property가 다름.
    animal = employee;
    ```

## Private Fields / 비공개 필드

- TS 3.8 부터는 `#`을 사용하여 비공개 필드를 처리할 수 있다.

```ts
class Animal {
    #name: string;
		#age: number;
		constructor(theName: string, age?: number) { 
        this.#name = theName; 
        this.#age = age || 10
    }

    public saySomething () {
        console.log(this.#name)
    }
}

const cat = new Animal("Cat")
cat.saySomething()

// ERR - Property '#name' is not accessible outside class 'Animal' 
// because it has a private identifier.
cat.#name
```

```ts
// .d.ts

declare class Animal {
		// Private은 property의 수와 관계 없는 듯
    #private;
    constructor(theName: string);
    saySomething(): void;
}
declare const cat: Animal;
```

```js
"use strict";

// Class's Private filed setter
var __classPrivateFieldSet = (
	this 
	&& this.__classPrivateFieldSet) 
	|| function (receiver, privateMap, value) {
    if (!privateMap.has(receiver)) {
        throw new TypeError("attempted to set private field on non-instance");
    }
    privateMap.set(receiver, value);
    return value;
};

// Class's private field getter
var __classPrivateFieldGet = (
	this 
	&& this.__classPrivateFieldGet) 
	|| function (receiver, privateMap) {
    if (!privateMap.has(receiver)) {
        throw new TypeError("attempted to get private field on non-instance");
    }
    return privateMap.get(receiver);
};

var _name, _age;
class Animal {
    constructor(theName, age) {
				// Set weakmap
        _name.set(this, void 0);
        _age.set(this, void 0);
				// Set privateMap's recevier
        __classPrivateFieldSet(this, _name, theName);
        __classPrivateFieldSet(this, _age, age || 10);
    }
    saySomething() {
        console.log(__classPrivateFieldGet(this, _name));
        console.log(__classPrivateFieldGet(this, _age));
    }
}
_name = new WeakMap(), 
_age = new WeakMap();
```

### `#` VS Private

> That means that when using the private keyword, privacy is only enforced at compile-time/design-time, and for JavaScript consumers, it’s entirely intent-based.

기존의 Private과 `#`(Private Fields)의 차이는 컴파일/디자인 시에만 체크하는지 여부에 달려있다.

따라서 정말 Private하게 사용한다면 `#`를 사용하는 것이 올바르고,

Application을 디자인하는 과정에서만 사용한다면 `private` modifier를 사용하는 것이 맞다.

## Readonly modifier / 읽기전용 지정자

프로퍼티를 읽기 전용으로 만들 수 있으며 `초기화는 선언/생성자`에서 이루어져야 한다.

```ts
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
```

```ts
// .d.ts
declare class Octopus {
    readonly name: string;
    readonly numberOfLegs: number;
    constructor(theName: string);
}
```

```js
"use strict";
class Octopus {
    constructor(theName) {
        this.numberOfLegs = 8;
        this.name = theName;
    }
}
```

## Parameter properties

위의 예제에서 `name, numberOfLegs` 를 읽기전용으로 지정하고 생성자에서 `name` 을 초기화 해주었다.

이럴 경우, Parameter property를 사용하면 한 곳에서 멤버를 만들고 초기화할 수 있다.

- private, protected, readonly도 동일하게 적용받음.

```ts
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```

```ts
declare class Octopus {
    readonly name: string;
    readonly numberOfLegs: number;
    constructor(name: string);
}
```

```js
"use strict";
class Octopus {
    constructor(name) {
        this.name = name;
        this.numberOfLegs = 8;
    }
}
```

## Accessors / 접근자

TS에서는 객체의 멤버에 대한 접근을 위해 Getter / Setter 를 지원한다.

이때 몇 가지 주의해야할 사항이 있다.

- ECMAScript 3으로의 하향 조정은 지원되지 않음.
- `get`은 있지만 `set` 이 없는 Accessor는 자동으로 `readonly`로 추론된다.
    - 코드로 확인한 결과, `.d.ts` 에서 readonly가 붙지는 않음.
- Accessors는 `.d.ts` 를 만드는 추론 과정에 많은 도움을 준다.

## Static properties

인스턴스의 멤버가 아닌 클래스 자체의 static member를 지정하는 방법.

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

```ts
// .d.ts
declare class Grid {
    scale: number;
    static origin: {
        x: number;
        y: number;
    };
    calculateDistanceFromOrigin(point: {
        x: number;
        y: number;
    }): number;
    constructor(scale: number);
}
declare let grid1: Grid;
declare let grid2: Grid;
```

```js
"use strict";
class Grid {
    constructor(scale) {
        this.scale = scale;
    }
    calculateDistanceFromOrigin(point) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
}
Grid.origin = { x: 0, y: 0 };
let grid1 = new Grid(1.0); // 1x scale
let grid2 = new Grid(5.0); // 5x scale
console.log(grid1.calculateDistanceFromOrigin({ x: 10, y: 10 }));
console.log(grid2.calculateDistanceFromOrigin({ x: 10, y: 10 }));
```

## Abstract classes / 추상 클래스

다른 클래스들이 파생될 수 있는 기초 클래스.

- 직접 인스턴스화 할 수 없고 인터페이스와 다르게 멤버에 대한 구현 세부 정보를 포함할 수 있다.
- 추상 메서드는 인터페이스 메스드와 비슷한 문법을 공유하지만 `abstract` 키워드를 포함해야 하며, Optional을 사용할 수 있다.

```ts
abstract class Department {
	constructor(public name: string) {}

	printName(): void {
		console.log("Department name: " + this.name);
	}

	// 반드시 파생된 클래스에서 구현되어야 함.
	abstract printMeeting(): void; 
}

class AccountingDepartment extends Department {
	constructor() {
		// 파생된 클래스의 생성자는 반드시 super()를 호출해야 합니다.
		super("Accounting and Auditing"); 
	}

	printMeeting(): void {
		console.log("The Accounting Department meets each Monday at 10am.");
	}
	
	generateReports(): void {
		console.log("Generating accounting reports...");
	}
}

let department: Department;

// ERR - Cannot create an instance of an abstract class.
department = new Department();

department = new AccountingDepartment();
department.printName();
department.printMeeting();

// ERR - Property 'generateReports' does not exist on type 'Department'.
department.generateReports();
```

## Advanced Techiniques / 고급 기법

### Constructor functions / 생성자 함수

클래스의 선언은 `클래스의 인스턴스를 나타내는 타입` + `생성자 함수` 두가지를 생성.

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

// 인스턴스의 타입
let greeter: Greeter;

// 생성자 호출
greeter = new Greeter("world");
```

위의 예제를 JS로 변경하여 생성자 함수를 보도록 한다.

```js
// ES6 이전의 코드
let Greeter = (function () {
		// 생성자 함수
    function Greeter(message) {
        this.greeting = message;
    }
    Greeter.prototype.greet = function () {
        return "Hello, " + this.greeting;
    };
    return Greeter;
})();

// ES6
"use strict";
class Greeter {
    constructor(message) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

### Using a class as an interface / 클래스를 인터페이스처럼 사용

클래스를 인터페이스를 사용할 수 있는 동일한 위치에서 사용할 수 있다.

- 클래스의 선언은 `클래스의 인스턴스를 나타내는 타입` + `생성자 함수` 두가지를 생성하기 때문.

```ts
class Point {
    x: number;
    y: number;
}

// Class as Interface
interface Point3d extends Point {
    z: number;
}

let point3d: Point3d = {x: 1, y: 2, z: 3};
```

## References

[Announcing TypeScript 3.8 Beta | TypeScript](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#ecmascript-private-fields)

[TypeScript 한글 문서](https://typescript-kr.github.io/pages/classes.html)

[Handbook - Classes](https://www.typescriptlang.org/docs/handbook/classes.html)
