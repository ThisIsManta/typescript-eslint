# Require explicit return types on functions and class methods (explicit-function-return-type)

Explicit types for function return values makes it clear to any calling code what type is returned.
This ensures that the return value is assigned to a variable of the correct type; or in the case
where there is no return value, that the calling code doesn't try to use the undefined value when it
shouldn't.

## Rule Details

This rule aims to ensure that the values returned from functions are of the expected type.

The following patterns are considered warnings:

```ts
// Should indicate that no value is returned (void)
function test() {
  return;
}

// Should indicate that a number is returned
var fn = function() {
  return 1;
};

// Should indicate that a string is returned
var arrowFn = () => 'test';

class Test {
  // Should indicate that no value is returned (void)
  method() {
    return;
  }
}
```

The following patterns are not warnings:

```ts
// No return value should be expected (void)
function test(): void {
  return;
}

// A return value of type number
var fn = function(): number {
  return 1;
};

// A return value of type string
var arrowFn = (): string => 'test';

class Test {
  // No return value should be expected (void)
  method(): void {
    return;
  }
}
```

## Options

The rule accepts an options object with the following properties:

```ts
type Options = {
  // if true, only functions which are part of a declaration will be checked
  allowExpressions?: boolean;
  // if true, type annotations are also allowed on the variable of a function expression rather than on the function directly
  allowTypedFunctionExpressions?: boolean;
  // if true, functions immediately returning another function expression will not be checked
  allowHigherOrderFunctions?: boolean;
};

const defaults = {
  allowExpressions: false,
  allowTypedFunctionExpressions: true,
  allowHigherOrderFunctions: true,
};
```

### Configuring in a mixed JS/TS codebase

If you are working on a codebase within which you lint non-TypeScript code (i.e. `.js`/`.jsx`), you should ensure that you should use [ESLint `overrides`](https://eslint.org/docs/user-guide/configuring#disabling-rules-only-for-a-group-of-files) to only enable the rule on `.ts`/`.tsx` files. If you don't, then you will get unfixable lint errors reported within `.js`/`.jsx` files.

```jsonc
{
  "rules": {
    // disable the rule for all files
    "@typescript-eslint/explicit-function-return-type": "off"
  },
  "overrides": [
    {
      // enable the rule specifically for TypeScript files
      "files": ["*.ts", "*.tsx"],
      "rules": {
        "@typescript-eslint/explicit-function-return-type": ["error"]
      }
    }
  ]
}
```

### allowExpressions

Examples of **incorrect** code for this rule with `{ allowExpressions: true }`:

```ts
function test() {}

const fn = () => {};

export default () => {};
```

Examples of **correct** code for this rule with `{ allowExpressions: true }`:

```ts
node.addEventListener('click', () => {});

node.addEventListener('click', function() {});

const foo = arr.map(i => i * i);
```

### allowTypedFunctionExpressions

Examples of **incorrect** code for this rule with `{ allowTypedFunctionExpressions: true }`:

```ts
let arrowFn = () => 'test';

let funcExpr = function() {
  return 'test';
};

let objectProp = {
  foo: () => 1,
};
```

Examples of additional **correct** code for this rule with `{ allowTypedFunctionExpressions: true }`:

```ts
type FuncType = () => string;

let arrowFn: FuncType = () => 'test';

let funcExpr: FuncType = function() {
  return 'test';
};

let asTyped = (() => '') as () => string;
let castTyped = <() => string>(() => '');

interface ObjectType {
  foo(): number;
}
let objectProp: ObjectType = {
  foo: () => 1,
};
let objectPropAs = {
  foo: () => 1,
} as ObjectType;
let objectPropCast = <ObjectType>{
  foo: () => 1,
};

declare functionWithArg(arg: () => number);
functionWithArg(() => 1);

declare functionWithObjectArg(arg: { method: () => number });
functionWithObjectArg({
  method() {
    return 1;
  },
});
```

### allowHigherOrderFunctions

Examples of **incorrect** code for this rule with `{ allowHigherOrderFunctions: true }`:

```ts
var arrowFn = () => () => {};

function fn() {
  return function() {};
}
```

Examples of **correct** code for this rule with `{ allowHigherOrderFunctions: true }`:

```ts
var arrowFn = () => (): void => {};

function fn() {
  return function(): void {};
}
```

### allowZeroOrSingleReturnStatement

This option allows you to avoid writing verbose return types only if there is zero or single non-`undefined` return statement.

Examples of **incorrect** code for this rule with `{ allowZeroOrSingleReturnStatement: true }`:

```ts
// Missing a return type because there is more than one non-undefined return statements
function fn(a: boolean) {
  if (a) return 1;
  return Math.random();
}
```

Examples of **correct** code for this rule with `{ allowZeroOrSingleReturnStatement: true }`:

```ts
// Do not have to specify `void` return type because there is zero return statements
function fn() {}
```

```ts
// Do not have to specify `number | undefined` because `undefined` does not change the primary return type
function fn(a: boolean, b: boolean, c: boolean) {
  if (a) return;
  if (b) return undefined;
  if (c) return void 0;
  return Math.random();
}
```

```ts
// Need to specify the return type because there are more than one return types.
function fn(a: boolean): number | 'invalid' {
  if (a) return 'invalid';
  return Math.random();
}
```

## When Not To Use It

If you don't wish to prevent calling code from using function return values in unexpected ways, then
you will not need this rule.

## Further Reading

- TypeScript [Functions](https://www.typescriptlang.org/docs/handbook/functions.html#function-types)
