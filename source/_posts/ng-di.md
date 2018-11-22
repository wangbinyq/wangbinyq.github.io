---
title: Angular 依赖注入详解
from: https://toddmotto.com/angular-dependency-injection
tags:
  angular
  frontend
  di
date: 2018-11-22 16:54:00
---
`Angular` 程序开发中 `Provider` 和依赖注入是两个非常重要的概念, 关系到我们如何编写程序. 本文将解释 `@Inject()`, `@Injectable()` 两个装饰器背后的原理和它们的使用场景, 以及 `Angular` 依赖注入框架中的 `token`, `provider` 和 `Angular` 是如何创建依赖的. 最后会有一些 AOT 源码的解释.
<!-- more -->

## 注入 providers

`Angular` 的依赖注入背后有很多魔法发生. `Angular 1.x` 时代我们可以通过字符串 token 来获取指定的依赖.

```js
function SomeController($scope) {
  // use $scope
}
SomeController.$inject = ['$scope']
```

你可以通过之前的[文章](https://toddmotto.com/angular-js-dependency-injection-annotation-process/)来获取更多信息.

虽然这种方式曾经很好, 但是也存在一些缺陷. 通常我们会通过创建各种模块 (`Module`) 和引用其他模块 (比如 `ui-router`) 来构建我们的程序. 不同的模块中的 controllers/services 不能用相同的名字, 否则就会发成冲突.

幸运的是,新的  `Angular` 重写了依赖注入系统, 使其更强大更灵活.

### 新的依赖注入系统

当需要注入服务 (`provider`) 到组件或其他服务中时, 我们在构造函数中指定需要注入的类型. 比如:

```ts
import { Component } from '@angular/core';
import { Http } from '@angular/http';

@Component({
  selector: 'exmaple-component',
  template: '<div>I am a component</div>',
})
class ExampleComponent {
  constructor(private http: Http) {
    // use `this.http` which is the Http provider
  }
}
```

注入的类型标记为 `Http`, `Angular` 能自动将其赋值给 `http`.

这看起来非常的神奇. 类型标记只在 `TypeScript` 中存在, 当程序转译成 `JavaScript` 并在浏览器中运行时, 我们对 `http` 参数毫无所知 (在运行时 `http` 可以为任何对象).

我们需要将 `tsconfig.json` 中的 `emitDecoratorMetadata` 设成 `true`. 这样在转译后的 `JavaScript` 的代码中, 会将参数类型加到装饰器上.

我们来看转译后的 `JavaScript` 代码 (ES6):

```js
import { Component } from '@angular/core';
import { Http } from '@angular/http';

var ExampleComponent = (function() {
  function ExampleComponent(http) {
    this.http = http;
  }
  return ExampleComponent;
})();
ExampleComponent = __decorate(
  [
    Component({
      selector: 'example-component',
      template: '<div>I am a component</div>',
    }),
    __metadata('design:paramtypes', [Http]),
  ],
  ExampleComponent
);import { Component } from '@angular/core';
import { Http } from '@angular/http';

@Component({
  selector: 'exmaple-component',
  template: '<div>I am a component</div>',
})
class ExampleComponent {
  constructor(private http: Http) {
    // use `this.http` which is the Http provider
  }
}
```

这里我们能看到转译后的代码中 `http` 对应上了 `@angualr/http` 中的 `Http`

```js
__metadata('design:paramtypes', [Http]);
```

最终 `@Component` 装饰器转译成了 ES 代码, 并通过 `__decorate` 附加了一些元信息 `metadata`. 这些信息能告诉 `Angular` 要将 `Http` 传递给组件的构造函数, 并最终赋值给 `this.http`.

```js
function ExampleComponent(http) {
  this.http = http;
}
```

`metadata` 实现了 `$inject` 中功能, 但是这里作为 `token` 的是类而不是字符串. 命名冲突就不会发生了.

<div class="tip">
你可能已经听过 token (或者 OpaqueToken) 的概念. Angular 使用 token 来存储或获取 providers. Token 作为 key 来引用 (hash?) provider. 不同于传统的 key, token 可以是任何值作为 key (对象, 字符串, 类等).
<div>

### `@Inject()`

所以 `@Inject` 起什么作用呢? 我们将组件改写成:

```ts
import { Component, Inject } from '@angular/core';
import { Http } from '@angular/http';

@Component({
  selector: 'example-component',
  template: '<div>I am a component</div>'
})
class ExampleComponent {
  constructor(@Inject(Http) private http) {
    // use `this.http` which is the Http provider
  }
}
```

这一次, 我们通过 `@Inject` 手动提供 `token`.

如果组件或服务需要很多依赖, 这样写会非常的麻烦. `Angular` 可以通过 `metadata` 自动找到依赖, 所有大多数情况下我们都不需要使用 `@Inject`.

唯一需要 `@Inject` 的情况是在同时使用 `OpaqueToken` 对象时.

```ts
const myToken = new OpaqueToken('myValue')

@Component(...)
class ExampleComponent {
  constructor(private token: myToken) {}
}
```

这里 `myToken` 不是类型, 而是值. 这意味着上面的代码会报错. 但是使用 `@Inject` 的话就可以了:

```ts
const myToken = new OpaqueToken('myValue')

@Component(...)
class ExampleComponent {
  constructor(@Inject(myToken) private token) {}
}
```

现在不会深入 `OpaqueToken`, 但是上面的例子足够表明 `@Inject` 的作用.

### `@Injectable()`

我们并不需要在所有的类上添加 `@Injectable` 装饰器, 才可以将其注入到组件或服务中. 虽然可能会发生变化, 有一个 [issue](https://github.com/angular/angular/issues/13820) 讨论需要强制加上 `@Injectable` (4.0 已经变成强制需要 `@Injectable`)

当我们使用

