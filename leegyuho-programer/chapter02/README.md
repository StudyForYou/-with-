> # 2.1 타입이란

### 자료형으로서의 타입

- 모든 프로그래밍 언어는 변수를 선언하는 것부터 시작
- 변수란 값을 저장할 수 있는 공간이자 값을 가리키는 상징적인 이름
- 7가지 데이터 타입(자료형)
  - undefined
  - null
  - Boolean
  - String
  - Symbol
  - Numeric(Number와 BigInt)
  - Object

### 집합으로서의 타입

- 타입 시스템은 코드에서 사용되는 유효한 값의 범위를 제한해서 런타임에서 발생할 수 있는 유효하지 않은 값에 대한 에러를 방지해줌

```js
function double(n) {
  return n * 2;
}

double(2); // 4
double('z'); // NaN
```

```js
function double(n: number) {
  return n * 2;
}

double(2); // 4
double('z'); // Error: Argument of type 'string' is not assignable to parameter of type 'number'
```

### 정적 타입과 동적 타입

- 정적 타입
  - 모든 변수의 타입이 컴파일 타임에 결정됨
  - `컴파일 타임`에 에러를 발견할 수 있기 때문에 프로그램의 안정성 보장
  - ex) C, Java, TypeScript
- 동적 타입
  - 변수 타입이 `런타임`에 결정
  - 직접 타입을 정의해줄 필요 X
  - ex) Python, JavaScript

### 강타입과 약타입

- 모든 프로그래밍 언어에는 값의 타입이 존재
- `암묵적 타입 변환`: 타입을 의도적으로 명시하지 않아도 컴파일러 또는 엔진 등에 의해 런타임에 타입이 자동으로 변경되는 것
- 암묵적 타입 변환 여부에 따라 타입 시스템을 강타입과 약타입으로 분류 가능
- `강타입`
  - 서로 다른 타입을 갖는 값끼리 연산을 시도하면 컴파일러 또는 인터프리터에서 에러 발생

```js
console.log('2' - 1); // 1 (javascript)
```

- `약타입`
  - 서로 다른 타입을 갖는 값끼리 연산할 때는 컴파일러 또는 인터프리터가 내부적으로 판단해서 특정 값의 타입을 변환하여 연산을 수행한 후 값을 도축

```ts
console.log('2' - 1); // '2' error type error (typescript)
```

### 컴파일 방식

- 컴파일: 사람이 이해할 수 있는 방식으로 작성한 코드를 컴퓨터가 이해할 수 있는 기계어로 바꿔주는 과정
- 타입스크립트의 컴파일 결과물은 여전히 사람이 이해할 수 있는 방식인 자바스크립트 파일<br>
  -> 타입스크립트를 컴파일하면 타입이 모두 제거된 자바스크립트 소스코드만 남는다

> # 2.2 타입스크립트의 타입 시스템

### 타입 애너테이션 방식

- 타입 애너테이션(type annotation)
  - 변수나 상수 혹은 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법

```ts
let isDone: boolean = false;
let decimal: number = 6;
let color: string = 'blue';
let list: number[] = [1, 2, 3];
let x: [string, number]; // tuple
```

### 구조적 타이핑

- 타입을 사용하는 언어에서 `값이나 객체는 하나의 구체적인 타입을 가짐`
- 타입은 이름으로 구분되며 `컴파일타임 이후에도 남아있음`
- 타입스크립트는 `구조로 타입을 구분`함

### 구조적 서브타이핑

- 타입 시스템에서 타입 호환성을 결정하는 방법 중 하나로 어떤 타입이 다른 타입을 대체할 수 있는지 여부를 결정하는 원칙
- 객체가 가지고 있는 속성을 바탕으로 타입을 구분하는 것
- `이름이 다른 객체라도 가진 속성이 돌일하다면` 타입스크립트는 서로 호환이 가능한 `동일한 타입으로 여김`
- 서로 다른 두 타입 간의 호환성은 오로지 `타입 내부의 구조에 의해 결정`됨

```ts
interface Pet {
  name: string;
}

interface Cat {
  name: string;
  age: number;
}

let pet: Pet;
let cat: Cat = { nameL: 'Zag'. age: 2 };

pet = cat //OK
```

- Cat은 Pet과 다른 타입으로 선언되었지만 Pet이 갖고 있는 name이라는 속성을 가지고 있다. 따라서 Cat 타입으로 선언한 cat을 Pet 타입으로 선언한 pet에 할당할 수 있다.

### 구조적 서브타이핑과 명목적 타이핑

