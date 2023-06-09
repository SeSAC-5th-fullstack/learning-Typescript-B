# Chapter 10

## 10.1 제네릭 함수

제네릭은 C++의 템플릿 기능과 유사하다. 제네릭 함수는 제네릭 타입이 적용된 함수를 말한다. 또한, 제네릭은 단일 타입이 아닌 다양한 타입에서 작동하는 컴포넌트를 작성할 수 있는 유용한 도구이다.

함수 이름 뒤에 <> 표기를 넣어서, <> 안에 제네릭 타입 매개변수를 선언할 수 있다. 관례적으로 파스칼 케이스를 사용한다.

제네릭 함수의 타입 매개변수는 원하는 만큼 선언할 수 있다. 복수의 타입 매개변수를 선언했다면, 호출시에 타입 인수를 모두 명시 하거나, 모두 생략해야 한다.

제네릭 함수를 호출 할 때 타입 인수를 명시하지 않으면, 타입 인수의 타입을 unknown으로 간주한다.

```tsx
function logWrapper<Input>(callback: (input: Input) => void) {
    return (input: Input) => {
        console.log("Input: ", input);
        callback(input);
    };
}

// <> 구문 생략, 타입 인수를 명시하지 않음
// 타입: (input: string) => void
logWrapper(
    (input: string) => {
        console.log(input.length);
    }
);

// <> 구문 생략, 타입 인수를 명시하지 않음
// 타입: (input: unknown) => void
logWrapper(
    (input) => {
        console.log(input.length); // Error
    }
);
```

## 10.2 제네릭 인터페이스

인터페이스도 제네릭을 적용하여 나타낼 수 있다.

제네릭 인터페이스 역시 여러 개의 타입 매개변수를 가질 수 있다.

타입 인수는 제네릭 함수와 마찬가지로 인스턴스를 통해 유추 가능하다.

```tsx
interface LinkedNode<Value> {
	next?: LinkedNode<Value>;
	val: Value;
}

function getLast<Value>(node: LinkedNode<Value>): Value {
	return node.next ? node.next : node.val;
}

// 유추된 Value 타입 인수: Date
let lastDate = getLast({
	val: new Date("09-13-1993"),
});

// 유추된 Value 타입 인수: string
let lastFruit = getLast({
	next:{
		val: "banana",
	},
	val: "apple",
});

// 유추된 Value 타입 인수: number
let lastMismatch = getLast({
	next:{
		val: 123,
	},
	val: false, // Error: 'boolean' type is not assignable to type 'number'
});
```

제네릭 인터페이스를 어떤 변수의 타입 애너테이션으로 사용하는 경우, 이에 상응하는 타입 인수를 반드시 제공해야 한다.

```tsx
interface CreateLike<T> {
    contents: T;
}

let miss: CreateLike = { // Error: Generic type 'CreateLike' requires 1 type argument.
    contents: 123,
}
```

## 10.3 제네릭 클래스

- 제너릭 클래스
    - 명시적 제너릭 클래스 타입
    - 제너릭 클래스 확장
    - 제너릭 인터페이스 구현 (aka. implements 키워드)
    - 메서드 제너릭
    - 정적 클래스 제너릭

### 명시적 제너릭 클래스 타입

클래스도 멤버에서 사용할 여러 개의 타입 매개변수를 선언할 수 있다.

생성자에 전달된 인수에서 타입 인수를 유추할 수 없는 경우에는 타입 인수의 기본값은 unknown이 된다. 이는 타입 인수를 <>에서 명시함으로써 unknown 값을 피할 수 있다.

```tsx
class CurriedCallback<Input> {
    #callback: (input: Input) => void;

    constructor(callback: (input: Input) => void){
        this.#callback = (input: Input) => {
            console.log("Input: ", input);
            callback(input);
        };
    }
    
    call(input: Input) {
        this.#callback(input);
    }
}

new CurriedCallback((input: string) => {
    console.log(input.length);
})

new CurriedCallback((input) => {
    console.log(input.length); // Error: input이 unknown 타입으로 간주됨.
})

new CurriedCallback<string>((input) => {
    console.log(input.length); // OK. 타입 인수 명시.
})
```

### 제너릭 클래스 확장

제니릭 클래스 역시 extends 키워드를 사용하여 클래스를 확장할 수 있다.

*타입스크립트는 기본 클래스에 대한 타입 인수를 유추하지 않는다.* 그러므로, 기본값이 없는 타입 인수는 명시적 타입 애너테이션을 지정해야 한다.

```tsx
class Quote<T> {
    lines: T;

    constructor(lines: T) {
        this.lines = lines;
    }
}

class SpokenQuote extends Quote<string[]> {
    speak() {
        console.log(this.lines.join("\n"));
    }
}

new Quote("THe only real failure is the failure to try.").lines;
new Quote([4, 8, 16, 32]).lines;

new SpokenQuote(["They may forget what you said", "but they never forget how you made them feel."]).lines;
new SpokenQuote([4, 8, 16, 32]).lines; // Error.
```

### 제너릭 인터페이스 구현

제네릭 클래스는 모든 매개변수를 인터페이스에 제공함으로써 제너릭 인터페이스를 구현한다. 이 때, 기본 인터페이스의 타입 매개변수는 클래스에서 선언되어야 한다.

```tsx
interface ActingCredit<Role> {
    role: Role;
}

class MoviePart implements ActingCredit<string> { // 인터페이스의 타입 매개변수는 클래스에서 <string>으로 선언됨.
    role: string;
    speaking: boolean;

    constructor(role: string, speaking: boolean) {
        this.role = role;
        this.speaking = speaking;
    }
}

const part = new MoviePart("Iron Man III", true);
part.role; // 타입: string
```

