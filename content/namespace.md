# namespace

##  개념

namespace는 파일 시스템에서 파일이 어떻게 구성되었는지 같은 자질구래한 세부사항을 추상화



eg.

.mine 함수가 schemes/scams/bitcoin/apps에 위치한다는 사실을 알 필요 없이

Schemes.Scams.Bitcoin.App.mine 같은 짧고 간편한 namespace로 접근이 가능



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

