# Narrowing

凭借 TypeScript ，它可以对许多类型进行自动 narrowing，以此来更好的辅助 coding

## typeof

`typeof` 返回以下 8 种类型。

* `"string"`
* `"number"`
* `"bigint"`
* `"boolean"`
* `"symbol"`
* `"undefined"`
* `"object"`
* `"function"`

## Truthiness

> 在 JavaScript 中，Array, null 都是 object 类型

```js
typeof null
> 'object'
```

因此，如果要通过 `typeof` 来判断一个变量是否为 array 可能会带来意想不到的问题，考虑以下代码：

```typescript
function foo(str: string[] | null) {
  if(typeof str === 'object'){
    // 报错，因为 s 可能是 array，但也可能是 null！
    for(const s of str){
      ...
    }
  }
}
```

如果使用 JavaScript 编写，上述问题就不能被正确发现，好在 TypeScript 会帮你指出这一问题，因此正确的解决方法是:

```typescript
function foo(str: string[] | null) {
  if(str && typeof str === 'object'){
    // ok, 先进行了 truthiness 判断
    for(const s of str){
      ...
    }
  }
}
```

## ==, !=

Typescript 在判断 `x != undefined` 的时候，会同时判断 `x != null`，反之依然（`x == undefined`)；同时，在判断 `x != null` 的时候，也会同时判断 `x != undefined` ，反之亦然。

```typescript
function foo(x: number | null | undefined) {
  if (x != null) {
    // x != null 实际上过滤掉了 null 和 undefined 两种情况
    // 因此 x 一定是 number
    console.log(x.toFixed(2))
  }
}
```

## 自定义类型保护

我们可以自己封装一个判断函数，来判断某一变量是否为指定的类型

```typescript
function isString(test: string | number): test is string {
  return typeof test === 'string'
}

function foo(bar: any){
  if(isString(bar)){
    console.log("it's a string")
    console.log(bar.toFixed(2))		// 会在编译时报错，因为 bar 是 string
  }
}
```

使用 `<var> is <type>` 谓词作为这个判断函数的返回值，ts 会在 if 逻辑后 narrow 它的类型

![image-20211226181638088](../imgs/image-20211226181638088.png)

bar 已经从 `any` narrow 到 `string` 了，因此会直接报错

那么为什么不直接返回一个 `boolean` 呢？

```typescript
function isString(test: string | number): boolean {
  return typeof test === 'string'
}

function foo(bar: any){
  if(isString(bar)){
    console.log("it's a string")
    console.log(bar.toFixed(2))		// 编译时不会报错，而 runtime 时会报错
  }
}
```

TypeScript 的 narrowing 机制需要借助 `is` 来实现，因此直接返回一个 `boolean` 并不可行

![image-20211226181743033](../imgs/image-20211226181743033.png)

在 if 逻辑后，ts 并不能正确地 narrow bar 的属性，它仍然是一个 `any`

## Discriminated unions

定义两种 Shape，Circle 和 Square，它们都有一个 `kind` 属性

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}

function foo(shape: Circle| Square){
  if(shape.kind === 'circle'){
    // 这时候 shape 已经被 narrow 到 Circle
    // 因此只能访问 radius 属性
    console.log(shape.radius)
  }
  ...
}
```

那么在 if 逻辑中对 kind 进行判断，相当于进行了 narrow

也可以使用 switch 语句对 kind 进行遍历

## never type

如果 TypeScript 不断地 narrowing，直到没有一个类型都没有剩下，那这时候它就是 `never` 类型

```typescript
function getArea(shape: Circle | Square) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      // 这时候 shape 就是被 narrow 到 never
      const _exhaustiveCheck: never = shape;		
      return _exhaustiveCheck;
  }
}
```
