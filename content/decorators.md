# Decorators



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



Decorator는 클래스선언, 메서드, 접근자, 프로퍼티, 매개변수에 첨부할 수 있는 특수한 종류의 선언

Decorator는 @expression 형식을 사용 - 여기서 expression은 Decorating된 선언에 대한 정보와 함께 런타임에 호출되는 함수

 데코레이터는 **함수** 라고 할 수 있습니다.

데코레이터는 말 그대로 코드 조각을 장식해주는 역할을 하며 타입스크립트에서는 그 기능을 함수로 구현할 수 있습니다.

```ts
function sealed(target) {
    // 'target' 변수와 함께 무언가를 수행합니다.
}
```



## Decorator Factories

Decorator 선언에 적용되는 방식을 원하는 대로 바꾸고 싶다면 Decorator Factory 작성

Decorator Factory는 단순히 Decorator가 런타임에 호출하 표현식을 반환하는 함수

```ts
function color(value: string) { // 데코레이터 팩토리
    return function (target) { // 데코레이터
        // 'target'과 'value' 변수를 가지고 무언가를 수행합니다.
    }
}

```



## Decorator Composition

데코레이터 합성



단일 행일 경우

```ts
@f @g x
```



여러 행일 경우

```ts
@f
@g
x
```



여러 데코레이터가 단일 선언에 적용되는 경우 수학은 합성 함수와 유사

f, g를 합성할 때  (*f*∘*g*)(*x*)의 합성 결과는 *f*(*g*(*x*))



1. 각 Decorator의 표현은 위에서 아래로 평가
2. 그런 다음 결과는 아래서 위로 함수로 호출

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



## Decorator Evaluation 

데코레이터 평가

