---
title: Variable Declaration
layout: docs
permalink: /docs/handbook/variable-declarations.html
oneline: How TypeScript handles variable declaration
translatable: true
---

`let`과 `const`는 JavaScript의 변수 선언에 대한 비교적 새로운 두 가지 개념입니다. [앞서 언급했듯이]((/docs/handbook/basic-types.html#a-note-about-let)) `let`은 어떤 면에서 `var`와 비슷하지만 사용자가 JavaScript에서 마주치는 흔한 "gotchas"를 방지 할 수 있습니다.

`const`는 변수에 대한 재할당을 방지한다는 점에서`let`을 확장한 것입니다.

TypeScript가 JavaScript의 확장이므로 TypeScript는 자연스럽게 `let`과`const`를 지원합니다.
여기서 우리는 이러한 새로운 선언에 대해 자세히 설명하고, 왜 이것들이 `var`보다 선호하는 이유를 설명합니다.

만약 당신이 JavaScript를 잘못 사용했다면, 다음 섹션이 기억을 되살리는 좋은 방법이 될 수 있습니다.
JavaScript에서 `var` 의 모든 특징에 대해 잘 알고 있다면 건너 뛰는 것이 더 나을 수 있습니다.

## `var` 선언

JavaScript에서 변수 선언은 항상 전통적으로 `var` 키워드를 사용했습니다.

```ts
var a = 10;
```

아시다시피 방금 우리는 값이 `10` 인 변수 `a`를 선언했습니다.

함수 내에서 변수를 선언할 수도 있습니다:

```ts
function f() {
  var message = "Hello, world!";

  return message;
}
```

다른 함수 내에서 동일한 변수에 접근할 수도 있습니다:

```ts
function f() {
  var a = 10;
  return function g() {
    var b = a + 1;
    return b;
  };
}

var g = f();
g(); // '11' 반환
```

위의 예에서 `g`는 `f`에 선언 된 변수 `a`를 가져왔습니다. `g`가 호출되는 지점에서 `a`의 값은 `f`의 `a` 값이 됩니다. `f` 실행이 완료되었더라도 `g`가 호출되면, `a`에 접근하고 수정할 수 있습니다.

```ts
function f() {
  var a = 1;

  a = 2;
  var b = g();
  a = 3;

  return b;

  function g() {
    return a;
  }
}

f(); // '2' 반환
```

## 스코프 규칙

var 선언에는 다른 언어에 사용되는 몇 가지 스코프 규칙이 있습니다. 다음 예를 살펴보십시오:

```ts
function f(shouldInitialize: boolean) {
  if (shouldInitialize) {
    var x = 10;
  }

  return x;
}

f(true); // '10' 반환
f(false); // 'undefined' 반환
```

일부 독자들은 이 예제를 이미 보셨을 수도 있습니다.
변수 `x` 는 _`if` 블록 내에서_ 선언되었지만, 해당 블록 밖에서 x에 접근 할 수 있었습니다. 왜냐하면 `var` 선언은 이를 포함한 함수, 모듈, 네임스페이스 또는 전역 범위 내에서 어느 곳에서나 접근할 수 있기 때문이며, 이 것이 포함된 내용은 나중에 살펴보도록 하겠습니다. 어떤 사람들은 이것을 _`var`-scoping_ 또는 _function-scoping_이라고 부릅니다. 매개 변수도 함수 단위 스코프입니다.

이러한 스코프 규칙은 여러 유형의 실수를 유발할 수 있습니다.
그들이 더 악화시키는 문제 중 하나는 동일한 변수를 여러번 선언하는 것을 오류가 아니라고 하는 것입니다:

```ts
function sumMatrix(matrix: number[][]) {
  var sum = 0;
  for (var i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i];
    for (var i = 0; i < currentRow.length; i++) {
      sum += currentRow[i];
    }
  }

  return sum;
}
```

경험이 많은 JavaScript 개발자들은 쉽게 알아차릴 수 있었겠지만, `i`가 동일한 함수-스코프 변수를 참조하기 때문에 내부 `for`-루프가 실수로 변수 `i`를 덮어 쓰게됩니다.
경험 많은 개발자들이 알고 있듯이 비슷한 종류의 버그는 코드 리뷰에서 발견하지 못한 채 끝없는 좌절감의 원천이 될 수 있습니다.

## Variable capturing quirks

Take a quick second to guess what the output of the following snippet is:

```ts
for (var i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100 * i);
}
```

For those unfamiliar, `setTimeout` will try to execute a function after a certain number of milliseconds (though waiting for anything else to stop running).

Ready? Take a look:

```
10
10
10
10
10
10
10
10
10
10
```

Many JavaScript developers are intimately familiar with this behavior, but if you're surprised, you're certainly not alone.
Most people expect the output to be

```
0
1
2
3
4
5
6
7
8
9
```

Remember what we mentioned earlier about variable capturing?
Every function expression we pass to `setTimeout` actually refers to the same `i` from the same scope.

Let's take a minute to consider what that means.
`setTimeout` will run a function after some number of milliseconds, _but only_ after the `for` loop has stopped executing;
By the time the `for` loop has stopped executing, the value of `i` is `10`.
So each time the given function gets called, it will print out `10`!

A common work around is to use an IIFE - an Immediately Invoked Function Expression - to capture `i` at each iteration:

```ts
for (var i = 0; i < 10; i++) {
  // capture the current state of 'i'
  // by invoking a function with its current value
  (function (i) {
    setTimeout(function () {
      console.log(i);
    }, 100 * i);
  })(i);
}
```

This odd-looking pattern is actually pretty common.
The `i` in the parameter list actually shadows the `i` declared in the `for` loop, but since we named them the same, we didn't have to modify the loop body too much.

## `let` declarations

By now you've figured out that `var` has some problems, which is precisely why `let` statements were introduced.
Apart from the keyword used, `let` statements are written the same way `var` statements are.

```ts
let hello = "Hello!";
```

The key difference is not in the syntax, but in the semantics, which we'll now dive into.

## Block-scoping

When a variable is declared using `let`, it uses what some call _lexical-scoping_ or _block-scoping_.
Unlike variables declared with `var` whose scopes leak out to their containing function, block-scoped variables are not visible outside of their nearest containing block or `for`-loop.

```ts
function f(input: boolean) {
  let a = 100;

  if (input) {
    // Still okay to reference 'a'
    let b = a + 1;
    return b;
  }

  // Error: 'b' doesn't exist here
  return b;
}
```

Here, we have two local variables `a` and `b`.
`a`'s scope is limited to the body of `f` while `b`'s scope is limited to the containing `if` statement's block.

Variables declared in a `catch` clause also have similar scoping rules.

```ts
try {
  throw "oh no!";
} catch (e) {
  console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

Another property of block-scoped variables is that they can't be read or written to before they're actually declared.
While these variables are "present" throughout their scope, all points up until their declaration are part of their _temporal dead zone_.
This is just a sophisticated way of saying you can't access them before the `let` statement, and luckily TypeScript will let you know that.

```ts
a++; // illegal to use 'a' before it's declared;
let a;
```

Something to note is that you can still _capture_ a block-scoped variable before it's declared.
The only catch is that it's illegal to call that function before the declaration.
If targeting ES2015, a modern runtime will throw an error; however, right now TypeScript is permissive and won't report this as an error.

```ts
function foo() {
  // okay to capture 'a'
  return a;
}

// illegal call 'foo' before 'a' is declared
// runtimes should throw an error here
foo();

let a;
```

For more information on temporal dead zones, see relevant content on the [Mozilla Developer Network](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).

## Re-declarations and Shadowing

With `var` declarations, we mentioned that it didn't matter how many times you declared your variables; you just got one.

```ts
function f(x) {
  var x;
  var x;

  if (true) {
    var x;
  }
}
```

In the above example, all declarations of `x` actually refer to the _same_ `x`, and this is perfectly valid.
This often ends up being a source of bugs.
Thankfully, `let` declarations are not as forgiving.

```ts
let x = 10;
let x = 20; // error: can't re-declare 'x' in the same scope
```

The variables don't necessarily need to both be block-scoped for TypeScript to tell us that there's a problem.

```ts
function f(x) {
  let x = 100; // error: interferes with parameter declaration
}

function g() {
  let x = 100;
  var x = 100; // error: can't have both declarations of 'x'
}
```

That's not to say that a block-scoped variable can never be declared with a function-scoped variable.
The block-scoped variable just needs to be declared within a distinctly different block.

```ts
function f(condition, x) {
  if (condition) {
    let x = 100;
    return x;
  }

  return x;
}

f(false, 0); // returns '0'
f(true, 0); // returns '100'
```

The act of introducing a new name in a more nested scope is called _shadowing_.
It is a bit of a double-edged sword in that it can introduce certain bugs on its own in the event of accidental shadowing, while also preventing certain bugs.
For instance, imagine we had written our earlier `sumMatrix` function using `let` variables.

```ts
function sumMatrix(matrix: number[][]) {
  let sum = 0;
  for (let i = 0; i < matrix.length; i++) {
    var currentRow = matrix[i];
    for (let i = 0; i < currentRow.length; i++) {
      sum += currentRow[i];
    }
  }

  return sum;
}
```

This version of the loop will actually perform the summation correctly because the inner loop's `i` shadows `i` from the outer loop.

Shadowing should _usually_ be avoided in the interest of writing clearer code.
While there are some scenarios where it may be fitting to take advantage of it, you should use your best judgement.

## Block-scoped variable capturing

When we first touched on the idea of variable capturing with `var` declaration, we briefly went into how variables act once captured.
To give a better intuition of this, each time a scope is run, it creates an "environment" of variables.
That environment and its captured variables can exist even after everything within its scope has finished executing.

```ts
function theCityThatAlwaysSleeps() {
  let getCity;

  if (true) {
    let city = "Seattle";
    getCity = function () {
      return city;
    };
  }

  return getCity();
}
```

Because we've captured `city` from within its environment, we're still able to access it despite the fact that the `if` block finished executing.

Recall that with our earlier `setTimeout` example, we ended up needing to use an IIFE to capture the state of a variable for every iteration of the `for` loop.
In effect, what we were doing was creating a new variable environment for our captured variables.
That was a bit of a pain, but luckily, you'll never have to do that again in TypeScript.

`let` declarations have drastically different behavior when declared as part of a loop.
Rather than just introducing a new environment to the loop itself, these declarations sort of create a new scope _per iteration_.
Since this is what we were doing anyway with our IIFE, we can change our old `setTimeout` example to just use a `let` declaration.

```ts
for (let i = 0; i < 10; i++) {
  setTimeout(function () {
    console.log(i);
  }, 100 * i);
}
```

and as expected, this will print out

```
0
1
2
3
4
5
6
7
8
9
```

## `const` declarations

`const` declarations are another way of declaring variables.

```ts
const numLivesForCat = 9;
```

They are like `let` declarations but, as their name implies, their value cannot be changed once they are bound.
In other words, they have the same scoping rules as `let`, but you can't re-assign to them.

This should not be confused with the idea that the values they refer to are _immutable_.

```ts
const numLivesForCat = 9;
const kitty = {
  name: "Aurora",
  numLives: numLivesForCat,
};

// Error
kitty = {
  name: "Danielle",
  numLives: numLivesForCat,
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

Unless you take specific measures to avoid it, the internal state of a `const` variable is still modifiable.
Fortunately, TypeScript allows you to specify that members of an object are `readonly`.
The [chapter on Interfaces](/docs/handbook/interfaces.html) has the details.

## `let` vs. `const`

Given that we have two types of declarations with similar scoping semantics, it's natural to find ourselves asking which one to use.
Like most broad questions, the answer is: it depends.

Applying the [principle of least privilege](https://wikipedia.org/wiki/Principle_of_least_privilege), all declarations other than those you plan to modify should use `const`.
The rationale is that if a variable didn't need to get written to, others working on the same codebase shouldn't automatically be able to write to the object, and will need to consider whether they really need to reassign to the variable.
Using `const` also makes code more predictable when reasoning about flow of data.

Use your best judgement, and if applicable, consult the matter with the rest of your team.

The majority of this handbook uses `let` declarations.

## Destructuring

Another ECMAScript 2015 feature that TypeScript has is destructuring.
For a complete reference, see [the article on the Mozilla Developer Network](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment).
In this section, we'll give a short overview.

## Array destructuring

The simplest form of destructuring is array destructuring assignment:

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

This creates two new variables named `first` and `second`.
This is equivalent to using indexing, but is much more convenient:

```ts
first = input[0];
second = input[1];
```

Destructuring works with already-declared variables as well:

```ts
// swap variables
[first, second] = [second, first];
```

And with parameters to a function:

```ts
function f([first, second]: [number, number]) {
  console.log(first);
  console.log(second);
}
f([1, 2]);
```

You can create a variable for the remaining items in a list using the syntax `...`:

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

Of course, since this is JavaScript, you can just ignore trailing elements you don't care about:

```ts
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

Or other elements:

```ts
let [, second, , fourth] = [1, 2, 3, 4];
console.log(second); // outputs 2
console.log(fourth); // outputs 4
```

## Tuple destructuring

Tuples may be destructured like arrays; the destructuring variables get the types of the corresponding tuple elements:

```ts
let tuple: [number, string, boolean] = [7, "hello", true];

let [a, b, c] = tuple; // a: number, b: string, c: boolean
```

It's an error to destructure a tuple beyond the range of its elements:

```ts
let [a, b, c, d] = tuple; // Error, no element at index 3
```

As with arrays, you can destructure the rest of the tuple with `...`, to get a shorter tuple:

```ts
let [a, ...bc] = tuple; // bc: [string, boolean]
let [a, b, c, ...d] = tuple; // d: [], the empty tuple
```

Or ignore trailing elements, or other elements:

```ts
let [a] = tuple; // a: number
let [, b] = tuple; // b: string
```

## Object destructuring

You can also destructure objects:

```ts
let o = {
  a: "foo",
  b: 12,
  c: "bar",
};
let { a, b } = o;
```

This creates new variables `a` and `b` from `o.a` and `o.b`.
Notice that you can skip `c` if you don't need it.

Like array destructuring, you can have assignment without declaration:

```ts
({ a, b } = { a: "baz", b: 101 });
```

Notice that we had to surround this statement with parentheses.
JavaScript normally parses a `{` as the start of block.

You can create a variable for the remaining items in an object using the syntax `...`:

```ts
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;
```

### Property renaming

You can also give different names to properties:

```ts
let { a: newName1, b: newName2 } = o;
```

Here the syntax starts to get confusing.
You can read `a: newName1` as "`a` as `newName1`".
The direction is left-to-right, as if you had written:

```ts
let newName1 = o.a;
let newName2 = o.b;
```

Confusingly, the colon here does _not_ indicate the type.
The type, if you specify it, still needs to be written after the entire destructuring:

```ts
let { a, b }: { a: string; b: number } = o;
```

### Default values

Default values let you specify a default value in case a property is undefined:

```ts
function keepWholeObject(wholeObject: { a: string; b?: number }) {
  let { a, b = 1001 } = wholeObject;
}
```

In this example the `b?` indicates that `b` is optional, so it may be `undefined`.
`keepWholeObject` now has a variable for `wholeObject` as well as the properties `a` and `b`, even if `b` is undefined.

## Function declarations

Destructuring also works in function declarations.
For simple cases this is straightforward:

```ts
type C = { a: string; b?: number };
function f({ a, b }: C): void {
  // ...
}
```

But specifying defaults is more common for parameters, and getting defaults right with destructuring can be tricky.
First of all, you need to remember to put the pattern before the default value.

```ts
function f({ a = "", b = 0 } = {}): void {
  // ...
}
f();
```

> The snippet above is an example of type inference, explained later in the handbook.

Then, you need to remember to give a default for optional properties on the destructured property instead of the main initializer.
Remember that `C` was defined with `b` optional:

```ts
function f({ a, b = 0 } = { a: "" }): void {
  // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to { a: "" }, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```

Use destructuring with care.
As the previous example demonstrates, anything but the simplest destructuring expression is confusing.
This is especially true with deeply nested destructuring, which gets _really_ hard to understand even without piling on renaming, default values, and type annotations.
Try to keep destructuring expressions small and simple.
You can always write the assignments that destructuring would generate yourself.

## Spread

The spread operator is the opposite of destructuring.
It allows you to spread an array into another array, or an object into another object.
For example:

```ts
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```

This gives bothPlus the value `[0, 1, 2, 3, 4, 5]`.
Spreading creates a shallow copy of `first` and `second`.
They are not changed by the spread.

You can also spread objects:

```ts
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
```

Now `search` is `{ food: "rich", price: "$$", ambiance: "noisy" }`.
Object spreading is more complex than array spreading.
Like array spreading, it proceeds from left-to-right, but the result is still an object.
This means that properties that come later in the spread object overwrite properties that come earlier.
So if we modify the previous example to spread at the end:

```ts
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
```

Then the `food` property in `defaults` overwrites `food: "rich"`, which is not what we want in this case.

Object spread also has a couple of other surprising limits.
First, it only includes an objects'
[own, enumerable properties](https://developer.mozilla.org/docs/Web/JavaScript/Enumerability_and_ownership_of_properties).
Basically, that means you lose methods when you spread instances of an object:

```ts
class C {
  p = 12;
  m() {}
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```

Second, the TypeScript compiler doesn't allow spreads of type parameters from generic functions.
That feature is expected in future versions of the language.
