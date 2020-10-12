# Function

### 기명함수, 익명함수

```typescript
// 기명 함수
fucntion add(x, y) {
  return x + y;
}

// 익명 함수
let myAdd = function(x, y) { return x + y };
```



### 캡처

함수는 외부의 변수를 참조할 수 있으며 이런 경우 해당 변수를 `캡처`라고 함

이것이 어떻게 동작하는지를 이해하는 것은 본문의 주제에서 벗어남

```typescript
let z = 100;

function addToZ(x, y) {
  return x + y + z;
}
```



## 함수 타입 (Function Types)

### 함수의 타이핑

TypeScript는 각 파라미터와 반환타입을 정해줄 수 있습니다

[CASE2] TypeScript는 반환 문을 보고 반환 타입을 파악할 수 있어 반환 타입은 생략할 수 있음

```typescript
// [CASE1]
function add (x: number, y: number): number {
    return x + y;
}

let myAdd = function (x: number, y: number): number { return x + y };

// [CASE2]
function add (x: number, y: number){
    return x + y;
}
```



### 함수 타입 작성하기

함수도 타입을 가지고 있으며, 함수의 타입은 매개변수타입과 반환타입을 가지고 있음 (`(x: number, y: number) => number`)

캡처된 변수는 타입에 반영되지 않음, 캡처된 변수는 함수의 숨겨진 상태의 일부이고 API를 구성하지 않음 (순수함수를 사용해야하는 이유)

```typescript
let myAdd: (x: number, y: number) => number = function (x: number, y: number): number { return x + y; };
```



### 타입 추론 (Inferring the types)

TypeScript의 컴파일러가 방정식의 한쪽에만 타입이 있어도 타입을 알아낼 수 있다는 것

```typescript
// myAdd는 전체 함수 타입을 가집니다
let myAdd = function (x: number, y: number): number { return  x + y; };

// 매개변수 x 와 y는 number 타입을 가집니다
let myAdd: (baseValue: number, increment: number) => number = function (x, y) { return x + y; };
```



### 선택적 매개변수와 기본 매개변수 (Optional and Default Parameter)

함수가 호출될 때, 컴파일러는 각 매개변수에 대해 사용자가 값을 제공했는지를 검사하며 
또한 컴파일러는 매개변수들이 함수로 전달될 유일한 매개변수라고 가정

죽, 함수에 주어진 인자의 수는 함수가 기대하는 매개변수의 수와 일치해야 함

```typescript 
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 오류, 너무 적은 매개변수
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 정확함
```



#### 선택적 매개변수

<u>JS에서는 모든 매개변수가 선택적이며, 사용자는 적합하다 생각하면 그대로 둘 수 있습니다. (TS와 차이가 있음)</u>

하여 그 값은 `undefined`가 됩니다.

typescript에서도 매개변수 이름 뒤에 `?`를 붙임으로써 해결할 수 있음

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");                  // 지금은 바르게 동작
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 정확함
```



#### 기본 매개변수

[CASE1] lastName 매개변수를 Smith라고 초기화

기본 매개변수는 선택적으로(Optional) 처리되며 함수를 호출할 때 생략 가능

[CASE2]에서 보면 알 수 있듯이 기본-초기화 매개변수는 필수 매개변수 뒤에 오는 것이 강요되지 않음

```typescript 
// CASE1
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 올바르게 동작, "Bob Smith" 반환
let result2 = buildName("Bob", undefined);       // 여전히 동작, 역시 "Bob Smith" 반환
let result3 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result4 = buildName("Bob", "Adams");         // 정확함

// CASE2
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // 오류, 너무 적은 매개변수
let result2 = buildName("Bob", "Adams", "Sr.");  // 오류, 너무 많은 매개변수
let result3 = buildName("Bob", "Adams");         // 성공, "Bob Adams" 반환
let result4 = buildName(undefined, "Adams");     // 성공, "Will Adams" 반환
```



#### 선택적 매개변수와 기본 매개변수의 타입

기본-초기화 매개변수는 선택적으로 처리되어 선택적 매개변수와 마찬가지로 해당 함수를 호출할 때 생략이 가능

그렇기에 두 경우는 `(firstName: string, lastName?: string) => string` 타입을 공유함



### 나머지 매개변수(Rest Parameters)

[JavaScript의 arguments](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/arguments)

[JavaScript의 Rest Parameters](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/rest_parameters)



필수, 선택적, 기본 매개변수는 한 번에 하나의 매개변수만을 가짐

하지만 JavaScript에서는 모든 함수 내부에 위치한 `arguments`라는 변수를 사용해 직접 인자를 가지고 작업할 수 있지만,

TypeScript에서는 이 인자들을 하나의 변수로 모을 수 있음



> TypeScript + JavaScript

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

// employeeName 은 "Joseph Samuel Lucas MacKinzie" 가 될것입니다.
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

```javascript
"use strict";
function buildName(firstName, ...restOfName) {
    return firstName + " " + restOfName.join(" ");
}
// employeeName 은 "Joseph Samuel Lucas MacKinzie" 가 될것입니다.
let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```



나머지 매개변수는 선택적 매개변수들의 수를 무한으로 취급

<u>나머지 매개변수로 인자들을 넘겨줄 때 원하는 만큼 넘겨줄 수도 또는 아무것도 넘기지 않을 수도 있음</u>



**나머지 매개변수의 함수 타입**

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
    return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```



## this

