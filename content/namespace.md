# Namespace

##  Namespace 개념

Namespace는 파일 시스템에서 파일이 어떻게 구성되었는지 같은 자질구래한 세부사항을 추상화



eg.

.mine 함수가 schemes/scams/bitcoin/apps에 위치한다는 사실을 알 필요 없이

Schemes.Scams.Bitcoin.App.mine 같은 짧고 간편한 namespace로 접근이 가능



### Namespace 특징

Namespace는 코드를 구성하는 Ts만의 고유한 방법

Mobule과 달리 여러 개의 파일을 포괄, (--outFile을 사용해 연결)

WebApplication에서 코드를 구조화하기에 좋은 방법, 모든 의존성은 HTML 페이지의 script 태그로 포함



### Namespace 주의점

**reference를 사용한 모듈**

일반적인 실수는 Module 파일을 참조하기 위해 import문 대신 reference 태그 구문을 사용하는 것



[import vs. reference 차이점]([https://medium.com/naver-fe-platform/%ED%83%80%EC%9E%85%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%BB%B4%ED%8C%8C%EC%9D%BC%EB%9F%AC%EA%B0%80-%EB%AA%A8%EB%93%88-%ED%83%80%EC%9E%85-%EC%84%A0%EC%96%B8%EC%9D%84-%EC%B0%B8%EC%A1%B0%ED%95%98%EB%8A%94-%EA%B3%BC%EC%A0%95-5bfc55a88bb6](https://medium.com/naver-fe-platform/타입스크립트-컴파일러가-모듈-타입-선언을-참조하는-과정-5bfc55a88bb6)) 

자세한 내용은 위의 경로 확인...

- import  
  - ES6 Module 패턴
  - 이 패턴은 모듈의 타입 선언을 불러오는 동시에 
    해당 구문을 `require('modulename')` 포맷으로 변환
- reference
  - Triple-slash directive 
  - 이 패턴은 모듈의 타입 선언만 불러오며, `.js`로 변환된 파일에서 단순 주석으로 취급
  - 보통 글로벌에 구현된 모듈의 타입 선언을 불러올 때 사용

```
/// <reference path=”./mymodule.d.ts” /> 
/// <reference types=”mymodule” /> 
```





**불필요한 Namespace를 자제할 것**

Ts 모듈의 특징중 하나는 서로 다른 두개의 모듈이 절대 같은 스코드 안에 같은 이름을 제공하지 않는 것

따라서 불필요한 Namespace로 감싸줄 필요가 없음

```ts
export namespace Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```





> 책에서는...
>
> namespace를 캡슐화 수단으로는 권하지 않음
>
> module과 namespace중에 어느 것을 사용할지 고민이라면 module을 사용하는 것을 권장 



## 사용

### Namespace 선언

```ts
// Get.ts
namespace Network {
  export function get<T>(url: string) : Promise<T> {
    // ...
  }
}
  
// App.ts
namespace App {
  Network.get<GitRepo>('http.naver.com/repos/ts-levelup')
}
```

- namespace에는 반드시 이름이 있어야 하며 (eg. Network와 같은)
  함수, 변수, 타입, 인터페이스, 다른 namespace를 export할 수 있음

- Namespace 블록 안의 모든 코드는 명시적 export를 하지 않으면 외부에서 볼 수 없음



### Namespace 중첩

Namespace가 Namespace를 익스포트할 수 있음으로 Namespace 중첩구조도 쉽게 만들 수 있음



eg.

`Network.HTTP.get`, ` Network.TCP.listenOn` 방식과 같이 호출

```ts
namespace Network {
  export namespace HTTP {
    export function get<T> (url:string): Promise<T> { 
      
  }
    
  export namespace TCP {
    listenOn(prot : number) : Connection {
      // .. 
    }
    // ..
  }
    
  export namespace UDP { 
  	// ..
  }    
  
  export namespace IP {
    // ..
  }
}
```



Namespace를 여러개로 쪼개 관리할 수도 있음 

Ts는 이름이 같은 Namespace를 재귀적으로 합쳐줌

```ts
// HTTP.ts
namespace Network {
  export namespace HTTP {
    export function get<T> (url:string): Promise<T> { 
      // ..
  }
}
    
// TCP.ts
namespace Network {
  export namespace TCP {
    listenOn(prot : number) : Connection {
      // .. 
    }
    // ..
  }
}
  
// UDP.ts
..

// IP.ts
..
```



파일이 여러 개 있으면 컴파일된 코드가 모두 로드되는지 확인이 필요함 (typescript-kr.github.io 왈)

1. 모든 입력 파일을 하나의 JS 출력파일로 컴파일 하기 위해 `--outFile` 플래그를 사용하여
   연결 출력(concatenated output)을 사용

```
tsc --outFile Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

2. 파일별 컴파일(기본값)을 사용하여 각 입력 파일을 하나의 js 파일로 생성 여러 js 파일이 생성되는 경우 web-page에서 생성된 개별 파일을 적절한 순서로 로드하기 위해 script 태그를 사용

```html
<script src="Validation.js" type="text/javascript" />
<script src="LettersOnlyValidator.js" type="text/javascript" />
<script src="ZipCodeValidator.js" type="text/javascript" />
<script src="Test.js" type="text/javascript" />
```



### Namespace Alias

Namespace 계층이 너무 깊어졌다면 짧은 별칭(Alias)을 지어줄 수 있음

따로 문법이 있는 것은 아님

```ts
// A.ts
namespace A {
  namespace B {
    namespace C {
      export let d = 3
    }
  }
}
    
// MyApp.ts 
import d = A.B.C.d
let e = d * 3
```



### Namespace 충돌

같은 이름을 익스포트하면 충돌이 발생

단, 함수 타임을 정제할 때는 사용하는 `오버로드된 앰비언트 함수 선언(overloaded ambient function declaration)`에는 이름 충돌 금지 규칙이 적용되지 않음

> ambient(구현을 정의하지 않은 선언)



eg.

생략

## Namespace 컴파일

import, export와 달리 Namespace는 tsconfig.json의 module 설정에 영향을 받지 않으며

`항상 전역 변수로 컴파일`



```ts
//Flowers.ts
namespace Flowers {
  export function give(count: number):{ 
    return count + ' flowers'
  }
}
```



```js 
// Flowers.js
let Flowers
(function (Flowers) { // 1.
  function give(count) {
    return count + ' flowers'
  }
  Flowers.give = give // 2.
})(Flowers || (Flowers = {})) // 3.
```

1. Flowers는 클로저를 만들고 Flowers 모듈에서 명시적으로 import하지 않는 변수가 노출되는 것을 방지하기 위해 IIFE(즉시 실행 함수)안에 선언
2. Flowers namepsace로 import한 give함수를 할당
3.  Flowers namespace가 이미 전역으로 정의되어 있으면 그것을 확장
   그렇지 않으면 새로이 생성

