# Functions in TypeScript

## 函数 Type Expression

Function type expression 是描述一个函数最简单的方法：

使用类似箭头函数的语法，来标识一个函数的传入参数类型及返回值类型

```typescript
(<para1>: <type>) => <type>
```

如，把一个函数作为参数传入至另一个函数中

```typescript
// callback 函数接受一个 string 类型变量，并且返回 void
function foo(callback: (msg: string) => void) {
  callback('bar')
}

foo((msg: string)=>console.log(msg))	// output: bar
```

当然也可以使用 `type` 关键字来定义函数 type expression

```typescript
type CallbackFn = (msg: string) => void
function foo(callback: CallbackFn){
  ...
}
```

## Call Signatures

JavaScript 函数是一等公民，因此函数也可以有其他属性

```typescript
type FooFn = {
  discription: string,			// 其他属性
  (param: number): boolean	// 注意，不是 => 而是 : 来标注返回值
}

function foo(fooFn: FooFn) {
  console.log(fooFn.discription, fooFn(0))
}
```

## 泛型

下列函数接受一个 `T[]` 类型数组，并且返回 `T` 或者 `undefined`

```typescript
function foo<T>(arr: T[]): T | undefined {
  return arr[0]
}

const n = foo([1,2,3])								// n 是 number
const s = foo<string>(['a','b',3])		// 报错，因为规定了传入的必须是 string[]
```
