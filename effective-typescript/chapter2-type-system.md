# 6. 편집기를 사용하여 타입 시스템 탐색하기

편집기에서 타입스크립트 언어 서비스를 활용하자!

# 7. 타입이 값들의 집합이라고 생각하기

## 타입: 할당 가능한 값들의 집합

- `never`: 가장 작은 집합. 공집합으로 어떠한 값도 할당할 수 없다.
- `unit`: 한 가지 값만 포함하는 타입. ex) `type A = 'A';`
- `union`: 두 개 혹은 세 개로 묶은 타입. ex) `type AB = 'a' | 'b';`
- `unknown`: 전체(universal) 집합.

## 타입 연산자는 인터페이스의 속성이 아닌, 값의 집합에 적용된다.

```ts
interface Person {
	name: string
}
interface Lifespan {
	birth: Date
	death?: Date
}
type PersonSpan = Person & Lifespan
```

여기서 `&` 연산자는 값의 집합(타입의 범위)에 적용되기 때문에, 추가적인 속성을 가지는 값도 여전히 타입에 속하게 된다. 따라서 `Person`과 `Lifespan`을 둘 다 가지는 값은 인터섹션 타입에 속한다.

```ts
const ps: PersonSpan = {
	name: 'Alan Turing',
	birth: new Date('1912/06/23'),
	death: new Date('1954/06/07'),
} // 정상, 더 많은 값을 가져도 정상이다.
```

## `extends`의 의미는 `~의 부분 집합`

```ts
interface Person {
	name: string
}
interface PersonSpan {
	birth: Date
	death?: Date
}
```

`PersonSpan`은 `Person`의 서브타입이다. 클래스의 관점에서는 서브 클래스로 볼 수 있다.

`extends`는 제너릭 타입에서 한정자로도 사용된다. 아래의 `K`는 `string`을 상속받는다기 보다는, 부분 집합의 범위를 갖는 타입이 된다고 보는 것이 이해가 쉽다. `string` 리터럴 타입, `string` 리터럴 타입의 유니온, `string` 자신을 포함한다.

```ts
function getKey<K extends string>(val: any, key: K) { ... }
```

## 타입스크립트에서 불가능한 값

정수에 대한 타입, 또는 x와 y 속성 외에 다른 속성이 없는 객체는 타입스크립트 타입에 존재하지 않는다. `Exclude`를 사용해서 일부 타입을 제외할 수는 있으나, 적절한 타입일때만 유효하다.

```ts
type T = Exclude<string | Date, string | number> // 타입은 Date
type NonZeroNums = Exclude<number, 0> // 타입은 number
```

# 8. 타입 공간과 값 공간의 심벌 구분하기

- 타입스크립트의 심벌(symbol)은 타입 공간이나 값 공간 중의 한곳에 존재한다. 따라서 이름이 같더라도 속하는 공간에 따라 다른 것을 나타낼 수 있다. 이를 잘 구분해야 한다.
- 따라서 같은 이름의 값으로 `instanceof`를 이용해 타입을 체크하는 등의 연산을 하는 경우, `instanceof`는 자바스크립트의 런타임 연산자이기 때문에 타입이 아닌 함수를 참조하는 이슈가 있을 수 있다.
- 타입 선언(`:`) 또는 단언문(`as`) 다음에 나오는 심벌은 타입인 반면, `=` 다음에 오는 모든 것은 값이다.
- `class`, `enum` 키워드는 값과 타입 두 가지로 모두 사용되기 때문에 주의한다.
- `typeof`, `this` 등의 연산자들과 키워드들은 타입 공간과 값 공간에서 다른 목적으로 사용될 때 상황에 따라 다르게 동작한다.
- 속성 접근자인 `[]`는 타입으로 쓰일 때도 동일하게 동작하지만, `obj['field']`와 `obj.field`는 값이 동일하더라도 타입은 다를 수 있기 때문에 반드시 `obj['field']`를 사용한다.

# 9. 타입 단언보다는 타입 선언을 사용하기

```ts
interface Person {
	name: string
}

const alice: Person = { name: 'Alice' } // 타입 선언
const mia: Person = { name: 'Mia' } // 타입 단언
```

- 타입 단언이 꼭 필요한 경우가 아니라면, 안정성 체크가 되는 타입 선언을 사용하자.
- 타입 단언은 타입 체커가 추론한 것보다 우리의 판단이 정확한 경우에 사용한다. ex) DOM 엘리먼트
- 단언문은 컴파일 중 제거되므로 non-null assertion은 값이 null이 아니라고 확신할 수 있을 때만 사용해야 한다.
  - non-null assertion(null 아님 단언문) = 접미사로 쓰인 `!(non-null assertion)`
  - `n!.toString()`

# 10. 객체 래퍼 타입 피하기

- 자바스크립트에서는 원시값에 메서드를 사용하기 위해 객체 래퍼 타입이 사용된다.
- 이 객체 래퍼 타입을 타입스크립트에서 사용하는 것은 지양하자.

# 11. 잉여 속성 체크의 한계 인지하기

- 잉여 속성 체크는 할당 가능 검사와는 별도의 과정이다.
- 잉여 속성 체크는 객체 리터럴인 경우에만 동작한다. 따라서 타입 단언문에도 적용되지 않는다.
- ```ts
  const n: Person = { color: 'blue' } // 'Person' 형식에 'color'이(가) 없습니다.

  const mia = { color: 'blue' } // 임시 변수
  const m: Person = mia // 정상 (객체 리터럴 x)

  const o = { color: 'blue' } as Person // 정상
  ```

# 12. 함수 표현식에 타입 적용하기

- 타입스크립트에서는 함수 표현식을 사용하는 것이 좋다.
- 함수의 매개변수부터 반환값까지 전체를 함수 타입으로 선언하여 함수 표현식에 재사용 가능하기 때문이다.
- ```ts
  type DiceRollFn = (sides: number) => number
  const rollDice: DiceRoolFn = (sides) => {
  	/*...*/
  }
  ```
- 동일한 타입 시그니처를 가지는 여러 함수를 작성할때는 함수 전체의 타입 선언을 적용하자.

# 13. 타입과 인터페이스의 차이점 알기

- 유니온 타입은 있지만, 유니온 인터페이스는 없다.
- 인터페이스는 타입을 확장할 수 있지만, 유니온은 할 수 없다.

### 타입 권장 케이스

- ```ts
  // 유니온 타입에 name 속성을 붙인 타입 -> 인터페이스로 불가능
  type NamedVariable = (Input | Output) & { name: string }
  ```
- ```ts
  // 튜플과 배열 타입도 type 키워드로 간결히 표현할 수 있다.
  type Pair = [number, number]
  type StringList = string[]
  type NamdedNums = [string, ...number[]]

  // 인터페이스로 구현
  interface Tuple {
  	0: number
  	1: number
  	length: 2
  }
  ```

### 인터페이스 권장 케이스

- ```ts
  // 속성을 확장하는 '선언 병합(declaration merging)'을 사용할 수 있다.
  interface State {
    name: string;
    capital: string;
  }
  interface State {
    population: number;
  }
  const wyoming: State {
    name: 'Wyoming',
    capital: 'Cheyenne',
    population: 500000,
  }
  ```

### 결론

- API에 대한 타입선언은 새로운 필드를 추가할 수 있는 interface를 권장한다.
- 복잡한 타입이라면 타입 별칭을 사용하는 것을 권장한다.
- 두 가지 모두 사용 가능한 케이스라면 프로젝트의 일관성, 그리고 보강의 관점에서 고려해보자.
