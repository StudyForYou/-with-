# 타입 활용하기

## 📝 조건부 타입

- 기존 타입을 사용해서 새로운 타입을 정의하는 것
- 기본적으로 `extends`, `교차 타입`, `유니온 타입`을 사용하여 타입을 확장

### ✏️ extends와 제네릭을 활용한 조건부 타입

- `T extends U ? X : Y`
- 타입 T를 U에 할당할 수 있으면 X 타입, 없으면 Y 타입으로 결정됨

  ```ts
  interface Bank {
    financialCode: string;
    fullName: string;
  }
  interface Card {
    financialCode: string;
    appCardType?: string;
  }
  type PayMethod<T> = T extends "card" ? Card : Bank;
  type CardPayMethodType = PayMethod<"card">;
  type BankPayMethodType = PayMethod<"bank">;
  ```

  - PayMethod의 매개변수가 "card"일 시 Card 타입, 아닐 시 Bank 타입

### ✏️ 조건부 타입을 사용하지 않았을 때의 문제점

#### React-Query + TypeScript 프로젝트 예시 코드

- 계좌(bank), 카드(card), 앱카드(appcard) 3가지의 결제 수단이 있다.
- 주어진 결제 수단 타입을 서버 응답을 처리하는 공통 함수(`useGetRegisteredList`)에 전달
- 각 API를 통해 결제 수단 정보를 배열로 받아와 최종적으로 필터링된 배열(`result`)을 반환하는 코드

  ```ts
  type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

  export const useGetRegisteredList = (
    type: "card" | "appcard" | "bank"
  ): UseQueryResult<PayMethodType[]> => {
    const url = `baeminpay/codes/${type === "appcard" ? "card" : type}`;
    const fetcher = fetcherFactory<PayMethodType[]>({
      onSuccess: (res) => {
        const usablePocketList =
          res?.filter(
            (pocket: PocketInfo<Card> | PocketInfo<Bank>) =>
              pocket?.useType === "USE"
          ) ?? [];
        return usablePocketList;
      },
    });
    const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
    return result;
  };
  ```

  - 하지만, 타입을 명확히 구분하는 로직이 없다.
  - 사용자가 인자로 "`card`"를 전달했을 때 함수가 반환하는 타입인 PayMethodType이 유니온으로 되어 있기 때문에 구체적으로 추론 불가능(`PayMethodInfo<Card>[]`면 좋겠지만 구체적으로 추론 X)

### ✏️ extends 조건부 타입을 활용하여 개선하기

```ts
type PayMethodType<T extends "card" | "appcard" | "bank"> = T extends
  | "card"
  | "appcard"
  ? Card
  : Bank;
```

- PayMethodType의 제네릭으로 받은 값이 "card" or "appcard"면 `PayMethodInfo<Card>` 타입을, 아니라면 `PayMethodInfo<Bank>` 타입을 반환

```ts
export const useGetRegisteredList = <T extends "card" | "appcard" | "bank">(
  type: T
): UseQueryResult<PayMethodType<T>[]> => {
  /* ... */
  const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher);
  return result;
};
```

#### `활용 예시 정리`

1. 제네릭과 extends를 함께 사용해 제네릭으로 받는 타입을 제한

   - 잘못된 값을 넘길 수 없기 때무에 휴먼 에러 방지

2. extends를 활용해 조건부 타입을 설정

   - 반환 값을 사용자가 원하는 값으로 구체화 가능
   - 불필요한 타입 가드, 타입 단언 등을 방지
   - 불필요한 타입 가드와 불필요한 타입 단언을 하지 않아도 된다!

### ✏️ infer를 활용해서 타입 추론하기

- extends를 사용할 때만 `infer` 키워드를 사용 가능

```ts
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;
const promises = [Promise.resolve("Mark"), Promise.resolve(38)];
type Expected = UnpackPromise<typeof promises>; // string | number
```

#### 왜 사용할까?

- Promise 처럼 Generic으로 받는 타입 내부의 타입을 추론할때 사용합니다.

## 📝 템플릿 리터럴 타입 활용하기

