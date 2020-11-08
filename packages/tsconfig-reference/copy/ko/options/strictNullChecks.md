---
display: "Strict Null Checks"
oneline: "When type checking take into account `null` and `undefined`."
---

`strictNullChecks` 가 `false` 이면 `null` 과 `undefined`는 언어에 의해 사실상 무시됩니다.
이로 인해 런타임에 예기치 않은 오류가 발생할 수 있습니다.

`strictNullChecks`가 `true` 인 경우`null`과 `undefined`는 고유한 타입을 가지며 구체적인 값이 예상되는 곳에 사용하려고하면 타입 오류가 발생합니다.

이 TypeScript 코드로 예를 들자면, `users.find`가 실제로 user를 찾을 것이라는 보장은 없지만 다음과 같이 코드를 작성할 수 있습니다:

```ts twoslash
// @strictNullChecks: false
// @target: ES2015
declare const loggedInUsername: string;

const users = [
  { name: "Oby", age: 12 },
  { name: "Heera", age: 32 },
];

const loggedInUser = users.find((u) => u.name === loggedInUsername);
console.log(loggedInUser.age);
```

`strictNullChecks`를 `true`로 설정하면 `loggedInUser`가 사용하기 전에 존재한다는 보장을하지 않았다는 오류가 발생합니다.

```ts twoslash
// @errors: 2339 2532
// @target: ES2020
// @strictNullChecks
declare const loggedInUsername: string;

const users = [
  { name: "Oby", age: 12 },
  { name: "Heera", age: 32 },
];

const loggedInUser = users.find((u) => u.name === loggedInUsername);
console.log(loggedInUser.age);
```

두 번째 예제는 array의 `find` 함수가 다음과 같이 단순화 되었기 때문에 실패했습니다.

```ts
// strictNullChecks: true 일 떄
type Array = {
  find(predicate: (value: any, index: number) => boolean): S | undefined;
};

// strictNullChecks: false 일 때, undefined는 타입 시스템에서 제거됩니다.
// 항상 결과를 찾았다고 가정하는 코드를 작성할 수 있습니다.
type Array = {
  find(predicate: (value: any, index: number) => boolean): S;
};
```
