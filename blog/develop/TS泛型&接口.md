---
slug: interface_<T>
title: TS泛型&接口
date: 2023-12-21
authors: kuizuo
tags: []
keywords: []
---

泛型是一种在定义函数、接口、或类时不预先指定类型。

而是在使用时再指定类型的方法。

泛型通常用尖括号 `<>` 包裹，比如 `<T>`

它允许你编写可以处理多种数据类型的代码

而不需要为每种数据类型编写不同的代码

它提供了一种在编译时创建可重用代码的方法

## 基本语法

```tsx
function identity<T>(arg: T): T {
 return arg;
}

identity<string>("ntscshen");
identity<number>(123);
```

这是一个普通函数，加了泛型之后的效果。我们先看调用部分、再看定义部分

1. **函数调用**：如果是个普通函数调用 `identity("ntscshen");` 我们都会。怎么改成`TS`泛型函数？`identity<string>("ntscshen");` 在普通函数的函数名后面加个 `<>` 特殊标识即可。普通函数 `"ntscshen"` 是函数实参，`string` 是泛型函数的泛型实参
2. **函数定义**：普通函数定义大伙都会， `funciton identity(arg) { return arg; }` ，既然调用的时候已经传递了泛型参数，那么定义时候泛型参数也要加上。`indentity<T>(arg)` 书写的位置和调用的位置一致，紧跟在函数名后面。

在定义泛型函数时候，泛型参数主要用在下面三个地方

1. **函数名之后**：用于声明传递进来的泛型参数
2. **参数类型**：指定函数形参的类型
3. **返回类型**：指定函数返回值的类型

除了这三个必写的地方，参数另一个重要地方是写在函数体内

再一次解释一下，TS理解的过程。

1. 第一个 `<T>` ，告诉TS， `identity` 是一个泛型函数，`<T>` 是一个占位符类型，类比函数参数的形参，它将在函数被调用时被确定是什么类型
2. 参数 `arg: T` ，表示 `arg` 类型是 `T`，由于 `T` 是泛型，意味着 `arg` 可以是任何类型。任何调用 `identity` 传递进来的值。`identity<string>`、`identity<number>`、 `identity<boolean>` `…` 当你传递任何类型的值给它，`T` 将会被替换为该值的类型。对比函数本身的实参和形参。
3. 返回类型`T`：返回值的类型将于传入参数 `arg` 的类型相同

**箭头函数**

```tsx
const identity = <T>(arg: T): T => {
 return arg;
}
```

普通泛型函数和箭头泛型函数在 TypeScript 中主要的书写区别在于它们的语法结构。功能上，它们都能实现相同的泛型逻辑。

从这两个例子看，它们书写的共同点都在**小括号()**参数之前获取传递机那里的泛型参数。使用和普通函数一致

## 类型约束

`TS`中的非常重要的特性，它允许你指定一个泛型参数必须满足的约束条件。

通过类型**约束，**可以限制泛型类型的能力。确保它具有我们需要的特定属性和方法

例如：如果你正在编写一个处理数组的泛型函数，使用 **`extends Array<any>`** 可以确保传递给该函数的参数确实是数组，从而安全地使用数组的方法和属性。

- 约束类型通常使用 `extends` 关键字来实现。

    在泛型约束的上下文中，`extends` 翻译成 “**扩展**”或“**约束**”，它被用来指定一个类型参数必须符合某种形式或结构。这种用法与类继承中的“继承”含义不同，更多地关注于类型的兼容性和约束。

    **泛型约束**：**`extends`** 在泛型约束中表示泛型类型参数必须是指定类型的子类型。

    **约束表达**：当你看到 **`T extends SomeType`** 这样的表达式时，它的意思是泛型类型 **`T`** 必须是 **`SomeType`** 类型或其子类型。

### 确保属性存在

使用泛型约束来确保这些属性的存在

```tsx
interface HasId {
 id: number;
}

function findById<T extends HasId>(items: T[], id: number): T {
 return items.find(item => item.id === id);
}
```

通过泛型约束 `T extends HasId` ，`TS`确保传递给函数的 `items` 数组中的每个元素都包含`id`属性

### 使用接口作为类型约束

处理复杂结构的对象时候，定义一个接口来描述当前结构，然后将这个接口用作泛型约束。

```tsx
interface Respondable {
 response: string
}

const processResponse = <T extends Respondable>(data: T) => {
 console.log(data.response);
}
```

**`processResponse`** 函数可以接受任何具有 **`response`** 属性的对象，无论这个对象的其他属性是什么。

### 确保类型具有某个方法

如果需要在泛型参数上调用某个方法，可以约束这个参数必须有该方法

```tsx
interface Printable {
 print(): void;
}

const printAll = <T extends Printable>(items: T[]) => {
 items.forEach(item => item.print());
}
```

### 限制函数参数为某些类型

限制泛型函数只接收某些类型的参数

```tsx
function processValue<T extends string | number>(value: T) {
 // ...
}
```

### 使用泛型创建可重用的工具函数

```tsx
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
 return obj[key];
}

const obj = { a: 0, b: 1 };
const value1 = getProperty(obj, 'a'); // √ 
const value2 = getProperty(obj, 'c'); // Cannot find name 'c'. (2304)
```

`keyof` keyof运算符接收一个对象类型，并生成其键的字符串或数字字面量联合