- JS의 템플릿 리터럴 문법을 사용해 특정 문자열에 대한 타입을 선언할 수 있는 기능
- 타입스크립트 4.1부터 지원하고 있는 기능이다.

  ```ts
  type Vertical = "top" | "bottom";
  type Horizon = "left" | "right";

  type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}`;
  // "top" | "topLeft" | "topRight" | "bottom" | "bottomLeft" | "bottomRight"
  ```

  - 더욱 읽기 쉬운 코드 작성 가능
  - 코드를 재사용하고 수정하는 데 용이한 타입 선언 가능

### 🚨 주의할 점

- TS 컴파일러가 유니온을 추론하는 데 시간이 오래 걸리면 비효율적이기 때문에 TS가 타입을 추론하지 않고 에러를 내뱉을 때가 있다.
- 템플릿 리터럴 타입에 삽입된 `유니온 조합의 경우의 수가 너무 많지 않게 적절하게 나누어 타입을 관리`하는 것이 좋다.

  ```ts
  type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
  type Chunk = `${Digit}${Digit}${Digit}${Digit}`;
  type PhoneNumberType = `010-${Chunk}-${Chunk}`; // Error - Expression produces a union type that is too complex to represent.
  ```

  - 위의 예제일 경우 PhoneNumberType은 10000 \*\* 2의 경우의 수를 가지고 있는 유니온 타입이 되기 때문에 TS에서 에러가 발생할 수 있다.

## 📝 커스텀 유틸리티 타입 활용하기

### ✏️ 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

- `Pick`, `Omit` 과 같은 유틸리티 함수를 활용하여 중복되는 코드를 제거할 수 있다.
- 유틸리티 함수를 사용하면 중복 코드 제거 및 유지보수에 용이

  ```ts
  export type Props = {
    height?: string;
    color?: keyof typeof colors;
    isFull?: boolean;
    className?: string;
    //...
  };

  export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
    //...
    return (
      <HrComponent
        height={height}
        color={color}
        isFull={isFull}
        classNane={className}
      ></HrComponent>
    );
  };

  //styles.ts
  //Pick 유틸리티 타입 활용: 특정 타입에서 몇 개의 속성을 선택하여 타입을 정의
  type StyledProps = Pick<Props, "height" | "color" | "isFull">;
  const HrComponent = styled.hr<StyledProps>`
    height: ${({ height }) => height || "10px"};
    background-color: ${({ color }) => colors[color || "gray7"]};
    ${({ isFull }) =>
      isFull &&
      css`
        margin: 0 -16px;
      `}
  `;
  ```

  #### Pick 과 Omit

  - `Pick` - 특정 타입에서 몇 개의 속성을 선택하여 타입을 정의
  - `Omit` - 특정 속성만 제거한 타입을 정의

  ```ts
  interface Product {
    id: number;
    name: string;
    price: number;
    brand: string;
    stock: number;
  }

  type shoppingInfo = Pick<Product, "id" | "name">;
  /*
  type shoppingInfo = {
    id: number;
    name: string;
  }
  */
  type shoppingInfo2 = Omit<Product, "id" | "name">;
  /*
  type shoppingInfo2 = {
    price: number;
    brand: string;
    stock: number;
  }
  */
  ```

### ✏️ PickOne 유틸리티 함수

#### 🚨 문제 상황

- TS는 서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 제대로 되지 않는 이슈가 있다.

  ```ts
  type Card = {
    card: string;
  };
  type Account = {
    account: string;
  };
  function withdraw(type: Card | Account) {}
  withdraw({ card: "hyundai", account: "hana" }); // Card와 Account 속성을 한 번에 받아도 에러 없음
  ```

  - 유니온은 집합 관점으로 볼 때 합집합이 되기 때문에 card, account 모두 포함되어도 합집합의 범주에 들어가기 때문에 타입에러가 발생하지 않는다.

  ### 해결방법 1. 식별할 수 있는 유니온 사용

  ```ts
  type Card = {
    type: "card"; // 판별자 추가
    card: string;
  };
  type Account = {
    type: "account"; // 판별자 추가
    account: string;
  };
  function withdraw(type: Card | Account) {}
  withdraw({ card: "hyundai", account: "hana" }); // 에러 발생 - Argument of type '{ card: string; account: string; }' is not assignable to parameter of type 'Card | Account'.
  withdraw({ type: "card", card: "hyundai" });
  withdraw({ type: "account", account: "hana" });
  ```

  - 일일이 type을 다 넣어줘야 하는 불편함이 있다.
  - 이미 구현된 상태에서 식별할 수 있는 유니온을 적용하려면 해당 함수를 사용하는 부분을 모두 수정해야하는 번거러움도 있다.

  ### 해결방법 2. PickOne 커스텀 유틸리티 타입 구현하기

  - 하나의 속성이 들어왔을 때 다른 타입을 옵셔널한 undefined 값으로 지정하는 방법
  - -> 사용자가 의도적으로 undefined 값을 넣지 않는 이상, 원치 않은 속성에 값을 넣었을 때 타입 에러가 발생한다.

  ```ts
  type PickOne<T> = {
    [P in keyof T]: Record<P, T[P]> &
      Partial<Record<Exclude<keyof T, P>, undefined>>;
  }[keyof T];
  ```

### ✏️ NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

#### NonNullable 타입

- `TS에서 제공하는 유틸리티 타입`으로 제네릭으로 받는 T가 null or undefined일 때 never or T를 반환하는 타입
- null이나 undefined가 아닌 경우를 제외하기 위해 사용

  ```ts
  type NonNullable<T> = T extends null | undefiend ? never : T;
  ```

#### NonNullable 커스텀 함수

```ts
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

