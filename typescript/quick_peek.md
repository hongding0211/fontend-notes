# Quick Peek

## Composing Types

```typescript
type MyBool = true | false
type WindowStates = "open" | "closed" | "minimized";
```

## 泛型 Generics

```typescript
type StringArray = Array<string>;
```

```typescript
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}
 
// This line is a shortcut to tell TypeScript there is a
// constant called `backpack`, and to not worry about where it came from.
declare const backpack: Backpack<string>;
 
// object is a string, because we declared it above as the variable part of Backpack.
const object = backpack.get();
```

## Typescript 编译器

### install

```bash
npm install -g typescript

tsc xxx.ts // basic usage
```

### Emitting with errors

默认情况下，即便 tsc 检查到错误，仍然会输出一个 js 文件。但是如果想要保证在没有错误的情况下才输出 js 文件的话，添加 `--noEmitOnError` 参数

```bash
tsc --noEmitOnError xxx.ts
```

### Downleveling

默认情况下，tsc 编译的目标是 `ES3` ，通过 `--target` 指定编译目标

```bash
tsc --target es2015 xxx.ts
```

## Strictness flags

多数用户对于 Typescript 最初的要求即：可以验证部分代码的准确性以及优雅的代码提示。这也是 ts 默认提供的编程体验。例如，默认情况下，ts 不检查潜在的 null / undefined 值。但是利用 ts 的优势，这些问题是可以避免的，因此牺牲部分编程体验来换的更好的代码健壮性是合理的，特别是在大型项目或者长期维护的项目中。

ts 有多个 type-checking 的 strictness flag，可以在 tsconfig.json 设置 `"strict": true` 来启用所有的 flag，也可以启用其中的部分。下面两个是最常用的 `strictness flags`

### noImplicityAny

不允许有会被推断出 any 的类型。如下面的 foo 方法中的 bar 参数，当没有指明 bar 的类型时，**ts 会自动推断它为 `any` ，这是不允许的。**

```typescript
function foo(bar){
  ...
}
```

### strictNullChecks

When `strictNullChecks` is `true`, `null` and `undefined` have their own distinct types and you’ll get a type error if you try to use them where a concrete value is expected.

```typescript
const loggedInUser = users.find((u) => u.name === loggedInUsername);
console.log(loggedInUser.age);	// find 的结果可能为 null，这里直接访问，会报错
```
