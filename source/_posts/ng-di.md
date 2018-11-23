---
title: 理解 Angular 依赖注入
from: https://toddmotto.com/angular-dependency-injection
tags:
  angular
  frontend
  di
date: 2018-11-22 16:54:00
---
`Angular` 程序开发中 `Provider` 和依赖注入是两个非常重要的概念, 关系到我们如何编写程序. 本文将解释 `@Inject()`, `@Injectable()` 两个装饰器背后的原理和它们的使用场景, 以及 `Angular` 依赖注入框架中的 `token`, `provider` 和 `Angular` 是如何创建依赖的.
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

当我们使用装饰器时, 将被装饰的类的元信息存储成能被 `Angular` 处理的格式, 这些元信息中就包含了这个类的依赖.

如果没有使用装饰器添加元信息, `Angular` 就无从得知类的依赖. 这就是我们为什么需要 `@Injectable()`. `@Injectable()` 并没有其他的功能, 仅仅是提供了一些元信息.

## Token 和依赖注入

现在我们知道了如何告诉 `Angular` 需要注入的内容, 现在我们来学习 `Angular` 如何知道从哪获取依赖以及如何实例化他们.

### 注册 provide

我们先来看如何注册一个服务到一个 `NgModule` 上.

```ts
import { NgModule } from '@angular/core';

import { AuthService } from './auth.service';

@NgModule({
  providers: [AuthService],
})
class ExampleModule {}
```

上面例子是下面的简化版:

```ts
import { NgModule } from '@angular/core';

import { AuthService } from './auth.service';

@NgModule({
  providers: [{
    provide: AuthService,
    useClass: AuthService
  }],
})
class ExampleModule {}
```

其中 `provide` 字段表示我们注册的 provider 的 `token`. 当 `Angular` 看到 `AuthService` `token` 时, 就会取 `useClass` 的值进行实例化.

这样做有很多优点. 其一, 我们可以注册多个相同 `class` 的 `provide`, 而且不会发生冲突 (只要 `token` 不一样); 第二, 我们可以通过相同的 `token` 来覆盖之前的 `provide`.

### 覆盖 `providers`

我们的 `AuthService` 代码如下:

```ts
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';

@Injectable()
export class AuthService {

  constructor(private http: Http) {}

  authenticateUser(username: string, password: string): Observable<boolean> {
    // returns true or false
    return this.http.post('/api/auth', { username, password });
  }

  getUsername(): Observable<string> {
    return this.http.post('/api/user');
  }

}
```

假设我们的程序中使用了这个服务, 例如登录:

```ts
import { Component } from '@angular/core';
import { AuthService } from './auth.service';

@Component({
  selector: 'auth-login',
  template: `
    <button (click)="login()">
      Login
    </button>
  `
})
export class LoginComponent {

  constructor(private authService: AuthService) {}

  login() {
    this.authService
      .authenticateUser('toddmotto', 'straightouttacompton')
      .subscribe((status: boolean) => {
        // do something if the user has logged in
      });
  }

}
```

显示用户名:

```ts
@Component({
  selector: 'user-info',
  template: `
    <div>
      You are {{ username }}!
    </div>
  `
})
class UserInfoComponent implements OnInit {

  username: string;

  constructor(private authService: AuthService) {}

  ngOnInit() {
    this.authService
      .getUsername()
      .subscribe((username: string) => this.username = username);
  }

}
```

我们将上面所有代码整合成一个模块, 比如 `AuthModule`:

```ts
import { NgModule } from '@angular/core';

import { AuthService } from './auth.service';

import { LoginComponent } from './login.component';
import { UserInfoComponent } from './user-info.component';

@NgModule({
  declarations: [LoginComponent, UserInfoComponent],
  providers: [AuthService],
})
export class AuthModule {}
```

其他的组件可能也会依赖 `AuthService`. 现在假设我们有一个新的需求, 需要修改授权方法使得用户可以通过 Facebook 登录.

一种方法是修改所有的组件, 将其构造函数中的 `AuthService` 替换成新的服务. 另外我们也可以通过修改 `providers` 来将 `AuthService` 覆盖成 `FacebookAuthService`:

```ts
import { NgModule } from '@angular/core';

// totally made up
import { FacebookAuthService } from '@facebook/angular';

import { AuthService } from './auth.service';

import { LoginComponent } from './login.component';
import { UserInfoComponent } from './user-info.component';

@NgModule({
  declarations: [LoginComponent, UserInfoComponent],
  providers: [
    {
      provide: AuthService,
      useClass: FacebookAuthService,
    },
  ],
})
export class AuthModule {}
```

这里我们没有采用简写方法, 并且替换了 `useClass` 的值. 这样在模块中的 `AuthService` `token` 就会使用 `FacebookAuthService` 类.

## 理解注入器 (Injector)

翻译略 (一些 AOT 源码的解释, 有兴趣的可以研究下).

## 参考资料

- [装饰器反射相关4篇](http://blog.wolksoftware.com/decorators-reflection-javascript-typescript)