[JavaScript의 this](https://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)

<u>JavaScript에서 this는 함수가 호출될 때 정해지는 변수입니다</u>



### this & arrow function

하지만 아래 예제는 alert가 아닌 error를 발생시킵니다.

`createCardPicker`에 의해 생성된 함수에서 사용 중인 `this`가 `deck` 객체가 아닌 `window`에 설정되었기 때문입니다

```js
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

>  최상위 레벨에서의 비-메서드 문법의 호출은 `this`를 `window`로 합니다. 
>
> (Note: strict mode에서는 `this`가 `window` 대신 `undefined` 가 됩니다. )



 문제는 나중에 사용할 함수를 반환하기 전에 바인딩을 알맞게 하는 것으로 해결할 수 있습니다.

이 방법대로라면 나중에 사용하는 방법에 상관없이 원본 `deck` 객체를 계속해서 볼 수 있습니다. 

```typescript
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: 아랫줄은 화살표 함수로써, 'this'(any)를 이곳에서 캡처할 수 있도록 합니다
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

>  `--noImplicitThis` 플래그를 컴파일러에 전달하는 실수를 하게 된다면 TypeScript는 경고를 표시할 것입니다.
>
> `this.suits[pickedSuit]` 의 `this`는 `any` 타입인 것을 짚고 넘어가겠습니다.



### this & parameter

불행히도 `this.suits[pickedSuit]`의 타입은 여전히 `any`입니다. `this`가 <u>객체 리터럴 내부의 함수에서 왔기 때문입니다.</u> 



이것을 고치기 위해 명시적으로 `this` 매개변수를 줄 수 있습니다.

 `this` 매개변수는 함수의 매개변수 목록에서 가장 먼저 나오는 가짜 매개변수입니다.

```ts
function f(this: void) {
    // 독립형 함수에서 `this`를 사용할 수 없는 것을 확인함
}
```



이제 TypeScript는 `createCardPicker`가 `Deck` 객체에서 호출된다는 것을 알게 됐습니다

이것은 `this`가 `any` 타입이 아니라 `Deck` 타입이며 

따라서 `--noImplicitThis` 플래그가 어떤 오류도 일으키지 않는다는 것을 의미합니다.

```ts
interface Card {
    suit: string;
    card: number;
}

interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}

let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: 아래 함수는 이제 callee가 반드시 Deck 타입이어야 함을 명시적으로 지정합니다.
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```



### this & callback 

먼저 라이브러리 작성자는 콜백 타입을 `this`로 표시를 해주어야 합니다.

```ts
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```



`this: void`는 `addClickListener`가 `onclick`이 <u>`this`타입을 요구하지 않는 함수가 될 것으로 예상하는 것을 의미합니다.</u> 

두 번째로, 호출 코드를 `this`로 표시합니다.

```ts
class Handler {
    info: string;
    // 이런, `this`가 여기서 쓰이는군요. 이 콜백을 쓰면 런타임에서 충돌을 일으키겠군요
    onClickBad(this: Handler, e: Event) {
        this.info = e.message;
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // 오류!
```



`this`로 표시를 한 상태에서 `onClickBad`가 반드시 `Handler`의 인스턴스로써 호출되어야 함을 명시해 주어야 합니다. 

그러면 TypeScript는 `addClickListener`가 `this: void`를 갖는 함수를 필요로 한다는 것을 감지합니다. 

오류를 고치기 위해 `this`의 타입을 바꿔줍니다:

```ts
class Handler {
    info: string;
    onClickGood (this: void, e: Event) {
        // void 타입이기 때문에 this는 이곳에서 쓸 수 없습니다!
        console.log('clicked!');
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickGood);
```



`onClickGood`이 `this` 타입을 `void`로 지정하고 있기 때문에 `addClickListener`로 넘겨지는데 적합합니다.

 물론, 이것이 `this.info`를 쓸 수 없는 것을 의미하기도 합니다. 



만약 당신이 `this.info`까지 원한다면 화살표 함수를 사용해야 할 겁니다:

```ts
class Handler {
    info: string;
    onClickGood = (e: Event) => { this.info = e.message }
}
```



이러한 작업은 화살표 함수가 외부의 `this`를 사용하기 때문에 가능하므로 

`this: void`일 것으로 기대하는 무언가라면 전달에 문제가 없습니다.



 `Handler` 타입 객체마다 하나의 화살표 함수가 작성된다는 점이 단점입니다. 

반면, 메서드들은 하나만 작성되어 `Handler`의 프로토타입에 붙습니다.  그들은 `Handler` 타입의 모든 객체 간에 공유됩니다.



## 오버로드 (Overloads)

타입시스템에서 이 것을 어떻게 구현할 것인가 ?

- method의 이름은 같지만 매개변수의 개수는 동일하게 type은 다르게 정의하여 사용하는 방법
- 다른 언어의 Overloading 개념은 method명만 같으면 되지만 
  typescript는 Overloading을 사용하기 위해서는 함수명과 <u>매개변수의 개수가 같아야 합니다.</u>

```ts
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x : any): any {
	// do something
}
```



오버로드 목록에서 첫 번째 오버로드를 진행하면서 제공된 매개변수를 사용하여 함수를 호출하려고 시도

만약 일치하게 된다면 해당 오버로드를 알맞은 오버로드로 선택하여 작업을 수행

이러한 이유로 가장 구체적인 것부터 오버로드 리스팅을 하는 것이 일반적입니다.

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // 인자가 배열 또는 객체인지 확인
    // 만약 그렇다면, deck이 주어지고 card를 선택합니다.
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // 그렇지 않다면 그냥 card를 선택합니다.
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```