```tsx
type Staff = {
 name: string;
 salary: number;
}
type staffKeys = keyof Staff; // "name" | "salary"
```

`getProperty` 函数接受一个对象和一个键名，安全地返回对应的值。

`T` 是对象

`keyof T` 是对象`T` 的`key`值集合

`K extends keyof T` **K被约束在，T的所有Key值集合内**

## ****构造器签名****

```tsx
interface Type<T = any> extends Function {
 new (...args: any): T;
}

forRepository<T extends Type<any>>(
 repositories: T[]
) {
 // ...
}

forRepository([PostEntity]); // PostEntity是一个NestJS中的实体类
```

`<T extends Type<any>` 按照上面学的，`T` 必须满足 `Type` 的约束条件。

`Type` 是什么？Type是一个接口，并且使用到了接口中的 **泛型**和**继承**，接口详情下面有扩展

**`(...args: any)` ：意味着可以传递任意数量、任意类型的参数**

`new (): T` ：构造器签名核心。一种特殊的类型注解。通常包含以下几个部分

1. `new` 关键字是构造器签名的核心，描述一个构造函数
2. **参数列表**(…args: any)：定义了构造函数可以接受的参数
3. **返回类型：**返回的通常是构造函数创建的实例的类型。在这里面通常是泛型的参数 **`T`**

**`Type<T>` 接口定义了一个构造函数签名。任何符合 `Type<T>` 类型的对象都必须有一个可以使用 `new` 运算符和任意参数调用的构造函数。**并且这个构造函数会创建一个 **`T`** 类型的对象。

构造器签名是 `TypeScript` 中用于描述可以被实例化的对象类型的一种方式。

构造器签名是 `TypeScript` 中用于描述可以被实例化的对象类型的一种方式。

构造器签名是 `TypeScript` 中用于描述可以被实例化的对象类型的一种方式。

### 接口 `interface`

当你需要**定义对象**的形状或者创建一个**可扩展的类型系统**时，使用接口。

1. **基本用法**

定义了 `Person` 接口，要求使用它的对象必须拥有 `name`和`age` 两个属性

```tsx
interface Person {
 name: string;
 age: number;
}

let person: Person = { name: "ntscshen", age: 30 };
```

1. **定义类的行为**

在 TS 中，接口（`interface`）可以用来定义一个类应该遵循的结构和行为。

你可以规定一个类必须拥有那些属性和方法，以及它们的类型。这可以确保类的实例符合特定的形状

```tsx
interface Person {
 name: string;
 age: number:
 greet(phrase: string): void;
}

class Developer implements Person {
 name: string;
 age: number;

 constructor(name: string, age: number) {
      this.name = name;
      this.age = age;
  }

  greet(phrase: string) {
      console.log(phrase + ' ' + this.name);
  }
}
```

**`Person`** 接口定义了一个 **`name`** 和 **`age`** 属性，和一个 **`greet`** 方法。

**`Developer`** 类通过实现 **`Person`** 接口，承诺提供接口中定义的所有属性和方法。

`**implements**` **关键字**确保一个类满足一个或多个特定的接口契约。这意味着该类必须实现接口中定义的所有属性和方法。在使用 **`implements`** 时，TypeScript 会进行类型检查，确保类正确实现了其声明要实现的接口。

**`implements`** 在中文中通常被翻译为“实现”。在编程的上下文中，这个词用于描述类对接口的实现关系。

1. **接口继承**

接口的继承是一种强大的特性，允许你创建一个接口，它继承自一个或多个其他接口

它会继承父接口的所有成员（属性和方法）。这类似于类的继承，但只涉及类型的声明，不涉及实现。

```tsx
interface Person {
    name: string;
    age: number;
}

interface Employee extends Person {
    employeeId: number;
    department: string;
}
```

**`Employee`** 接口继承了 **`Person`** 接口的所有成员，并添加了两个新的属性：**`employeeId`** 和 **`department`**。

```tsx
interface Contactable {
    email: string;
    phone: string;
}

interface Addressable {
    address: string;
}

interface Employee extends Person, Contactable, Addressable {
    employeeId: number;
    department: string;
}
```

**`Employee`** 接口现在继承了 **`Person`**、**`Contactable`** 和 **`Addressable`** 接口。因此，它包含了所有这些接口的成员：**`name`**、**`age`**、**`email`**、**`phone`** 和 **`address`**，以及它自己的 **`employeeId`** 和 **`department`** 属性。

1. **泛型接口**

泛型接口与普通接口类似，但它们接受一个或多个泛型参数。这些参数可以用于定义接口中属性或方法的类型。

```tsx
interface Response<T> {
 status: number;
  message: string;
 data: T;
}
```

**`Response<T>`** 是一个泛型接口，其中 **`T`** 是一个泛型参数。**`data`** 属性的类型被指定为 **`T`**，这意味着 **`data`** 可以是任何类型。

举个实际场景的例子：

```tsx
interface User {
    id: number;
    name: string;
}

interface Product {
    id: number;
    title: string;
    price: number;
}

const userResponse: Response<User> = {
 status: 200,
 message: "Success",
 data: { id: 1, name: "ntscshen" }
}

const productResponse: Response<Product> = {
   status: 200,
    message: "Success",
    data: { id: 2, title: "Laptop", price: 980 }
}
```

使用 `Response<T>` 来表示通用的data响应内容。

将 `User、Product` 传递给 `Response<T>` ，分别响应两种不同的类型。类型的函数封装。
