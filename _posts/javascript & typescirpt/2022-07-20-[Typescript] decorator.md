---
title: "[Typescript] decorater"
tags: typescript
categories:
  - javascript-typescript
---

## 데코레이터

데코레이터는 클래스 선언, 메서드, 접근자, 프로퍼티, 매개변수에 추가할 수 있습니다. 아래는 클래스 데코레이터의 예시입니다.

```ts
function Logger(cons: Function) {
  console.log("Logging..");
}

@Logger
class Person {
  name = "MAX";

  constructor() {
    console.log("Creating person object");
  }
}

const pers = new Person();
console.log(pers);
```

출력결과는 아래와 같습니다.

![image](https://user-images.githubusercontent.com/67682840/179465121-925cccf4-45da-4bd1-819c-f08e17c7a5ad.png)

실행순서를 살펴보면 객체가 생성되기 전에 `@Logger`의 출력을 확인할 수 있습니다. 이를 통해 아래와 같은 사실을 알 수 있습니다.

> 데코레이터는 클래스가 정의될 때 실행되며 객체가 생성될 때 실행되는 것이 아니다

<br>

## 데코레이터 팩토리

데코레이터를 일반화하기 위하여 파라미터를 사용하고 싶다면 데코레이터 팩토리를 사용할 수 있습니다. 데코레이터 팩토리는 데코레이터 구현에 대한 책임을 리턴하는 함수에게 전달합니다. 즉, 데코레이터 함수를 리턴하며 데코레이터 팩토리는 데코레이터를 감싸는 Wrapper 함수라고 할 수 있습니다.

```ts
// decorator factory
function Logger(message: string) {
  //decorator function
  return function (constructor: Function) {
    console.log(message);
  };
}

@Logger("Customized Logging...")
class Person {
  name = "MAX";

  constructor() {
    console.log("Creating person object");
  }
}

const pers = new Person();
console.log(pers);
```

실행결과는 아래와 같습니다.

![image](https://user-images.githubusercontent.com/67682840/179512190-18daafef-9953-4be0-b4db-706ad914b8bb.png)

<br>

## 클래스 데코레이터

클래스 데코레이터는 하나의 파라미터를 전달받습니다.

- 클래스의 constructor

만일 리턴값이 있다면 클래스에서 해당 값을 클래스 선언에서 대체하게 됩니다. 첫번째 예시는 클래스 데코레이터에서 `toString` 메서드를 리턴함으로써 override한 예시입니다.

```ts
function toString<T extends { new (...args: any[]): any }>(BaseClass: T) {
  return class extends BaseClass {
    toString() {
      return "custom toString " + JSON.stringify(this);
    }
  };
}

@toString
class C {
  public foo = "foo";
  public num = 24;
}

console.log(new C().toString());
```

출력결과는 아래와 같습니다.

![image](https://user-images.githubusercontent.com/67682840/179659762-1bd1b129-2906-4e3a-8f88-20f59b9288ec.png)

`extend BaseClass`
원래 class의 properties들을 유지하기 위함입니다. `foo`와 `bar`가 이에 해당합니다.

`<T extends { new (...args: any[]): any }>`
여러개의 `any`를 파라미터로 하고 `any`를 리턴하는 함수를 constructor로 사용함을 의미합니다.
new keyword는 해당 함수가 class constructor로 사용됨을 의미합니다.

두 번째 예시는 기존에 정의된 constructor를 override하는 예시입니다.

```ts
function LogConstructor<T extends { new (...args: any[]): any }>(BaseClass: T) {
  return class extends BaseClass {
    constructor(...args) {
      super();
      console.log("Logging Constructor...");
    }
  };
}

@LogConstructor
class C {
  public foo = "foo";
  public num = 24;
}

let obj = new C();
```

`super()`을 통해 새로운 constructor안에서 기존 constructor을 호출한 모습입니다.

<br>

## 프로퍼티 데코레이터

프로퍼티 데코레이터는 아래의 두가지 파라미터를 전달 받습니다.

- 첫 번째
  - static member라면 클래스의 생성자 함수
  - instance member라면 prototype
- 두 번째
  - property의 이름

프로퍼티 데코레이터를 활용해서 프로퍼티를 재정의할 수 있습니다. 아래 예시는 allowList에 있는 string만 프로퍼티에 허용되는 데코레이터의 예시입니다.

```ts
const allowlist = ["Jon", "Jane"];

const allowlistOnly = (target: any, memberName: string) => {
  let currentValue: any = target[memberName];

  Object.defineProperty(target, memberName, {
    set: (newValue: any) => {
      if (!allowlist.includes(newValue)) {
        return;
      }
      currentValue = newValue;
    },
    get: () => currentValue,
  });
};

class Person {
  @allowlistOnly
  name: string = "Jon";
}

const person = new Person();
console.log(person.name);

person.name = "Peter";
console.log(person.name);

person.name = "Jane";
console.log(person.name);
```

```
실행결과
> Jon
> Jon
> Jane
```

<br>

## 메서드 데코레이터

메서드 데코레이터는 세가지 파라미터를 전달받습니다.

- 첫번째
  - static member라면 클래스의 생성자 함수
  - instance member라면 prototype
- 두번째
  - property의 이름
- 세번째
  - member에 대한 property descriptor

아래는 메서드 데코레이터의 예시입니다.

```ts
function Log3(
  target: any,
  propertyName: string | Symbol,
  descriptor: PropertyDescriptor
) {
  console.log("method decorator");
  console.log(target);
  console.log(propertyName);
  console.log(descriptor);
}

class Product {
  static title: string;
  private _price: number;

  set price(val: number) {
    if (val > 0) {
      this._price = val;
    } else {
      throw new Error("Invalid price");
    }
  }

  constructor(t: string, p: number) {
    this._price = p;
  }

  @Log3
  getPriceWithTax(tax: number) {
    return this._price * (1 + tax);
  }
}
```

출력결과는 아래와 같습니다. 빨간색 사각형을 한 부분은 접근자 데코레이터와 비교할 때 다시 살펴보도록 하겠습니다.

![image](https://user-images.githubusercontent.com/67682840/179648081-9c1358c4-1152-4dc4-94b4-dfebaf06154f.png)

프로퍼티 데코레이터와 다른부분은 `descriptor` 파라미터입니다. 이를 통해 원래 메서드의 구현을 override하거나 원하는 로직을 넣을 수 있습니다. 다음은 `add`를 호출할 때 input과 output에 대해 로그를 출력하는 메서드 데코레이터의 예시입니다.

```ts
function logger(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const original = descriptor.value;

  descriptor.value = function (...args) {
    console.log("params: ", ...args);
    const result = original.call(this, ...args);
    console.log("result: ", result);
    return result;
  };
}

class C {
  @logger
  add(x: number, y: number) {
    return x + y;
  }
}

const c = new C();
c.add(1, 2);
```

실행결과는 아래와 같습니다.

![image](https://user-images.githubusercontent.com/67682840/179649145-a59b774c-9413-4c37-8d0a-1f25cd670155.png)

<br>

## 접근자 데코레이터

접근자 데코레이터는 메서드 데코레이터와 일반적으로는 동일합니다. 다른 점은 `descriptor`에 있는 key입니다. 아래는 `@Log3`의 위치가 `set` 메서드(또는 `get`)에 적용된 점만을 제외하면 메서드 데코레이터의 첫번째 예시와 동일한 예시입니다.

```ts
function Log3(
  target: any,
  propertyName: string | Symbol,
  descriptor: PropertyDescriptor
) {
  console.log("accessor decorator");
  console.log(target);
  console.log(propertyName);
  console.log(descriptor);
}

class Product {
  static title: string;
  private _price: number;

  @Log3
  set price(val: number) {
    if (val > 0) {
      this._price = val;
    } else {
      throw new Error("Invalid price");
    }
  }

  constructor(t: string, p: number) {
    this._price = p;
  }

  getPriceWithTax(tax: number) {
    return this._price * (1 + tax);
  }
}
```

실행결과는 아래와 같습니다.

![image](https://user-images.githubusercontent.com/67682840/179653573-e5911968-0db7-43a1-8c41-0f2ba7a18c81.png)

메서드 데코레이터와 다른점은 빨간색 사각형에서 찾아 볼 수 있음을 확인할 수 있습니다. 메서드 데코레이터의 `descriptor` key는 아래와 같습니다

- `value`
- `writable`
- `enumerable`
- `configurable`

접근자 데코레이터의 `descriptor` key는 아래와 같습니다

- `get`
- `set`
- `enumerable`
- `configurable`

<br>

두 번째로 활용할 수 있는 예시는 `@immutable` 접근자 데코레이터를 통해 mutable한 객체를 immutable하게 만드는 예시입니다. javascript에서 `Object`의 변수는 객체에 대한 참조를 가리키기 때문에 메모리에 직접 접근하여 수정할 수 있습니다. 이를 immutable하게 만들기 위해서 스프레드 연산자(`{...variable}`)를 활용한 코드입니다.

```ts
function immutable(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const original = descriptor.set;

  descriptor.set = function (value: any) {
    return original.call(this, { ...value });
  };
}

class C {
  private _point = { x: 0, y: 0 };

  @immutable
  set point(value: { x: number; y: number }) {
    this._point = value;
  }

  get point() {
    return this._point;
  }
}

const c = new C();
const point = { x: 1, y: 1 };
c.point = point;

console.log(c.point === point);
// -> false
```

<br>

#### 참고

- https://typescript-kr.github.io/pages/decorators.html
- https://haeguri.github.io/2019/08/25/typescript-decorator/
- https://saul-mirone.github.io/a-complete-guide-to-typescript-decorator/
- https://poiemaweb.com/js-immutability
- https://stackoverflow.com/questions/50726326/how-to-go-about-understanding-the-type-new-args-any-any
