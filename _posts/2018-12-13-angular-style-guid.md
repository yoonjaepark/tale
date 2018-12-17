---
layout: post
title: "앵귤러 스타일 가이드"
author: "YoonJae"
description: "사이트의 가장 중요한건 바로 SEO다"
---

## 목차
---
1. [파일을 400 줄로 제한하는 것을 고려하십시오.](#파일을-400-줄로-제한하는-것을-고려하십시오)
1. [함수를 짧게 작성하세요.](#함수를-짧게-작성하세요)
1. [trackBy](#trackby)
1. [const vs let](#const-vs-let)
1. [Pipeable operators](#pipeable-operators)
1. [Isolate API hacks](#isolate-api-hacks)
1. [Subscribe in template](#subscribe-in-template)
1. [Clean up subscriptions](#clean-up-subscriptions)
1. [Use appropriate operators](#use-appropriate-operators)
1. [Lazy load](#lazy-load)
1. [Avoid having subscriptions inside subscriptions](#avoid-having-subscriptions-inside-subscriptions)
1. [Avoid any; type everything;](#avoid-any;-type-everything;)
1. [Make use of lint rules](#make-use-of-lint-rules)
1. [Small reusable components](#small-reusable-components)
1. [Components should only deal with display logic](#components-should-only-deal-with-display-logic)
1. [Avoid long methods](#avoid-long-methods)
1. [DRY](#dry)
1. [Add caching mechanisms](#add-caching-mechanisms)
1. [Avoid logic in templates](#avoid-logic-in-templates)
1. [Strings should be safe](#strings-should-be-safe)
1. [State Management](#state-management)
1. [Immutable state](#immutable-state)
1. [Jest](#jest)
1. [Karma](#karma) 
1. [Universal](#universal)

---

## [참조 URL](#참조)

---

## 파일을 400 줄로 제한하는 것을 고려하십시오.

---

* 같은 파일 당 서비스 또는 컴포넌트 요소로, 한 가지를 정의합니다. 
* 파일 당 하나의 컴포넌트 를 사용하면 소스 제어에서 팀과의 충돌을 읽고, 유지하고, 충돌을 피할 수 있습니다. 
* 파일 당 하나의 구성 요소는 변수를 공유하거나 불필요한 종결을 만들거나 종속성과 원하지 않는 결합을 일으키는 파일에서 구성 요소를 결합 할 때 종종 발생하는 숨겨진 버그를 방지합니다. 
* 하나의 구성 요소가 파일의 기본 내보내기가 될 수 있으므로 라우터를 통한 지연로드가 용이합니다. 
* **핵심은 코드를 재사용 가능하고 읽기 쉬우며 실수가 적은 코드로 만드는 것입니다.**

[top](#목차)

## 함수를 짧게 작성하세요.

---

* 75 줄 이하로 제한하는 것을 고려하십시오. 
* 함수는 한 가지 일을 하고 한 가지 목적을 수행 할 때 테스트하기 쉽습니다. 
* 작은 함수는 재사용을 용이하고 유지 보수가 쉽습니다.

[top](#목차)

## trackBy

---

* 배열 리스트가 변경될 때 DOM 트리를 다시 랜더링 하지만 trackBy하면 Angular는 어떤 요소가 변경되었는지 알게되고 특정 요소에 대해서만 DOM을 변경합니다.

```hack
// bad
<li *ngFor="let item of items;">{{ item }}</li>

// good
// in the template

<li *ngFor="let item of items; trackBy: trackByFn">{{ item }}</li>
// in the component
trackByFn(index, item) {
   return item.id; // unique id corresponding to the item
}
```
[top](#목차)

## const vs let

---

* const 변경되지 않는 값에 사용하십시오.
* let 다른 곳에서 사용할 경우.
* 실수로 컴파일 타임 오류가 발생하여 값을 상수에 다시 할당 할 때 문제를 식별하는 데 도움이됩니다. 
* 코드의 가독성을 향상시키는데도 도움이됩니다.

```typescript
// bad
let car = 'ludicrous car';
let myCar =`My $ {car}`;

// good
const car = 'ludicrous car';
let myCar =`My $ {car}`;
yourCar =`Your $ {car};
```
[top](#목차)

## Pipeable operators

---

* RxJs 연산자를 사용할 때는 pipe를 사용하세요.
* 파이프 가능한 연산자는 트리 셰이크가 가능하므로 우리가 실행해야하는 코드 만 가져올 때 포함됩니다.
또한 파일에서 사용되지 않는 연산자를 쉽게 식별 할 수 있습니다.

- **"pipable"(이전 "lettable") 연산자는 RxJS 5.5부터 연산자를 사용 하는 현재의 권장 방법 입니다.**

```typescript
// bad
import "rxjs/add/operator/map";
import "rxjs/add/operator/take";

iAmAnObservable.map(value => value.item).take(1);

// good
import { map, take } from "rxjs/operators";

iAmAnObservable.pipe(
  map(value => value.item),
  take(1)
);
```

[top](#목차)

## Isolate API hacks

---

* 모든 API가 확실한 것은 아니며, 때때로 API의 버그를 보완하기 위해 코드에 로직을 추가해야 할 때도 있다. 
* 서비스처럼 한 곳에 격리하고 컴포넌트에서 서비스를 사용하는 것이 좋다. 
* API에서 버그를 수정할 때 코드베이스에 분산될 수 있는 버그를 찾는 것보다 한 파일에서 버그를 찾는 것이 더 쉽다.

[top](#목차)

## Subscribe in template

---

컴포넌트에서 데이터 변경 감지 하지말고 템플릿에서 감지해야합니다. async파이프는 자동으로 구독을 취소하고 구독을 수동으로 관리 할 필요가 없으므로 코드가 간단 해집니다. 또한 구성 요소의 구독을 실수로 잊어 버리면 메모리 누수가 발생할 위험이 줄어 듭니다. 이 위험은 lint 규칙을 사용하여 구독 취소 된 관찰 가능 항목을 탐지하여 완화 할 수도 있습니다. 또한 구성 요소가 상태가 유지되고 구독 데이터 외부에서 데이터가 변경되는 버그가 발생하는 것을 방지합니다.

```typescript
// bad
// template
<p>{{"{{ textToDisplay "}}}}</p>
// component
iAmAnObservable
.pipe(
  map(value => value.item),
  takeUntil(this._destroyed$)
)
.subscribe(item => this.textToDisplay = item);

// good
// template
<p>{{"{{ textToDisplay$ | async "}}}}</p>
// component
this.textToDisplay$ = iAmAnObservable
.pipe(
  map(value => value.item)
);
```

[top](#목차)

## Clean up subscriptions

---

subscriptions을 사용할 때는 항상 적절하게  `take`, `takeUntil` 같은 연산자를 사용하여 구독을 해제 했는지 확인합니다. 스트림 구독을 취소하지 못하면 관찰 가능한 스트림이 열려 있기 때문에 원치 않는 메모리 누수가 발생할 수 있습니다. 잠재적으로 구성 요소가 손상되었거나 사용자가 다른 페이지로 이동 한 후에도 가능합니다. 더 좋게는, 구독 취소되지 않은 관찰 가능 항목을 탐지하기위한 make a lint rule을 작성하세요.

```typescript
// bad
iAmAnObservable
.pipe(
  map(value => value.item)
)
.subscribe(item => this.textToDisplay = item);

// good
private destroyed$ = new Subject();
public ngOnInit (): void {
  iAmAnObservable
  .pipe(
    map(value => value.item)
    // We want to listen to iAmAnObservable until the component is destroyed,
    takeUntil(this._destroyed$)
  )
  .subscribe(item => this.textToDisplay = item);
}

public ngOnDestroy (): void {
  this._destroyed$.next();
}
```

```typescript
// good
iAmAnObservable
.pipe (
  map (value => value.item),
  take (1),
  takeUntil (this._destroyed$)
)
.subscribe (item => this.textToDisplay = item);
```

takeUntil다른 관측 값이 값을 방출 할 때까지 변경 사항을 듣고 싶을 때 사용
이와 같이 개인적인 주체를 사용하는 것은 구성 요소에서 많은 관측 가능 항목의 구독 취소를 관리하는 패턴입니다.

`take`observable에 의해 생성 된 첫 번째 값만 원할 때 사용
여기 에 takeUntilwith 의 사용법에 유의 take하십시오. 이는 구성 요소가 손상되기 전에 구독이 값을받지 못했을 때 발생하는 메모리 누수를 방지하기위한 것입니다. takeUntil여기가 없으면 구독은 첫 번째 값을 얻을 때까지 계속 돌아가지만 구성 요소가 이미 파괴 되었기 때문에 절대로 값을 얻지 못하여 메모리 누수가 발생합니다.

[top](#목차)

## Use appropriate operators

---

구독 가능 개체와 함께 병합 연산자를 사용할 때는 해당 연산자를 사용하십시오.

*switchMap :* 새로운 출력이있을 때 이전 출력을 무시하려는 경우

*mergeMap :* 모든 출력을 동시에 처리하려는 경우

*concatMap :* 출력 될 때 하나씩 다른 출력을 처리하려는 경우

*exhaustMap :* 이전 출력을 처리하는 동안 새 출력을 모두 취소하려는 경우

가능하면 단일 연산자를 사용하여 여러 다른 연산자를 결합하여 동일한 효과를 얻으려는 경우 코드가 사용자에게 더 효과적 일 수 있습니다. 잘못된 연산자를 사용하면 다른 연산자가 다른 방식으로 구독 가능 항목을 처리하므로 원치 않는 동작이 발생할 수 있습니다.

[top](#목차)

## Lazy load

---

가능하면 Angular 응용 프로그램에서 모듈을 지연 로드하십시오. 지연로드는 사용되는 경우에만 로드합니다. 이렇게 하면 로드 할 응용 프로그램의 크기가 줄어들고 사용되지 않는 모듈을 로드하지 않아 응용 프로그램 부팅 시간을 향상시킬 수 있습니다.

```typescript
// bad
// app.routing.ts
{ path: 'not-lazy-loaded', component: NotLazyLoadedComponent }

// good
// app.routing.ts
{
  path: 'lazy-load',
  loadChildren: 'lazy-load.module#LazyLoadModule'
}
// lazy-load.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { LazyLoadComponent }   from './lazy-load.component';
@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild([
      {
        path: '',
        component: LazyLoadComponent
      }
    ])
  ],
  declarations: [
    LazyLoadComponent
  ]
})

export class LazyModule {}
```

[top](#목차)

## Avoid having subscriptions inside subscriptions

---

때로는 하나 이상의 observable 구독 값을 사용하여 작업을 수행 할 수 있습니다. 이 경우, 다른 관찰 대상의 구독 블록에서 하나의 관찰 가능 항목을 구독하지 마십시오. 대신 적절한 연결 연산자를 사용하십시오. chaining operators 이전에 관측 대상에서 실행됩니다. 일부 chaining operators 다음과 같습니다 withLatestFrom, combineLatest등

```typescript
// bad
firstbuservable$.pipe (
  take (1)
)
.subscribe(firstValue => {
  secondObservable$.pipe (
    take(1)
  )
  .subscribe(secondValue => {
    console.log (`결합 된 값은 $ {firstValue}와 $ { secondValue}`);
  });
});

// good
firstObservable$.pipe (
  withLatestFrom(secondObservable$),
  first()
)
.subscribe(([firstValue, secondValue]) => {
  console.log(`Combined values are: ${firstValue} & ${secondValue}`);
});

```

[top](#목차)

## Avoid any; type everything;

---

항상 변수나 상수를 타입을 선언하세요.

타이핑없이 Typescript에서 변수 나 상수를 선언 할 때, 변수 / 상수의 타이핑은 그것에 할당 된 값에 의해 추론됩니다. 이로 인해 의도하지 않은 문제가 발생합니다.

새 프로젝트를 시작하는 경우 모든 엄격한 유형 검사 옵션을 사용 strict:true하려면 tsconfig.json파일에서 설정하는 것이 좋습니다.

```typescript
// bad
const x = 1;
const y = 'a';
const z = x + y;
console.log(`Value of z is: ${z}`);
// Output
Value of z is 1a

// good
const x: number = 1;
const y: number = 'a';
const z: number = x + y;
// This will give a compile error saying:
Type '"a"' is not assignable to type 'number'.
const y:number
```

[top](#목차)

## Make use of lint rules

---

lint rule을 정해두면, 컴파일 전에 미리 많은 규칙을 정해 미리 방지 할 수 있습니다.를 얻을 수 있습니다. 또 가독성이 일관되게 유지됩니다. 구성 할 수있는 추가 규칙 은 여기 를 참조 하십시오 .

tslint 레퍼런스 - [https://palantir.github.io/tslint/rules/](https://palantir.github.io/tslint/rules/)

tsquery 레퍼런스 - [https://github.com/phenomnomnominal/tsquery](https://github.com/phenomnomnominal/tsquery)

``` typescript
// bad
public ngOnInit (): void {
  console.log('I am a naughty console log message');
  console.warn('I am a naughty console warning message');
  console.error('I am a naughty console error message');
}
// Output
 No errors, prints the below on console window:
I am a naughty console message
I am a naughty console warning message
I am a naughty console error message

// good
// tslint.json
{
  "rules": {
    .......
    "no-console": [
      true,
      "log",    // no console.log allowed
      "warn"    // no console.warn allowed
    ]
  }
}
// ..component.ts
public ngOnInit (): void {
  console.log('I am a naughty console log message');
  console.warn('I am a naughty console warning message');
  console.error('I am a naughty console error message');
}
// Output
Lint errors for console.log and console.warn statements and no error for console.error as it is not mentioned in the config
Calls to 'console.log' are not allowed.
Calls to 'console.warn' are not allowed.
```

[top](#목차)

## Small reusable components

---

구성 요소에서 재사용 할 수있는 컴포넌트를 추출하여 새 구성 요소로 만듭니다. 구성 요소를 가능한 한 단순하게 만들면 더 많은 시나리오에서 작동합니다. 구성 요소를 단순하게 만드는 것은 구성 요소에 특별한 논리가 없고 해당 구성 요소에 제공된 입력과 출력을 기반으로 순전히 작동한다는 것을 의미합니다. 일반적으로 구성 요소 트리의 마지막 자식은 모두 단순할 것입니다. 재사용 가능한 구성 요소는 코드 중복을 줄이므로 유지 관리와 변경 작업을보다 쉽게 ​​수행 할 수 있습니다. 단순한 구성 요소는 더 간단하므로 버그가 발생할 가능성이 적습니다. 단순한 구성 요소를 사용하면 공용 구성 요소 API에 대해 더 많은 것을 생각하게되며 된 우려 사항을 파악하는 데 도움이됩니다.

[top](#목차)

## Components should only deal with display logic

---

구성 요소에서 로직을 제외 하세요. 구성 요소는 표시를 위해 설계되었으며 뷰에서 수행해야 할 작업을 제어합니다. 모든 비즈니스 로직은 비즈니스 로직을 뷰 로직과 분리하여 적절한 경우 자체 메소드나 서비스로 추출해야합니다. 비즈니스 논리는 일반적으로 서비스에 추출 할 때 단위 테스트가 더 쉽고 같은 비즈니스 로직을 적용해야하는 다른 구성 요소에서 재사용 할 수 있습니다.

[top](#목차)

## Avoid long methods

---

긴 메소드는 일반적으로 너무 많은 일을하고 있음을 나타냅니다. Single Responsibility 지향하세요. 단일책임원칙 이란 모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 함을 일컫는다. 클래스가 제공하는 모든 기능은 이 책임과 주의 깊게 부합해야 한다. 긴 메소드는 이해 및 유지하기가 어렵습니다. 한 가지를 변경하면 그 방법의 많은 다른 것들에 영향을 미칠 수 있으므로 버그가 발생하기 쉽습니다. 또한 리팩토링 을 어렵게 만듭니다. 이것은 때때로 "[순환 적 복잡성](<[https://en.wikipedia.org/wiki/Cyclomatic_complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)>)"으로 측정됩니다. 또한 순환 적 복잡성을 감지하기 위한 몇 가지 "[TSLint 규칙](<[https://www.npmjs.com/package/tslint-sonarts](https://www.npmjs.com/package/tslint-sonarts)>)"이 있습니다. 이를 사용해 프로젝트에서 버그를 피하고 코드 냄새 및 유지 관리 문제를 감지하는 데 사용할 수 있습니다.

[top](#목차)

## DRY

---

코드를 반복하지 마십시오. 동일한 코드가 다른 위치에 복사되지 않도록 하십시오. 동일한 코드를 여러 위치에 두는 것은 해당 코드의 논리를 변경 하려는 경우 여러 위치에서 수행 해야 한다는 것을 의미합니다. 이로 인해 유지 관리가 어려워지고 모든 코드를 변경하지 못하여 버그가 발생하기 쉽습니다. 중복되는 로직을 변경하는 데 시간이 오래 걸리며 테스트도 더 오래 걸리게 됩니다. 이러한 경우 반복 코드를 추출해 사용하십시오. 이것은 변경해야 할 한 곳과 테스트 할 한 곳을 의미합니다. 발생되는 중복 코드가 적어지면 응용 프로그램이 더 빨라 질것입니다.

[top](#목차)

## Add caching mechanisms

---

API 호출을 할 때 일부 함수은 응답이 자주 변경되지 않습니다. 이러한 경우 캐싱 메커니즘을 추가하고 API에서 값을 저장할 수 있습니다. 동일한 API에 대한 다른 요청이 있을 때 캐시에 값이 있는지 확인하고 캐시에 값이 있으면 사용하십시오. 그렇지 않으면 API 호출을 수행하고 결과를 캐시에 저장 하십시오.
값은 변경되지만 자주는 아니지만 마지막으로 캐시 저장된 시간을 확인하고 API를 호출할지 여부를 결정할 수있는 캐시 주기를 설정할 수 있습니다.
캐싱 메커니즘을 사용하면 원하지 않는 API 호출을 피할 수 있습니다. 필요한 경우에만 API 호출을 수행하고 중복을 피함으로써 네트워크를 기다릴 필요가 없으므로 애플리케이션 속도가 향상됩니다. 또한 동일한 정보를 반복해서 다운로드하지 행위를 피할수 있습니다.

* 참고 [https://blog.thoughtram.io/angular/2018/03/05/advanced-caching-with-rxjs.html#fetching-new-data-on-demand](https://blog.thoughtram.io/angular/2018/03/05/advanced-caching-with-rxjs.html#fetching-new-data-on-demand)

[top](#목차)

## Avoid logic in templates

---

html 템플릿에 로직이 있다면, 간단한 '&&'절이더라도 컴포너트 클래스로 추출하는 것이 좋습니다. 템플릿에 논리가 있으면 단위 테스트가 불가능하므로 템플릿 코드를 변경할 때 버그가 발생하기 쉽습니다.

```typescript
// bad
// template
<p *ngIf="role==='developer'"> Status: Developer </p>
// component
public ngOnInit (): void {
  this.role = 'developer';
}

// good
// template
<p *ngIf="showDeveloperStatus"> Status: Developer </p>
// component
public ngOnInit (): void {
  this.role = 'developer';
  this.showDeveloperStatus = true;
}
```

[top](#목차)

## Strings should be safe

---

문자열 유형으로 선언하는 대신 일련의 값만 가질 수있는 string 유형의 변수가있는 경우 가능한 값 목록을 타입으로 선언 할 수 있습니다. 변수의 타입을 적절하게 선언함으로써 런타임 중에 컴파일 타임에 코드를 작성하는 동안 버그를 피할 수 있습니다.

```typescript
// bad
private myStringValue: string;
if (itShouldHaveFirstValue) {
  myStringValue = 'First';
  } else {
    myStringValue = 'Second'
  }

// good
private myStringValue: 'First' | 'Second';
if (itShouldHaveFirstValue) {
  myStringValue = 'First';
} else {
  myStringValue = 'Other'
}
// This will give the below error
Type '"Other"' is not assignable to type '"First" | "Second"'
(property) AppComponent.myValue: "First" | "Second"
```

[top](#목차)

## State Management

---

Consider using [@ngrx/store](https://github.com/ngrx/platform) for maintaining the state of your application and [@ngrx/effects](https://github.com/ngrx/effects) as the side effect model for store. State changes are described by the actions and the changes are done by pure functions called reducers.

**Why?**

*@ngrx/store* isolates all state related logic in one place and makes it consistent across the application. It also has memoization mechanism in place when accessing the information in the store leading to a more performant application. *@ngrx/store* combined with the change detection strategy of Angular leads to a faster application.

[top](#목차)

## Immutable state

---

When using @ngrx/store, consider using ngrx-store-freeze to make the state immutable. ngrx-store-freeze prevents the state from being mutated by throwing an exception. This avoids accidental mutation of the state leading to unwanted consequences.

Why?

Mutating state in components leads to the app behaving inconsistently depending on the order components are loaded. It breaks the mental model of the redux pattern. Changes can end up overridden if the store state changes and re-emits. Separation of concerns — components are view layer, they should not know how to change state.

[top](#목차)

## Jest

---

[Jest](https://jestjs.io/)는 Facebook 용 자바 스크립트용 단위 테스트 프레임 워크입니다. 코드 기반에서 테스트 실행을 병렬 처리하여 단위 테스트를 더 빠르게 수행합니다. whatch 모드에서는 변경 사항과 관련된 테스트 만 실행되므로 테스트 응답이 짧씁니다. Jest는 테스트의 코드 범위를 제공하며 VS Code, Webstorm에서 지원됩니다.
Jest에 [preset](https://github.com/thymikee/jest-preset-angular)을 참고하세요.

[top](#목차)

## Karma

---

[Karma](<[https://karma-runner.github.io/2.0/index.html](https://karma-runner.github.io/2.0/index.html)>)는 AngularJS 팀이 개발 한 테스트 러너입니다. 테스트를 실행하려면 실제 브라우저 / DOM이 필요합니다. 그것은 또한 다른 브라우저에서 실행할 수 있습니다. Jest는 테스트를 실행하기 위해 크롬 헤드리스 / 팬텀을 필요로하지 않으며 순수한 노드에서 실행됩니다.

[top](#목차)

## Universal

---

앱을 Universal 앱으로 만들지 않았다면 지금 할 수 있습니다. [Angular Universal](<[https://angular.io/guide/universal](https://angular.io/guide/universal)>)을 사용하면 서버에서 Angular 응용 프로그램을 실행하고 정적 사전 렌더링 된 HTML 페이지를 제공하는 서버 사이드 렌더링 (SSR)을 수행 할 수 있습니다. 이렇게 하면 JS 번들로드 및 구문 분석을 기다릴 필요없이 Angular to bootstrap을 사용하지 않고도 화면에 콘텐츠를 거의 즉시 표시 할 수 있으므로 앱이 빠르게 빠르게 처리됩니다. 또한 Angular Universal은 정적 컨텐츠를 생성하고 웹 크롤러가 자바 스크립트를 실행하지 않고도 응용 프로그램의 색인을 생성하고 검색 가능한 상태가되도록하기 때문에 SEO가 편리합니다. Universal은 응용 프로그램의 성능을 대폭 향상시킵니다. 사이트 로딩 시간이 수 초에서 수십 밀리 초로 단축시킬 수 있습니다. 또한 사이트가 소셜 미디어 미리보기 스니펫에 올바르게 표시되도록합니다. 첫 번째로 의미있는 페인트는 정말 빠르며 원치 않는 지연없이 사용자에게 콘텐츠를 보여줍니다.

[top](#목차)

## Conclusion

---

좋은 가이드라인을 잡아 주실분은 댓글이나 커밋해주시면 잘 배우겠습니다. 

[https://github.com/yoonjaepark/yoonjaepark.github.io/blob/master/_posts/2018-12-13-angular-style-guid.md
](https://github.com/yoonjaepark/yoonjaepark.github.io/blob/master/_posts/2018-12-13-angular-style-guid.md
)

## 참조
---
- [https://angular.io/guide/styleguide](https://angular.io/guide/styleguide)
- [https://medium.freecodecamp.org/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f](https://medium.freecodecamp.org/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f)

[top](#목차)
