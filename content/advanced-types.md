# 	고오급타입

## Intersection Types & Union Types

### Intersection Types (교차타입 - &)

여러 타입을 하나로 결합. 즉, 여러 인터페이 스에 있는 모든 property를 갖게 됩니다.

MixIn 예제 (기존 객체-지향 틀과는 맞지 않는 믹스인이나 다른 컨셉들에서 교차 타입이 사용되는 것을 볼 수 있습니다)

```typescript
function extend<First, Second>(first: First, second: Second): First & Second {
    const result: Partial<First & Second> = {};
    for (const prop in first) {
        if (first.hasOwnProperty(prop)) {
            (result as First)[prop] = first[prop];
        }
    }
    for (const prop in second) {
        if (second.hasOwnProperty(prop)) {
            (result as Second)[prop] = second[prop];
        }
    }
    return result as First & Second;
}

class Person {
    constructor(public name: string) { }
}

interface Loggable {
    log(name: string): void;
}

class ConsoleLogger implements Loggable {
    log(name) {
        console.log(`Hello, I'm ${name}.`);
    }
}

const jim = extend(new Person('Jim'), ConsoleLogger.prototype);
jim.log(jim.name);
```



### Union Types (유니언타입)

교차타입과 밀접하게 관련있지만 매우 다르게 사용

```typescript
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // "    Hello world"를 반환합니다.
function padLeft(value: string, padding: string | number) {
    // ...
}
```



Union 타입이 반환값 일 때 > 교집합(두 인터페이스에 모두 포함되어 있는) 속성만 접근이 가능

```typescript
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // 성공
pet.swim();    // 오류
```



## Type Guards and Differentiating Types

Union 타입은 값의 타입이 겹쳐질 수 있는 상황을 모델링하는데 유용

Fish가 있는지 구체적으로 알고 싶을 때는 어떻게 할까요 ... ? `멤버의 존재를 검사`

```typescript
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // 성공

// 이렇게 각 프로퍼티들에 접근하는 것은 오류를 발생시킵니다
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}

// `Type Assertion`을 사용해서 검사해야만 한다.
if ((pet as Fish).swim) {
    (pet as Fish).swim();
} else if ((pet as Bird).fly) {
    (pet as Bird).fly();
}
```



### UserDefined Type Gurads (사용자정의 타입가드)

Typescript는 타입 가드라는 것이 있는데, 

타입 가드는 스코프 안에서의 타입을 보장하는 런타임 검사를 수행한다는 표현식



#### Using type predicates (타입 서술어 사용하기)

타입 가드를 정의하기 위해서 반환 타입이 타입 서울어인 함수를 정의 

`pet is Fish` 는 이 예제에서 타입 서술어 (`parameterName is Type` - parameterName은 현재 함수의 매개변수 이름)

isFish가 변수와 함께 호출될 때마다, Typescript는 기존 타입과 호환된다면 그 변수를 특정 타입으로 제한

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}

// 이제 'swim'과 'fly'에 대한 모든 호출은 허용됩니다
if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```



> typescript는 반환값으로 타입을 제한하고 있다.

```typescript
interface Bird {
    fly : any
    layEggs: any
}

interface Fish {
    swim: any
    fly: any
    layEggs: any
}

// 파충류 - 걷는다, 수영은 할 수도 못할 수도
interface Reptile { 
    swim?: any
    walk: any
    layEggs: any
}

// 만약 여기에 반환값으로 Reptile이 없다면 ? -> 아래 isFish 함수의 나머지 반환값이 Bird로 한정 (에러가 나지 않는다)
// function getSmallPet(): Fish | Bird {
function getSmallPet(): Fish | Bird {
    let fish : Fish = {
        swim : 1,
        layEggs : 2,
    }
    return fish
}

function isFish(pet: Fish | Bird | Reptile): pet is Fish {
    return (pet as Fish).swim !== undefined;
}

let pet = getSmallPet(); // Fish | Brid
if (isFish(pet)) { // Fish | Bird | Reptile
	// Fish
  pet.swim();
} else {
  // Bird
  pet.fly();
}
```



