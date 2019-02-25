---
layout: post
title: "Var 와 Let, Const 차이"
author: "YoonJae"
description: "Var 와 Let, Const 차이 | 초보개발자 기술 블로그"
---

## Var 와 Let, Const 차이

1. [let과 const는 es6 이후 변수 선언방식](#let과-const는-es6-이후-변수-선언방식)
1. [let과 const는 한번 선언하면 재선언 불가능하다](#let과-const는-한번-선언하면-재선언-불가능하다)
1. [let은 재할당은 가능하다.](#let은-재할당은-가능하다)
1. [var는 Function Scoped이고 let const는 Block Scoped입니다](#var는-function-scoped이고-let-const는-block-scoped입니다)

## let과 const는 es6 이후 변수 선언방식

```typescript
/// es5
var a = 0;

/// es6
let a = 0;
const b = 0;
```

## let과 const는 한번 선언하면 재선언 불가능하다.

```typescript
/// es5
var a = 0;
var a = 1;

/// es6
let a = 0;
let a = 1; // Uncaught SyntaxError: Identifier 'a' has already been declared
const b = 0;
const b = 1; // Uncaught SyntaxError: Identifier 'b' has already been declared
```

## let은 재할당은 가능하다.

```typescript
let a;
a = 1;

const a; // Uncaught SyntaxError: Missing initializer in const declaration
```

- let은 변수의 재할당이 가능하지만 const 는 재할당이 불가능하다. 이처럼 변수의 선언 초기화가 누락되었다는 에러가 발생한다.

## var는 Function Scoped이고 let, const는 Block Scoped입니다.

```typescript
// es5;

var a = '전역변수';

(function b() {
  var a = '함수범위 스코프';
  console.log(a); // 1. 함수범위 스코프
})();
console.log(a); // 2. 전역변수

var b = '전역 변수';
{
    var b = '블록내 선언 변수';
}

console.log(b); // 3. 블록내 선언 변수
```
* var 은 function scope이기 때문에 함수 단위로 스코프가 변경되어 위와 같이 로그가 함수 내에서는 1. a가 `함수범위 스코프`로 찍히고 함수밖의 변수 3.b는 `블록내 선언 변수`로 찍힙니다.

```typescript
// es6
let a = '전역변수';

(function b() {
  let a = '함수범위 스코프';
  console.log(a); // 1. 함수범위 스코프
})();
console.log(a); // 2. 전역변수

let b = '전역 변수';
{
    let b = '블록내 선언 변수';
}
console.log(b); // 3. 전역 변수
```
* let 은 block scope이기 때문에 블록 단위로 스코프가 변경되어 위와 같이 로그가 함수 내에서는 1. a가 `함수범위 스코프`로 찍히고 함수밖의 변수 3. b는 `전역변수`로 찍힙니다.

```typescript
// es5
console.log(a); // undefined

a = '호이스팅';
var a;
console.log(a); // 호이스팅
```

```typescript
// es5
console.log(a); // undefined

a = '호이스팅';
var a;
console.log(a); // 호이스팅
```

```typescript
// es6
console.log(a); // Uncaught ReferenceError: a is not defined

a = '호이스팅';
let a;
console.log(a);
```
* let이나 const를 사용하면 이처럼 변수를 읽으려 해도 초기화 구문이 아직 실행되지 않아 ReferenceError가 발생합니다. 이처럼 var 보단 let과 const를 활용하여 자바스크립트의 코드를 보다 명확한 코드를 작성하도록 합니다. 