### 메서드 제너릭

클래스 메서드는 클래스 인스턴스와 별개로 자체 제네릭 타입을 선언할 수 있다.

호출은 각 타입 매개변수에 대해 다른 타입 인수를 갖는다.

```tsx
class CreatePairFactory<Key> {
    key: Key;

    constructor(key: Key) {
        this.key = key;
    }

    createPair<Value>(value: Value) {
        return { key: this.key, value };
    }
}

const factory = new CreatePairFactory("role");

// 타입: { key: string, value: number }
const numberPair = factory.createPair(123);

// 타입: { key: string, value: string }
const stringPair = factory.createPair("Shakespeare");
```

### 정적 클래스 제너릭

클래스의 정적 멤버는 인스턴스 멤버와 구분되므로, 정적 멤버는 인스턴스 멤버에 접근할 수 없다.

```tsx
class BothLogger<OnInstance> {
    instanceLog(value: OnInstance) {
        console.log(value);
        return value;
    }

    static staticLog<OnStatic> (value: OnStatic) { // 정적 멤버(메서드)
        // let fromInstance: OnInstance; // Error. OnInstance는 인스턴스 멤버임.
        
        console.log(value);
        return value;
    }
}

const logger = new BothLogger<number[]>;
logger.instanceLog([1,2,3]);

BothLogger.staticLog([false, true]);
BothLogger.staticLog<string>("Gold medals are made out of sweat, blood and tears and effort in the gym every day.");
```

## 10.4 제네릭 타입 별칭

type 키워드를 사용한 type alias에도, 제네릭을 적용하여 타입 매개변수를 작성할 수 있다.

일반적으로 제네릭 함수의 타입을 설명하는 함수와 함께 사용된다.

```tsx
type CreatesValue<Input, Output> = (input: Input) => Output;

let creator: CreatesValue<string, number>;
creator = (text) =>  { return text.length };

console.log(creator("abc")); // 3
```

## 10.5 제네릭 제한자

제네릭 타입 매개변수 뒤에 = 연산자를 선언하여 기본값을 제공할 수 있다.

기본값을 가진 매개변수는 가장 마지막에 있어야 한다.

```tsx
interface KeyValuePair<Key, Value = Key> {
    key: Key;
    value: Value;
}

// 타입: KeyValuePair<string, number>
let allExplicit: KeyValuePair<string, number> = {
    key: "rating",
    value: 10,
}

// 타입: KeyValuePair<string, string>
let oneDefaulting: KeyValuePair<string> = {
    key: "rating",
    value: "ten",
}

// 타입 에러: 인수를 전달하지 않아서 타입을 유추할 수 없음.
let firstMissing: KeyValuePair = {
    rating: "rating",
    value: 10,
}
```

## 10.6 제한된 제네릭 타입

타입 매개변수가 타입을 확장해야 할 때 extends 키워드를 사용하여 확장할 수 있다. extends 뒤에 오는 타입은 인터페이스 타입만 가능하다.(?)

```tsx
interface Lengthwise {
  length: number;
}
 
function loggingIdentity<Type extends Lengthwise>(arg: Type): Type {
  console.log(arg.length); // Now we know it has a .length property, so no more error
  return arg;
}
```

keyof 연산자는 제한된 타입 매개변수와도 잘 작동한다.

```tsx
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}

const x = {
  a: 1, 
  b: 2, 
  c: 3, 
  d: 4 
};

getProperty(x, "c"); // 3
getProperty(x, "m"); // Error: 'm' is not assingable to parameter type of 'a | b | c | d'
```

## 10.7 Promise

자바스크립트의 Promise는 요청 이후 결과를 받기까지 대기가 필요한 것을 나타낸다. 각 Promise는 대기 중인 작업이 resolve 또는 reject 하는 경우 콜백을 등록하기 위한 메서드를 제공한다.

타입스크립트에서 Promise 생성자는 단일 매개변수를 받도록 작성된다.

```tsx
// 타입: Promise<string>
const textEventually = new Promise<string>(
    (resolve) => { 
        setTimeout( () => resolve("Done"), 1000); // 1초 후에 resolve() 호출
    }
);

// 타입: Promise<void>
const lengthEventually = textEventually.then( (text) => { console.log(text.length); });
```

function 앞에 async를 붙이면 해당 함수는 항상 Promise를 반환한다. Promise가 아닌 값을 반환하더라도 이행 상태의 Promise로 값을 감싸 이행된 Promise가 반환되도록 한다.

await 연산자는 Promise에 의해 만족되는 값이 반환된다. async 함수 내부에서만 사용할 수 있다.

```tsx
// 타입: (text: string) => Promise<number>
async function lengthAfterSecond(text: string) {
    await new Promise( (resolve) => setTimeout(resolve, 1000) );
    return text.length;
}

// 타입: (text: string) => Promise<number>
async function lengthImmediately(text: string) {
    return text.length;
}

const ret = lengthAfterSecond("str");
console.log(ret); // 1초 뒤에, Promise 출력

async function process() {
    const ret = await lengthAfterSecond("string");
    console.log(ret);
}
process(); // 1초 뒤에, 6 출력
```

## 10.8 제네릭 올바르게 사용하기

1. 타입 매개변수는 최소 2개 이상일 때 권장
2. 제네릭 명명 규칙은 파스칼 케이스 사용, 하나의 이니셜 보다는 풀네임을 사용