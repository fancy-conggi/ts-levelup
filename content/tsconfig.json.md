# tsconfig.json.md

> 시작전에 궁금했던 것

- tsconfig.js 컴파일 과정 ?

## tsconfig.json

### tsc & tsconfig.json를 찾아가는 과정



`tsc` 명령어로 타입스크립트를 자바스크립트로 컴파일



npm으로 typescript 설치과정 생략

설치후에 버전확인

```shell
tsc -v
```



tsconfig.json 초기화

```shell
tsc --init
```



tsc를 호출화면 컴파일러는 현재 디렉토리부터 상위 디렉토리 체인으로 tsconfig.json 검색

```shell
tsc app.ts # tsconfig.json 무시
```



tsconfig.json 파일이 있다면 프로젝트의 root가 됨

`--project (-p)` 옵션을 사용 가능 > 입력파일을 지정하면 tsconfig.json 파일 무시



##  속성

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "sourceMap": true
    },
    "files": [
        "core.ts",
        "sys.ts",
        "types.ts",
        "scanner.ts",
        "parser.ts",
        "utilities.ts",
        "binder.ts",
        "checker.ts",
        "emitter.ts",
        "program.ts",
        "commandLineParser.ts",
        "tsc.ts",
        "diagnosticInformationMap.generated.ts"
    ]
     "include": [
        "src/**/*"
    ],
    "exclude": [
        "node_modules",
        "**/*.spec.ts"
    ]
}
```



### files, include, exclude

생략



### typeRoots, types (@types)

#### @types

대표적인 예로 `@types/classnames`, `@types/react`가 있음

타입스크립트 설정 파일은 기본적으로 `node_modules`를 제외하지만 

써드 파티 라이브러리의 타입을 정의해놓는 `@types` 폴더는 컴파일에 포함합니다. 



> 자세히 알아보기 전에.. `@types`는 어떻게 동작하는가 ?

- [d.ts 파일은 무엇인가?](https://www.slideshare.net/gloridea/dts-74589285) 

  - 타입스크립트 코드의 타입 추론을 돕는 파일 (react, jquery등등도 타입으로 정의가 되어 있어야 사용이 가능)

  - 프로젝트 내에 .d.ts 선언 > 없는 모듈의 @types를 만들어 npm에 올리면 좋지만, 필요한 부분만 만들어서 사용할 때

  - 파일 위치

    1. @types 안에 해당 모듈명의 폴더 생성

    2. 폴더 안에 index.d.ts 생성

    3. export 할 타입의 인터페이스 선언

    4. 해당 인터페이스 타입의 변수(상수무방) 선언

    5. export 선언

![image-20200928154916449](file:///Users/user/code/moonscode/gatsby-gitbook-starter/public/static/images/image-20200928154916449.png?lastModify=1601280839)

![image-20200928154932032](file:///Users/user/code/moonscode/gatsby-gitbook-starter/public/static/images/image-20200928154932032.png?lastModify=1601280839)



#### typeRoots

typeRoots를 지정하면 typeRoots 아래에 있는 패키지만 포함

이 설정파일에 따르면 ./typings의 모든 패키지가 포하되며 node_modules/@types는 포함되지 않음

```json
{
   "compilerOptions": {
       "typeRoots" : ["./typings"]
   }
}
```

`./typings`의 *모든* 패키지가 포함



#### types

type를 지정하면 @types 패키지가 자동으로 포함되지 않음

전역 선언이 포함된 파일을 사용하는 경우에만 자동 포함이 중요하다는 점에 명심

```json
{
   "compilerOptions": {
       "types" : ["node", "lodash", "express"]
   }
}
```

`./node_modules/@types/node`, `./node_modules/@types/lodash` 및 `./node_modules/@types/express`만 포함



### extends

- extends는 최상위 속성에 포함 (상속하려는 파일의 경로를 포함하는 문자열)

- 우선순위 : 기본 파일 설정 -> 상속 파일 설정 (순환성 에러에 유의)
  - 단, files, include, exclude는 기본 파일 설정을 덮어씀



/config/base.json

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```





tsconfig.json

```json
{
  "extends": "./configs/base",
  "files": [
    "main.ts",
    "supplemental.ts"
  ]
}
```



