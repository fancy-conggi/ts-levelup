# TS Study(W5) - 유틸리티 타입

## Utility type?

공통 타입(Common Type) 변환을 쉽게 도와주는 몇가지 타입들.

## Partial<T>

`T`의 프로퍼티를 Optional로 구성하는 타입.

## Readonly<T>

`T`의 프로퍼티를 Read only로 구성하는 타입.

- 생성된 타입의 프로퍼티를 재할당할 수 없음.

```tsx
interface Todo {
    title: string;
}

const todo: Readonly<Todo> = {
    title: 'Delete inactive users',
};

// Cannot assign to 'title' because it is a read-only property.
todo.title = 'Hello';
```

## Record<K, T>

 `K` 를 key로 `T` 값을 갖는 타입을 구성.

```tsx
interface PageInfo {
    title: string;
}

type Page = 'home' | 'about' | 'contact';

// Page = Keys
// PageInfo = Type of value
const x: Record<Page, PageInfo> = {
    about: { title: 'about' },
    contact: { title: 'contact' },
    home: { title: 'home' },
};
```

## Pick<T, K>

`T` 에서 `K` 의 집합을 선택해 새로운 타입을 구성.

```tsx
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

// Todo에서 title과 compleated 프로퍼티만을 Pick함.
type TodoPreview = Pick<Todo, 'title' | 'completed'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

## Omit<T, K>

`T` 에서 모든 프로퍼티를 선택한 다음 `K` 를 제거한 새로운 타입을 구성.

```tsx
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Omit<Todo, 'description'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

## Exclude<T, U>

`T`에서 `U` 에 할당할 수 있는 속성을 제외한 타입을 구성.

```tsx
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2 = Exclude<string | number | (() => void), Function>;  // string | number
```

## Extract<T, U>

`T`에서 `U` 에 할당할 수 있는 속성을 포함한 타입을 구성.

```tsx
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void
```

## NonNullable<T>

`T` 에서 null과 undefined를 제외한 타입을 구성.

```tsx
 type T0 = NonNullable<string | number | undefined>;  // string | number
type T1 = NonNullable<string[] | null | undefined>;  // string[]
```

## Parameters<T>

함수 타입 `T` 의 매개변수 타입들을 튜플 타입으로 구성.

```tsx
declare function f1(arg: { a: number, b: string }): void
type T0 = Parameters<() => string>;         // []
type T1 = Parameters<(s: string) => void>;  // [string]
type T2 = Parameters<(<T>(arg: T) => T)>;   // [unknown]
type T4 = Parameters<typeof f1>;            // [{ a: number, b: string }]
type T5 = Parameters<any>;                  // unknown[]
type T6 = Parameters<never>;                // never
type T7 = Parameters<string>;               // 오류
type T8 = Parameters<Function>;             // 오류
```

## ConstructorParameters<T>

생성자 함수 타입의 모든 매개변수 타입을 추출하여 튜플 타입으로 생성.

```tsx
type T0 = ConstructorParameters<ErrorConstructor>;     
// type T0 = [message?: string | undefined]

type T1 = ConstructorParameters<FunctionConstructor>;  
// type T1 = string[]

type T2 = ConstructorParameters<RegExpConstructor>;  
// type T2 = [pattern: string | RegExp, flags?: string | undefined]
```

## ReturnType<T>

함수 타입 `T`의 반환 타입으로 구성된 타입을 생성.

```tsx
declare function f1(): { a: number, b: string }
type T0 = ReturnType<() => string>;         // string
type T1 = ReturnType<(s: string) => void>;  // void
type T2 = ReturnType<(<T>() => T)>;         // {}
type T3 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T4 = ReturnType<typeof f1>;            // { a: number, b: string }
type T5 = ReturnType<any>;                  // any
type T6 = ReturnType<never>;                // any
type T7 = ReturnType<string>;               // 오류
type T8 = ReturnType<Function>;             // 오류
```

## InstanceType<T>

생성자 함수 타입 `T` 의 인스턴스 타입으로 구성된 타입을 생성.

```tsx
class C {
    x = 0;
    y = 0;
}

type T0 = InstanceType<typeof C>;  // C
type T1 = InstanceType<any>;       // any
type T2 = InstanceType<never>;     // any
type T3 = InstanceType<string>;    // 오류
type T4 = InstanceType<Function>;  // 오류
```

## Required<T>

`T` 타입의 프로퍼티 중, Required인 프로퍼티로 타입을 구성.

```tsx
interface Props {
    a?: number;
    b?: string;
};

const obj: Props = { a: 5 };

// Property 'b' is missing in type '{ a: number; }' 
// but required in type 'Required<Props>'.
const obj2: Required<Props> = { a: 5 };
```

## ThisParameterType<T>

함수 타입 `T` 의 매개변수 this의 타입을 추출.

- this가 없다면 `unknown` 이 됨

```tsx
function toHex(this: Number) {
  return this.toString(16);
}

function numberToString(n: ThisParameterType<typeof toHex>) {
  return toHex.apply(n);
}
```

## OmitThisParameter<T>

타입 `T`의 명시적 this 매개변수를 제거한 새로운 타입을 반환.

```tsx
function getAge(this: typeof cat) {
  return this.age;
}

// 기존 데이터
const cat = {
  age: 12 // Number
};
getAge.call(cat); // 12

// 새로운 데이터
const dog = {
  age: '13' // String
};

/**
Argument of type '{ age: string; }' is not assignable to parameter of type '{ age: number; }'.
Types of property 'age' are incompatible.
Type 'string' is not assignable to type 'number'
*/
getAge.call(dog);

const getAgeForDog: OmitThisParameter<typeof getAge> = getAge;
getAgeForDog.call(dog); // '13'
```

## ThisType<T>

타입 `T`의 명시적 this Context를 명시, 별도의 반환은 없음.

```tsx
interface IUser {
  name: string,
  getName: () => string
}

function makeNeo(methods: ThisType<IUser>) {
  return { name: 'Neo', ...methods } as IUser;
}

const neo = makeNeo({
  getName() {
    return this.name;
  }
});

neo.getName(); // Neo
```

## References
[한눈에 보는 타입스크립트(updated)](https://heropy.blog/2020/01/27/typescript/)  
[TypeScript 한글 문서](https://typescript-kr.github.io/pages/utility-types.html#omitthisparameter)