typescript가 pet이 `if문` 안에서 Fish라는 것을 알고 있고

`else문`에서는 Fish가 없다는 것을 알고 있음(자동적으로 Bird라는 것을 인식)

<img src="../../gatsby-gitbook-starter/public/static/images/image-20201102121931996.png" alt="image-20201102121931996" style="zoom:50%;" />



#### in

in 연산자는 타입을 좁히는 표현으로 작용

`n in x`, n은 문자열 리터럴 혹은 문자열 리터럴 타입 x는 유니언 타입

- true 분기에서는 선택적 혹은 필수 프로퍼티 n을 가지는 타입
- false 분기에서는 선택적 혹은 프로퍼티 n을 가지는 않는 타입

```typescript
function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly();
}
```



> Optional Property일 경우 어느 분기에 속할지 보장하지 못한다

```typescript
interface Bird {
    fly : any
    layEggs: any
}

interface Fish {
    swim?: any
    layEggs: any
}

function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly(); // 오류
}
```

<img src="../../gatsby-gitbook-starter/public/static/images/image-20201102172201461.png" alt="image-20201102172201461" style="zoom:50%;" />



#### typeof (primitive)

 TypeScript는 `typeof`를 타입 가드로 인식하기 때문에 `typeof x === "number"`를 함수로 추상할 필요가 없음

typeof 타입가드에서 표현식으로 인정하는 것은 다음 뿐, `"number"`, `"string"`, `"boolean"` 그리고 `"symbol"`

(다른 문자열과도 비교는 가능하지만 타입가드로 인지되지는 않음)

```typescript
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
       return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```



#### instanceof (object)

*`instanceof` 타입 가드* 는 생성자 함수를 사용하여 타입을 좁히는 방법

instanceof의 오른쪽에는 생성자 함수여야 함

```ts
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// 타입은 'SpaceRepeatingPadder | StringPadder' 입니다
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 타입은 'SpaceRepeatingPadder'으로 좁혀집니다
}
if (padder instanceof StringPadder) {
    padder; // 타입은 'StringPadder'으로 좁혀집니다
}
```



## --strictNullChecks



### Nullable

Typescript에는 특수타임 `null`과 `undefined`가 있음

null은 사용하지 않는게 좋으며, `--strictNullChecks` 옵션으로 ts내에 null을 허용하지 않을 수도 있음

```ts
let s = "foo";
s = null; // 오류, 'null'은 'string'에 할당할 수 없습니다
let sn: string | null = "bar"; // 명시적 허용은 가능
sn = null; // 성공
sn = undefined; // 오류, 'undefined'는 'string | null'에 할당할 수 없습니다. ('string | undifined' 사용)
```

> `string | null`은 `string | undefined`와 `string | undefined | null`과는 다른 타입



### Optional Parameters and Property

`--strictNullChecks`를 적용하면, 선택적 매개변수가 `| undefined`를 자동으로 추가

(--strictNullChecks 없다면 당연히 null 사용가능)

```ts
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // 오류, 'null'은 'number | undefined'에 할당할 수 없습니다
```

<img src="../../gatsby-gitbook-starter/public/static/images/image-20201102175619320.png" alt="image-20201102175619320" style="zoom:50%;" />



```ts
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // 오류, 'undefined'는 'number'에 할당할 수 없습니다
c.b = 13;
c.b = undefined; // 성공
c.b = null; // 오류, 'null'은 'number | undefined'에 할당할 수 없습니다.
```



### TypeGuard & TypeAssertion

널러블 타입이 유니언으로 구현되기 때문에, `null`을 제거하기 위해 타입 가드를 사용할 필요가 있음

