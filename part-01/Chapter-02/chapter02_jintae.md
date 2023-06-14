# Chapter 02

## 2.1 타입의 종류

“타입”은 typeof 연산자가 설명하는 것을 의미한다. 가장 기본적인 타입은 자바스크립트의 일곱가지 기본 원시 타입과 동일하다.

- null
- undefined
- boolean
- string
- number
- bigint
- symbol

타입 시스템이란, 프로그래밍 언어가 프로그램에서 가질 수 있는 타입을 이해하는 방법에 대한 규칙 집합을 말한다. 타입 스크립트의 타입 시스템은 다음과 같다.

1. 코드를 읽고 존재하는 모든 타입과 값을 이해한다.
2. 각 값이 초기 선언에서 가질 수 있는 타입을 확인한다.
3. 각 값이 추후 코드에서 어떻게 사용될 수 있는지 모든 방법을 확인한다.
4. 값의 사용법이 타입과 일치하지 않으면 사용자에게 오류를 표시한다.

```tsx
let firstName = "Whitney";
firstName.length();

// 타입스크립트는 다음과 같은 순서로 오류를 표기한다.
// 1. firstName을 읽고 변수를 이해한다.
// 2. "Whitney" 초기값을 읽고, firstName이 string타입이라고 결론 짓는다.
// 3. firstName의 .length 메소드를 확인한다.
// 4. .length는 메소드가 아니므로 오류를 표시한다.
```

타입스크립트의 오류는 두 가지가 있다.

- 구문 오류 → 타입스크립트 코드를 자바스크립트 코드로 변환하지 않는다. 
(단, tsconfig.json파일에서 설정을 변경하면 변환이 가능하다.)
- 타입 오류 → 타입스크립트 코드를 자바스크립트 코드로 변환한다. 다만, 자바스크립트 코드의 동작이 이상할 수 있다.

## 2.2 할당 가능성

할당 가능성(assignability)이란, 전달된 값이 예상된 타입으로 할당이 가능한지 여부를 확인하는 것을 의미한다.

할당 가능성은 이후에 복잡한 객체를 비교할 때 더 중요한 용어가 된다.

## 2.3 타입 애너테이션

초기 타입을 유추할 수 없는 변수는 진화하는 any라고 한다. 진화하는 any는 새로운 값이 할당될 때마다 변수 타입을 발전시킨다.

```tsx
let rocker;
rocker = "Joan Jett"; // 타입: string

rocker = 1234; // 타입: number
```

이를 방지하기 위해서 타입 애너테이션(type annotation)을 활용할 수 있다. 타입 애너테이션은 초기값이 할당되지 않은 변수에 미리 타입을 고정시킬 수 있는 수동적인 장치이다.

```tsx
let rocker: string
rocker = "Joan Jett"

rocker = 1234 // Error
```

필자 왈, 초기값이 할당되어 타입을 추론할 수 있는 변수에 대해서는 타입 애너테이션을 쓰지 않는 것을 권장한다.

다만, 코드를 명확하게 문서화 하거나 실수로 변수 타입이 변경되지 않도록 타입스크립트를 보호하는 경우는 타입 애너테이션을 중복시키기도 한다.

```tsx
let rocker:string = "Joan Jett" // 타입 애너테이션 중복
```

## 2.4 타입 형태

타입스크립트는 변수에 할당된 값이 원래 타입과 일치하는지 확인하는 것 이상을 수행한다. 타입스크립트는 접근하려는 속성이 해당 변수의 타입에 존재하는지 확인한다.

```tsx
let cher = {
	firstName = "Masatoshi";
	lastName = "Yamaguchi";
};

cher.middleName; // Error
```

**모듈**

- 모듈: export 또는 import가 있는 파일
- 스크립트: 모듈이 아닌 파일

타입스크립트의 스크립트 파일은 전역 스코프를 갖는 것으로 간주한다. 따라서 각 스크립트 파일 내의 변수들은 이름이 중복되면 오류가 발생한다.  

이와 달리 모듈 파일 내의 변수에서 export한 변수와 import한 변수의 이름이 겹치지만 않으면 상관 없다. 다시 말해서, 각각의 모듈에서 변수명이 겹쳐도 문제 없다.

```tsx
// a.ts
export const shared = "Cher";
```

```tsx
// b.ts
export const shared = "Cher";
```

```tsx
// c.ts
import { shared } from "./a";

export const shared = "Cher"; // Error
```

스크립트를 강제로 모듈로 만들게 하려면 export {}; 을 추가해야 한다.

```tsx
// a.ts
const shared = "Cher";
...

// b.ts
const shared = "Cher";
...

// 위의 두 코드는 다른 스크립트 파일로 작성되었더라도, 오류가 발생한다.
// 이를 해결하기 위해서는 각 파일을 모듈 파일로 바꾸어 사용하면 된다.
```