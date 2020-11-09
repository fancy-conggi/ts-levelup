---

---

# Decorators

## Decorator 활성화



cmd

```
tsc --target ES5 --experimentalDecorators
```



tsconfig.json

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```



## Decorators

Decorator는 클래스선언, 메서드, 접근자, 프로퍼티, 매개변수에 첨부할 수 있는 특수한 종류의 선언

Decorator는 @expression 형식을 사용 - 여기서 expression은 Decorating된 선언에 대한 정보와 함께 런타임에 호출되는 함수



데코레이터는 **함수** 라고 할 수 있습니다.

데코레이터는 말 그대로 코드 조각을 장식해주는 역할을 하며 타입스크립트에서는 그 기능을 함수로 구현할 수 있습니다.



예를 들어, 데코레이터 `@sealed`를 사용하면 다음과 같이 `sealed` 함수를 작성할 수 있음

```ts
function sealed(target) {
    // 'target' 변수와 함께 무언가를 수행합니다.
}
```



### execute time

이렇게 Decorators로 정의된 함수는 Decorators가 적용된 메소드가 실행되거나 

클래스가 new라는 키워드를 통해 인스턴스화 될 때가 아닌 런타임(컴파일아님?) 때 실행 - 즉, 매번 실행되지 않음 

- Decorators가 메소드에 적용되는 경우
- Decorators가 클래스에 적용되는 경우
- Decorators가 프로퍼티에 적용되는 경우



### Decorator to Property

```ts
class Greeter {
    @format("Hello, %s")
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}

function format(target, name, desciptor) {
  
}
```



### Decorator to Method

메소드에 적용되는 경우

우선 Decorator로 사용할 chaining이라는 함수 정의 (추 후  @chaining 형식으로 사용)

- `@`과 함께 함수가 호출되는 경우 파라미터 [defineProperty](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
  - target - 속성을 정의하고자 하는 객체
  - name - 속성의 이름
  - descriptor - 새로 정의하고자 하는 속성에 대한 설명

```ts

// 이는 Object.defineProperty() 를 통해 이를 정의하고 있기 때문
function chaining(target: any, key: string, descriptor: PropertyDescriptor): any {
	// target은 해당 메소드가 속해있는 클래스 프로토타입을 가리키며 Pet의 프로토타입에는 constructor, bark 메소드가 있는 것을 확인
  console.log(target); // {bark: f, constructor: f}
  
  // 
  console.log(key); // bark
  
  // descriptor는 defineProperty에서 정의할 수 있는 각각의 속성값
  console.log(descriptor); // {value: f, writable: true, enumerable: true, configurable: true}
}
```



위의 예제로 chaining 기능을 만들어 봅시다

```ts
// Decorator to method
function chaining(target: any, key: string, descriptor: PropertyDescriptor): any {
  const fn: Function = descriptor.value;

  // value가 데코레이터가 적용된 함수, 즉 실행 대상
  descriptor.value = function(...args: any[]) {
    fn.apply(target, args);
    return target;
  }
}
```



Pet의 메소드에 chaining기능을 Decorating 합니다.

```ts
class Pet {
  @chaining
  bark() {
  }
}

const pet = new Pet();
```



Pet 클래스에서 bark라는 메소드는 Pet.prototype.bark

<u>class syntax 내부에서 위 코드에서 bark라는 메소드가 Pet의 prototype의 프로퍼티로 추가되기 전에</u>

<u>decorator 함수가 실행되어 본래 bark 메소드에서 정의된 것에 추가적인 '장식'을 더해 prototype에 추가되도록 함!</u>

> 만약 compile target이 ES5보다 낮다면 `PropertyDescriptor`값이 `undefined`



```ts
pet.bark().bark();
```

위의 Decorators의 효과로 `return this;`를 하지 않아도 chaining 기능을 사용하여 메소드를 호출할 수 있음



### Decorators to Class

ClassDecorator는 클래스 선언 전에 선언

ClassDecorator는 클래스 생성자에 적용되고 클래스 정의를 관찰, 수정, 또는 대체하는데 사용

```ts
function component(target, name, descriptor) {
  console.log(target); // ...
  console.log(name); // undefined
  console.log(descriptor); //undefined
}
```



Class에 적용되는 Decorator함수에 전달되는 인자는 `constructor` 하나

```ts
function classDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
  return class extends constructor {
    newProperty = "new property";
  }
}
```



클래스에 적용되는 Decorator 함수내에서 새로운 생성자 함수를 반환하면 원래 prototype을 유지해야함

런타임에 Decorator를 적용하는 로직은 이를 수행하지 않음

위 코드에서 기존의 prototype을 유지하기 위해서 적용되는 클래스의 `consturctor`를 `extends` 함



```ts
@classDecorator
class Pet {
  constructor(name: string) {
    this.name = name;
  }
}