- 매개변수(value)가 null 또는 undefined일 때 false를 반환하는 함수
- 반환값이 true라면 null과 undefined가 아닌 다른 타입으로 타입 가드된다.

#### Promise.all 사용 시 NonNullable 적용 예시

```ts
const shopList = [
  { shopNo: 100, category: "chicken" },
  { shopNo: 101, category: "pizza" },
  { shopNo: 102, category: "noodle" },
];

// 반환 타입이 Array<AdCampaign[] | null>로 추론된다.
class AdCampaignAPI {
  static async operating(shopNo: number): Promise<AdCampaign[]> {
    try {
      return await fetch(`/ad/shopNumber=${shopNo}`);
    } catch (error) {
      return null;
    }
  }
}

const shopAdCampaignList = await Promise.all(
  shopList.map((shop) => AdCampaignAPI.operating(shop.shopNo))
);

// null이나 undefined값을 필터링 할 수 있게 된다.
const shopAds = shopAdCampaginList.filter(NonNullable);
```

- ### 그럼 shopAdCampaginList.filter(shop => !!shop)을 사용하면 되지 않을까?
  - JS에서 제공하는 filter method는 결과 배열에 요소를 유지하려면 true, 아니면 false를 반환합니다.
  - 즉, null 타입을 타입 추론해주지 않습니다.

## 📝 불변 객체 타입으로 활용하기

- 컴포넌트나 함수에서 객체를 사용할 때 열린 타입(`any`)으로 설정할 수 있다.

  ```ts
  const colors = {
    red: "#F45452",
    green: "#0C952A",
    blue: "#1A7CFF",
  };

  const getColorHex = (key: string) => colors[key]; // 에러 발생 - colors에 어떤 값이 추가될지 모르기 때문에 getColorHex의 반환값은 any
  ```

  - 두 가지 방법을 통해 객체 타입을 더 정확하고 안전하게 설정할 수 있다.
    - `as const` 키워드로 객체를 불변(`readonly`) 객체로 선언
    - `keyof` 연산자로 함수 인자를 colors 객체에 존재하는 키값만 받도록 설정

  ```ts
  const colors = {
    red: "#F45452",
    green: "#0C952A",
    blue: "#1A7CFF",
  } as const; // colors 객체를 불변 객체로 선언

  const getColorHex = (key: keyof typeof colors) => colors[key];
  const redHex = getColorHex("red");
  const unknownHex = getColorHex("yellow"); // 오류 발생
  ```

### ✏️ Atom 컴포넌트에서 theme style 객체 활용하기

- 대부분의 프로젝트에서는 `스타일 값을 theme 객체`를 두고 관리한다.