```ts 
// 방법1
function f(sn: string | null): string {
    if (sn == null) {
        return "default";
    }
    else {
        return sn;
    }
}

// 방법2
function f(sn: string | null): string {
    return sn || "default";
}
```



`identifier!`는 `null`과 `undefined`를 `identifier`의 타입에서 제거

```ts
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // 오류, 'name'은 아마도 null 일수도
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // 성공
  }
  name = name || "Bob";
  return postfix("great");
}
```



## Type Aliases ( 타입 별칭 )

타입별칭은 타입의 새로운 이름을 만듬

인터페이스와 유사하지만 원시 값, 유니언, 튜플 그리고 손으로 작성해야 하는 다른 타입의 이름을 지을 수 있음

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === "string") {
        return n;
    }
    else {
        return n();
    }
}
```



인터페이스처럼, 타입 별칭은 제네릭이 될 수 있습니다 - 타입 매개변수를 추가하고 별칭 선언의 오른쪽에 사용하면 됩니다

```ts
type Container<T> = { value: T };
```



프로퍼티 안에서 자기 자신을 참조하는 타입 별칭을 가질 수 있습니다:

```ts
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}

// 하지만, 타입 별칭을 선언의 오른쪽 이외에 사용하는 것은 불가능합니다.
type Yikes = Array<Yikes>; // 오류
```



교차 타입과 같이 사용하면, 아주 놀라운 타입을 만들 수 있습니다.

타입별칭을 통해 LinkedList 만드는 예제

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```



### Interface vs TypeAliases

- 한 가지 차이점은 인터페이스는 어디에서나 사용할 수 있는 새로운 이름을 만들 수 있음
  타입 별칭은 새로운 이름을 만들지 못함 — 예를 들어, 오류 메시지는 별칭 이름을 사용하지 않음
- 타입 별칭은 extend 하거나 implement 할 수 없음
  ( 2.7 버전부터, 타입 별칭은 교차 타입을 생성함으로써 extend 할 수 있음 - type Cat = Animal & { purrs: true })

```ts
type Alias = { num: number }
interface Interface {
    num: number;
}

// 아래의 코드에서, 에디터에서 
// interfaced에 마우스를 올리면 Interface를 반환한다고 보여주지만 
// aliased는 객체 리터럴 타입을 반환한다고 보여줌
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```



> 가능하면 항상 타입 별칭보다 인터페이스를 사용해야 합니다.
>
> 반면에, 만약 인터페이스로 어떤 형태를 표현할 수 없고 유니언이나 튜플 타입을 사용해야 한다면, 일반적으로 타입 별칭을 사용



## Literal

### 문자열 리터럴

생략

문자열 리터럴의 오버로드

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... 더 많은 오버로드 ...
function createElement(tagName: string): Element {
    // ... 이곳에 코드를 ...
}
```





### 숫자형 리터럴

생략



## Enum Member Type

생략



## Discriminated Unions(판별유니언)

각 인터페이스는 다른 <u>문자열 리터럴</u> 타입의 `kind` 프로퍼티를 가짐

```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}

type Shape = Square | Rectangle | Circle;
```



Discriminated Unions(판별유니언) 사용하기

```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```



### Exhaustiveness checking (엄격한 검사)

```ts
type Shape = Square | Rectangle | Circle | Triangle;
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
    // 여기서 오류 발생 - "triangle"의 케이스를 처리하지 않음
}
```



컴파일러가 완전함을 검사하기 위해 사용하는 `never` 타입을 사용하는 것입니다.

-> never 타입을 사용하고 오류로 유도한다. CompileError는 넘어갈 수 있음

```ts
function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
        default: return assertNever(s); // 빠진 케이스가 있다면 여기서 오류 발생
    }
}
```



## Polymorphic `this` types (다형성 this 타입)

다형성 this 타입은 포함하는 클래스나 인터페이스의 *하위 타입*을 나타냄





## 인덱스 타입



## 매핑 타입