- `구조적 서브타이핑`
  - 구조적 서브타이핑은 타입 호환성을 결정할 때 타입의 구조를 고려
  - 두 타입이 호환 가능하려면 그 구조가 호환되어야 한다. 즉, 두 타입이 동일한 구조를 갖고 있어야 한다.
  - 이러한 방식은 타입의 이름에 의존하지 않고 타입의 내부 구조에만 의존
  - 주로 동적 타입 언어나 일부 정적 타입 언어에서 사용<br>
    -> ex) JavaScript의 객체들은 그들이 갖고 있는 속성과 메서드에 따라 타입이 결정
- `명목적 타이핑`
  - 명목적 타이핑은 타입 호환성을 결정할 때 타입의 이름을 고려
  - 두 타입이 호환 가능하려면 그들의 이름이나 명시적으로 선언된 인터페이스가 동일해야함
  - 이러한 방식은 타입의 이름에 의존하므로, 동일한 구조를 갖고 있더라도 다른 이름을 가진 두 타입은 호환되지 않는다.
  - 타입의 동일성을 확인하는 과정에서 구조적 타이핑에 비해 조금 더 안전
  - 대부분의 정적 타입 언어에서 사용된다. <br>
    -> 예를 들어, Java나 C#에서 클래스나 인터페이스의 이름으로 타입 호환성을 판단함

<details>
<summary>💡 덕 타이핑</summary>
어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식<br>
"만약 어떤 새가 오리처럼 걷고, 헤엄치며 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다."
</details>

### 구조적 타이핑의 결과

- 구조적 타이핑의 특징 때문에 예기치 못한 결과가 나올 때도 있다.

```ts
interface Cube {
  width: number;
  height: number;
  depth: number;
}

function addLines(c: Cube) {
  let total = 0;

  for (const axis of Object.keys(c)) {
    const length = c[axis];

    total += length;
  }
}

// addLines() 함수의 매개변수인 c는 Cube 타입으로 선언되었고 Cube 인터페이스의 모든 필드는
// number 타입을 가지기 때문에 c[axis]는 당연히 number 타입일 것이라고 예차할 수 있지만
// width, height 외에도 어떤 속성이든 가질 수 있기 때문에 c[axis]의 타입이 string일 수도 있어 에러가 발생

const namedCube = {
  width: 6,
  height: 5,
  depth: 4,
  name: 'SweetCube', // string 타입의 추가 속성이 정의되었다.
};

addLines(namedCube); // Ok
```

### 타입스크립트의 점진적 타입 확인

- 컴파일 타임에서 타입을 검사하면서 필요에 따라 타입 선언 생략을 허용하는 방식

```ts
function add(x, y) {
  return x + y;
}

// 위 코드는 아래와 같이 암시적 타입 변환이 일어난다
function add(x: any, y: any): any;
```

### 값 vs 타입

- 값: 프로그램이 처리하기 위해 메모리에 저장하는 모든 데이터
- 값공간과 타입 공간의 이름은 서로 충돌하지 않기 때문에 `타입과 변수를 같은 이름으로 정의할 수 있다.`<br>
  -> type으로 선언한 내용은 자바스크립트 `런타임에서 제거`되기 때문

```js
11; // 숫자 값
('hello typescript'); // 문자열 값
let foo = 'bar'; // 변숫값

// 객체 역시 값이며 자바스크립트에서는 함수도 값이다.(런타임에 객체로 변환됨)
```

- 타입스크립트의 구조분해할당

```ts
function email(options: { person: Person; subject: string; body: string }) {
  // ...
}

function email({ person, subject, body }) {
  // ...
}

function email({ person, subject, body }: { person: Person; subject: string; body: string }) {
  // ...
}
```

- 값과 타입 공간에 동시에 존재하는 심볼도 있다. 대표적인 것이 클래스와 enum이다.
- 클래스(class)

  - 객체 인스턴스를 더욱 쉽게 생성하기 위한 문법기능으로 실제 동작은 함수와 같음

  ```ts
  class Developer {
    name: string;
    domain: string;

    constructor(name: string, domain: string)
    this.name = name;
    this.domain = domain
  }

  // 변수명 me 뒤에 등장하는 Developer에서 Developer는 타입에 해당
  // new 키워드 뒤의 Developer는 클래스의 생성자 함수인 값으로 동작
  ```

- enum
  - 런타임에 객체로 변환되는 값
  - 타입 공간에서 타입을 제한하는 역할을 하지만 자바스크립트 런타임에서 실제 값으로도 사용될 수 있음

| 키워드          | 값  | 타입 |
| --------------- | --- | ---- |
| class           | Y   | Y    |
| const, let, var | Y   | N    |
| enum            | Y   | Y    |
| function        | Y   | N    |
| interface       | N   | Y    |
| type            | N   | Y    |
| namespace       | Y   | N    |

### 타입을 확인하는 방법

- interfaceof
- typeof

