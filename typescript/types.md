# Types in TypeScript

## Object Types

在函数声明时，可以指定传入 object 的类型

```typescript
function foo (obj: { x: number, y: number }){
  ...
}
```

指定其中某些参数是 `optional` 的

```typescript
function foo (obj: { x: number, y?: number }){
  ...
}
// both ok
foo({x: 1, y: 1})
foo({x: 1})
```

不过，如果某一参数是 `optional` 的，那么就应该在函数体内部对它进行判空

```typescript
function foo (obj: { x: number, y?: number }){
  if(obj.y !== undefined){
    ...
  }
}
```

## Union Types

可以由多个现有的 type 构建新的 type

```typescript
function foo(id: number | string) {...}

foo(1)		// allowed
foo(true) // not allowed
```

foo接受的 id 就是一个 `union type` ，他可以是一个 number 或者 string

与上面的 `optional` 类似，如果在函数中定义了一个参数为 `unino type`，那么在函数体内可能需要 `narrowing`

```typescript
function foo(id: number | string) {
  // not allowed，因为 id 可能是 number，number 没有 toUpperCase() 属性
  id.toUpperCase()
  // narrowing，allowed
  if(typeof id === 'string')
    id.toUpperCase()
}
```

## type 关键字

类似于其他语言的 `typedefine`，可以将上述的 Object Types、Union Types 定义成一个新的”类型“

```typescript
type Point = {
  x: number,
  y: number
}
type ID = number | string

// useage
function foo(point: Point)
function foo(id: ID)
```

> 注意的是，type 定义的新类型其实就是一个“别名” **(type aliases)**
>
> 如 type NewString = string，本质上 NewString 就是 string 类型，它们不是两个不同的类型

## interface 关键字

`interfaces` 关键字是另一种给 Object Types 定义”类型“的方式

```typescript
interface Point = {
  x: number,
  y: number
}
```

## type aliases 和 interface 的区别

两者在大部分情况下没有区别，可以随意选择使用。

> 在多数情况下，可以按照自己的偏好选择 type 或者 interface，不过有一个不错的策略是：
>
> **尽量使用 interface，除非必须使用 type 的时候**

关键的区别就是：`interfaces`**是可以以增加新的属性的**

```typescript
interface Animal {
  name: string
}
// interface 创建后可以继续添加属性
interface Aniaml {
  age: number
}

let cat: Cat
cat.name = 'xxx'
cat.type = 'xxx'
```

```typescript
type Animal = {
  name: string
}
// type 创建后就不能修改
type Aniaml = {
  age: number
}
```

### interface 增加属性

```typescript
interface Animal {
  name: string
}
interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
```

### type 增加属性

```typescript
type Animal = {
  name: string
}
type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;
```

## type 断言

有时候，你能确定一个值的类型，那么就可以使用断言，例如你确定 id 为 'main\_canvas' 的 DOM 元素一定是 HTMLCanvasElement 类型

```typescript
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement
```

另一种用`<>`的写法

```typescript
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas")
```

> 类型断言在编译后会被移除，所以不必担心会出现 runtime 时问题

断言只能**缩小**或者**扩大**某一个类型，如下面的例子是不合法的

```typescript
const foo = 'bar' as number		// not allowed
```

但是上述这一个限制有时候也有点太妨碍了，可以用下面的方法来解决

```typescript
const foo = (bar as any) as T		// 先转为 any 再转为想要的类型
```

## 字面量 type

在 Typescript 中，下面两种写法是等价的

```typescript
const foo = 'bar'
const foo: 'bar' = 'bar'

foo = 'foorbar'		// not allowed
```

不过上面的第二种写法没有太大意义，但是利用`union type`，可以限制某一变量的取值范围

```typescript
const foo: 'foo' | 'bar' = 'bar'
foo = 'foo'			// ok
foo = 'foobar'	// not ok
```

> union type 不仅仅可以组合相同类型的 type，下面的写法也是可以的

```typescript
type Foo = 'bar' | number

let foo: Foo = 1		// ok
foo = 'bar'					// ok
foo = 'foobar'			// not ok
```

## null & undefined

TypeScript 和 JavaScript 一样，也有 null 和 undefined。但是它们的行为取决于 `strictNullChecks` flag 是否开启

### strictNullChecks off

null 和 undefined 仍然可以被正常访问，也能当做属性赋值给任意值。这和 C# Java 类似，并不做 null checks。

### strictNullChecks on

> null undefined 是大多数 bug 的来源，因此推荐打开 `strictNullChecks`

当 `strictNullChecks` 打开时，需要在访问可能为空的值时，提前判空

### ! 断言

如果能 100% 保证变量不会为空，可以使用 `!` 断言，以省去判空逻辑

```typescript
function foo(x: number){
  console.log(x!.toFixed(2))
}
```

## Enums 枚举类型

Enums 分为数字型和字符串型

**数字型**

```typescript
// 如果不指定，默认从 0 开始
enum Direction {
  Up = 1, Down, Left, Right
}
```

**字符串型**

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
```

> Enums 并不是 TypeScript 对 JavaScript 在\*\*“类型系统层面”\*\*上的扩展，因为 JavaScript 本身并没有枚举类型。如上面的数字型 enum 在 tsc 编译后的 js 文件是这样的：

```javascript
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 1] = "Up";
    Direction[Direction["Down"] = 2] = "Down";
    Direction[Direction["Left"] = 3] = "Left";
    Direction[Direction["Right"] = 4] = "Right";
})(Direction || (Direction = {}));
```