const pet = new Pet("async");
console.log(pet.newProperty); // new Property
```

`classDecorator`가 적용된 Pet 클래스의 인스턴스에 `newProperty`가 존재하지 않지만 

classDecorator에서 해당 클래스의 constructor를 재정의 했기에 newProperty에 접근 가능 



### Decorator with Parameter

descriptor의 `enumerable` 속성을 변경하는 데코레이터

다음과 같이하면, enumerableToFalse이 적용된 메소드의 enumerable 속성은 false

```ts
function enumerableToFalse(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  descriptor.enumerable = false;
};
```



enumerableToFalse 함수를 한 번 깜싸서 반환하는 함수

```ts
function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value;
  };
}
```



false라는 인자를 받는 Decorator를 정의

인자에는 함수도 들어갈 수 있고, Decorator도 들어갈 수 있음

```ts
class Pet {
  @enumerable(false)
  bark() {
	  //...
  }
}
```





### Decorator to Accessor

접근자 데코레이터

```ts
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    };
}
```



```ts
class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}
```



## Decorator Factories

Decorator 선언에 적용되는 방식을 원하는 대로 바꾸고 싶다면 Decorator Factory 작성

Decorator Factory는 단순히 <u>Decorator가 런타임에 호출할 표현식을 반환하는 함수</u>

```ts
function color(value: string) { // 데코레이터 팩토리
    return function (target) { // 데코레이터
        // 'target'과 'value' 변수를 가지고 무언가를 수행합니다.
    }
}

```



## Decorator Composition

데코레이터 합성



**단일 행일 경우**

```ts
@f @g x
```



**여러 행일 경우**

```ts
@f
@g
x
```



> 여러 데코레이터가 단일 선언에 적용되는 경우 수학은 합성 함수와 유사
>
> f, g를 합성할 때  (*f*∘*g*)(*x*)의 합성 결과는 *f*(*g*(*x*))



ts에서 단일 선언에서 여러 Decorators를 사용할 때 다음 단계가 수행

1.  각 Decorator의 표현은 위에서 아래로 평가
2. 그런 다음 결과는 아래서 위로 함수로 호출



데코레이터 팩토리를 사용하는 겨우

다음 예제를 통해 이 수행 순서를 확인 가능

```ts
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```



결과

```
f(): evaluated
g(): evaluated
g(): called
f(): called
```



컴파일 결과

```js
"use strict";
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
var __metadata = (this && this.__metadata) || function (k, v) {
    if (typeof Reflect === "object" && typeof Reflect.metadata === "function") return Reflect.metadata(k, v);
};
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey, descriptor) {
        console.log("f(): called");
    };
}
function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey, descriptor) {
        console.log("g(): called");
    };
}
class C {
    method() { }
}
__decorate([
    f(),
    g(),
    __metadata("design:type", Function),
    __metadata("design:paramtypes", []),
    __metadata("design:returntype", void 0)
], C.prototype, "method", null);
```



## Decorator Evaluation 

데코레이터 평가

클래스에서 다양한 선언에 데코레이터를 적용하는 방법은 다음과 같이 잘 정의되어 있습니다.

1. *메서드*, *접근자* 또는 *프로퍼티 데코레이터*가 다음에 오는 *매개 변수 데코레이터*는 각 인스턴스 멤버에 적용됩니다.
2. *메서드*, *접근자* 또는 *프로퍼티 데코레이터*가 다음에 오는 *매개 변수 데코레이터*는 각 정적 멤버에 적용됩니다.
3. *매개 변수 데코레이터*는 생성자에 적용됩니다.
4. *클래스 데코레이터*는 클래스에 적용됩니다.



https://jaeyeophan.github.io/2018/01/09/TS-6-Decorator/