```ts
interface Person {
  first: string;
  last: string;
}

const person: Person = { first: 'zig', last: 'song' };

function email(options: { pserson: Person; subject: string; body: string }) {}

const v1 = typeof person; // 값은 'object'
const v2 = typeof email; // 값은 'function'

type T1 = typeof person; // 타입은 Person
type T2 = typeof email; // 타입은 (options: { pserson: Person; subject: string; body: string }) => void
```

```ts
class Developer {
  name: string;
  sleepingTime: number;

  constructor(name: string, sleepingTime: number)
  this.name = name;
  this.sleepingTime = sleepingTime
}

const d = typeof Developer; // 값이 'function'
type T = typeof Developer; // 타입이 typeof Developer
```

- as 키워드를 사용하여 타입을 강제할 수 있음

```ts
const loaded_text: unknown; // 어딘가에서 unknown 타입 값을 전달받았다고 가정

const validateInputText = (text: string) => {
  if (text.length < 10) return '최소 10글자 이상 입력해야 합니다.';
  return '정상 입력된 값입니다.';
};

validateinputText(loaded_text as string); // as 키워드를 사용해서 string으로 강제하지 않으면 타입스크립트 컴파일러 단계에서 에러 발생
```

> # 2.3 원시 타입

### boolean

- true와 false 값만 할당할 수 있는 boolean 타입
- 형 변환을 통해 true / false로 취급되는 Truthy / Falsy 값이 존재하지만 boolean 원시 값이 아니므로 boolean 타입에 해당하지 않음

### uncdfined

- 정의되지 않았다는 의미의 타입으로 오직 `undefined 값만 할당`할 수 있음

```ts
let value: string;
console.log(value); // undefined (값이 아직 할당되지 않음)

type Person = {
  name: string;
  job?: string;
};

//Person 타입의 job 속성은 옵셔널로 지정되어 있는데 이런 경우에도 undefined 할당 가능
```

### null

- 오직 `null만 할당` 가능

```ts
type Person1 = {
  name: string;
  job?: string;
};

type Person2 = {
  name: string;
  job: string | null;
};

// Person1은 job이라는 속성이 있을 수도 있고 없을 수도 있음을 나타냄
// Person2는 job이라는 속성을 사람마다 갖고 있지만 값이 비어있을 수도 있다는 것을 나타냄
```

### number

- 자바스크립트의 `숫자에 해당하는 모든 원시 값을 할당`할 수 있음
- NaN이나 Infinity도 포함

### bigInt

- Number.MAX_SAFE_INTEGER(2^53 - 1)를 넘어가는 값
- number 타입과 bigInt 타입은 엄연히 다른 값

### string

- 문자열을 할당할 수 있는 타입

### symbol

- 고유하고 변경 불가능한 값을 나타내며 주로 객체의 프로퍼티 키로 사용
- symbol 타입과 const 선언에서만 사용할 수 있는 unique symbol 타입이라는 symbol 하위 타입이 있다.

> # 2.4 객체 타입

- 7가지 원시 타입에 속하지 않는 값은 모두 객체 타입
- 다양한 형태를 가지는 객체마다 개별적으로 타입 지정 가능<br>
  -> 배열 또는 클래스를 타입으로 지정 가능

### object

- 가급적 사용하지 말도록 권장

### {}

- 중괄호는 자바스크립트에서 객체 리터럴 방식으로 객체를 생성할 때 사용
- 중괄호 안에 객체의 속성 타입을 지정해주는 식으로 사용

```ts
const noticePopup: { title: string; descriptrion: string } = {
  title: 'IE 지원 종료 안내',
  description: '2022.07.15일부로 배민상회 IE 브라우저 지원을 종료합니다.',
};
```

### array

- 타입스크립트 배열 타입은 하나의 타입 값만 가질 수 있다는 점에서 자바스크립트 배열보다 조금 더 엄격

```ts
const getCartList = async (cartId: number[]) => {
  const res = await CartApi.GET_CART_LIST(cartId);
  return res.getData();
};

getCartList([]);
getCartList([1001]);
getCartList([1001, 1002]);
```

### type과 interface 키워드

```ts
type NoticepopupType = {
  title: string;
  description: string;
}

interface INoticepopup {
  title: string;
  description: string;
}

const noticePopup1: NoticepopupType = { ... };
const noticePopup2: INoticepopup = { ... };
```

### function

- 함수를 별도 함수 타입으로 지정할 수 있다.
- 주의할 점

  - 자바스크립트에서 typeof 연산자로 확인한 function이라는 키워드 잧를 타입으로 사용하지는 않는다.
  - 함수는 매개변수 목록을 받ㅇ르 수 있는데 타입스크립트에서는 매개변수도 별도 타입으로 지정해야 한다.
  - 함수가 반환하는 값이 있다면 반환 값에 대한 타이핑도 필요하다.

  ```ts
  function add(a: number, b: number): number {
    return a + b;
  }
  ```
