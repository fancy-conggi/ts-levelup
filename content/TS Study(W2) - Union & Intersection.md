## Union & Intersection

이미 존재하는 타입을 조합하여 필요한 타입을 만들어내는 방법.

## Union Types / 유니언 타입

Union type은 여러 타입 중, 하나의 타입이 될 수 있음을 의미하며  `|` 로 각 타입을 구별.

- any를 이용하여 다양한 타입을 처리하는 예제.

```ts
// 매개변수 padding의 타입이 number 혹은 string인 경우 동작하는 함수.
function padLeft(value: string, padding: any) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // "Hello world"
```

- Union을 사용하는 예제

```ts
// 매개변수 padding의 타입이 number 혹은 string인 경우 동작하는 함수.
function padLeft(value: string, padding: number | string) {
	...
}

padLeft("Hello world", 4); // "Hello world"
```

```ts
// .d.ts
declare function padLeft(value: string, padding: number | string): string;
```

### Unions with common fields

Union type일 때, 모든 type에 공통인 property에만 접근할 수 있으며,

만약 특정 type에만 명시된 property인 경우, type을 지정해주어야 한다.

```ts
interface Bird {
  fly(): void;
  layEggs(): void;
}

interface Fish {
  swim(): void;
  layEggs(): void;
}

declare function getSmallPet(): Fish | Bird;

let pet = getSmallPet();

// layEggs는 Common field이기 때문에 바로 호출이 가능함.
pet.layEggs();

// ERR - Property 'swim' does not exist on type 'Bird | Fish'.
// ERR - Property 'swim' does not exist on type 'Bird'.
pet.swim();

// Type assertion을 해주면 ERR가 발생하지 않음.
(pet as Fish).swim();
```

### Discriminating Unions / Union 구별

Union을 사용하는 경우,  특정 타입을 추론하기 위해서 리터럴 타입을 갖는 단일 필드를 이용하는 방법이 있다.

- 아래 예제의 `state` property를 이용하는 방법.

```ts
type NetworkLoadingState = {
  state: "loading";
};

type NetworkFailedState = {
  state: "failed";
  code: number;
};

type NetworkSuccessState = {
  state: "success";
  response: {
    title: string;
    duration: number;
    summary: string;
  };
};

type NetworkState =
  | NetworkLoadingState
  | NetworkFailedState
  | NetworkSuccessState;

function networkStatus(state: NetworkState): string {
  // 현재 TypeScript는 셋 중 어떤 것이 state가 될 수 있는 잠재적인 타입인지 알 수 없음.
	...

  // 유니언 타입 좁히기
  switch (state.state) {
    case "loading":
      return "Downloading...";
    case "failed":
      return `Error ${state.code} downloading`;
    case "success":
      return `Downloaded ${state.response.title} - ${state.response.summary}`;
  }
}
```

## Intersection Types / 교차 타입

여러 타입을 하나로 결합하여 필요한 기능을 모두 가진 단일 타입을 얻는 방법.

```ts
interface ErrorHandling {
  success: boolean;
  error?: { message: string };
}

interface ArtworksData {
  artworks: { title: string }[];
}

interface ArtistsData {
  artists: { name: string }[];
}

type ArtworksResponse = ArtworksData & ErrorHandling;
/**
ArtworksResponse = {
  artworks: { title: string }[];
	success: boolean;
  error?: { message: string };
}
**/
type ArtistsResponse = ArtistsData & ErrorHandling;
/**
	artists: { name: string }[];
	success: boolean;
  error?: { message: string };
**/
```

### Mixins via Intersections / 교차를 통한 믹스인

- Mixins ?
    - 프로토 타입을 바꾸지 않고 한 객체의 프로퍼티를 다른 객체에 복사해서 사용하는 방식.
    - 상속과 다르게 필요한 프로퍼티만을 복사하는 방식.

```ts
class Person {
  constructor(public name: string) {}
}

interface Loggable {
  log(name: string): void;
}

class ConsoleLogger implements Loggable {
  log(name: string) {
    console.log(`Hello, I'm ${name}.`);
  }
}

// 두 객체를 받아 하나로 합칩니다.
function extend<First extends {}, Second extends {}>(
  first: First,
  second: Second
): First & Second {
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

const jim = extend(new Person("Jim"), ConsoleLogger.prototype);
jim.log(jim.name);
```

## References

[TypeScript 한글 문서](https://typescript-kr.github.io/pages/unions-and-intersections.html)

[Handbook - Unions and Intersection Types](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html)