```ts
const colors = {
  black: "#000000",
  gray: "#222222",
  white: "#FFFFFF",
  mint: "#2AC1BC",
};

const theme = {
  colors: {
    default: colors.gray,
    ...colors,
  },
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
};
```

```tsx
interface Props {
  fontSize?: string;
  color?: string;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}

const Button: FC<Props> = ({ fontSize, color, children }) => {
  return (
    <ButtonWrap fontSize={fontSize} color={color}>
      {children}
    </ButtonWrap>
  );
};

// 컴포넌트 ButtonWrap이 props로 스타일 키 값(fontSize, backgroundColor, color)을 전달받음
const ButtonWrap = styled.button<Omit<Props, "onClick">>`
  color: ${({ color }) => theme.color[color ?? "default"]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize ?? "default"]};
`;
```

- fontsize, backgroundcolor 같은 props 타입이 stirng 타입일 경우
  1. 키값이 자동 완성되지 않는다.
  2. 잘못된 키값을 넣어도 에러가 발생하지 않는다.
- 이러한 문제는 `타입을 구체화`해 해결 가능하다.

  ### 해결방법 - keyof, typeof 를 사용하여 타입을 구체화

  ```ts
  type ColorType = typeof keyof theme.colors;  // "default" | "black" | "gray" | "white" | "mint"
  type FontSizeType = typeof keyof theme.fontSize;  // "default" | "small" | "large"

  interface Props {
    fontSize?: ColorType;
    color?: FontSizeType;
    onCLick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
  }
  ```

## 📝 Record 원시 타입 키 개선하기

- 객체 선언 시 키가 어떤 값인지 명확하지 않으면 Record의 키를 string이나 number같은 원시 타입으로 명시하곤 하는데 이는 런타임 에러를 야기할 수 있어 주의가 필요하다.

### ✏️ 무한한 키를 집합으로 가지는 Record

```ts
type Category = string;
interface Food {
  name: string;
  // ...
}
const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};
```

- 객체 `foodByCategory`는 `string` 타입을 Record의 키로 사용하기 때문에 무한한 키 집합을 가지고 있다. 따라서 "한식", "일식"이 아닌 없는 키를 사용하더라도 타입 오류가 일어나지 않는다.

  ```ts
  foodByCategory["양식"]; // Food[]로 추론 - 오류 발생 X
  foodByCategory["양식"].map((food) => console.log(food.name)); // 런타임에서 오류 발생 - Cannot read properties of undefined (reading ‘map’)

  foodByCategory["양식"]?.map((food) => console.log(food.name)); // 정상 동작
  ```

  - undefined로 인한 런타임 에러를 방지하기 위해서 옵셔널 체이닝(`?.`)을 사용할 수 있지만 undefined일 수 있는 값을 인지하고 코드를 작성해야 하므로 휴먼 에러가 발생할 수 있다.

### ✏️ 유닛 타입으로 변경하기

- `키가 유한한 집합이라면 유닛 타입`을 사용할 수 있다.

```ts
type Category = "한식" | "일식";
interface Food {
  name: string;
  // ...
}

const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

foodByCategory["양식"]; //  오류 발생 - Property '양식' does not exist on type 'Record<Category, Food[]>'
```

- 하지만 키가 무한해야 하는 상황에는 적합하지 않다.

### ✏️ Partial을 활요앟여 정확한 타입 표현하기

- 키가 무한한 상황에서는 Partial을 사용하여 해당 값이 undefined일 수 있는 상태임을 표현할 수 있다.
- 객체 값이 undefined일 수 있는 경우에 Partial을 사용해서 `PartialRecord` 타입을 선언하고 객체를 선언할 때 이것을 활용할 수 있다.

```ts
type PartialRecord<K extends string, T> = Partial<Record<K, T>>;
type Category = string;

interface Food {
  name: string;
  // ...
}

const foodByCategory: PartialRecord<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

foodByCategory["양식"]; // Food[] 또는 undefined 타입으로 추론
foodByCategory["양식"].map((food) => console.log(food.name)); // 오류 발생 - Object is possibly 'undefined'
```

- 개발자는 안내를 보고 옵셔널 체이닝 or 조건문을 사용하여 사전에 조치하여 런타임 에러를 방지할 수 있다.
