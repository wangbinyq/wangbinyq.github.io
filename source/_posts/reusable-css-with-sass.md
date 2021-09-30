---
title: 使用 SASS 编写可复用的 CSS
from: https://blog.logrocket.com/how-to-write-reusable-css-with-sass/
date: 2021-09-30 11:20:05
tags:
  - sass
  - css
  - 复用
---

[Sass](https://sass-lang.com)是一个 `CSS` 预处理器，在前端工程师的工具箱中正变得至关重要。`Sass` 之所以受到欢迎，是因为它解决了一些 `CSS` 的陷阱。

它也是 [Bootstrap 4](https://getbootstrap.com) 的运行基础。这意味着学习 `Sass` 对于理解如何操作 `Bootstrap` 代码是非常有帮助的，而不是重写代码（这是大多数开发者的习惯方法）。了解 `Sass` 可以更好地在源代码层面理解使用的工具。

当使用 `CSS` 时，我们经常是在一个全局的范围内工作，同时会错误地对一个元素进行样式更改。

自定义 `CSS`（即使有 `CSS` 变量）仍然是非常多余的。`CSS` 并不是为我们今天这种复杂的架构而设计的，我们会遇到在另一个样式表中导入一个样式表的问题，这可能会导致一个非常大的样式库，如果没有适当的文档，可能就无法理解。

<!-- more -->

## TL; DR

在这篇文章中，我们将集中讨论为什么预处理器很重要，以及 `SASS` 和将其规则组合在一起的能力。使用 `SASS` 可以为现代网络组件的样式设计提供一个更合理的方法。

我们还将研究为什么我们要使用这些预处理器，并通过演示例子说明我们如何将样式分解成更小的特定组件，同时无需我们的用户下载大量不需要的 `CSS` 文件。

## 前提条件

在我们进一步讨论之前，本文假设了以下条件。

- [Node.js ≥v6](https://nodejs.org/) 已安装在你的机器上
- 你的机器上安装了 [npm](https://www.npmjs.com/)。
- [Sass](https://sass-lang.com) 已安装在你的机器上
- 你的机器上安装了 [Create-react-app](https://www.npmjs.com/package/create-react-app)
- 你对 CSS 有基本的了解

## 开始使用

`Sass` 可以通过多种方式添加到你的项目中，你可以找到所有的安装选项[这里](https://sass-lang.com/install)。在这篇文章中，我们将通过运行 [npm](https://www.npmjs.com/) 来使用。

```sh
npm install -g sass
```

## CSS 的问题？

在学习网页开发的基础知识时，我们会被介绍到传统的 `CSS`，它涉及使用 `class` 或 `id` 等标识符来处理和操作 `HTML`。

在使用 `CSS` 的过程中，我们经常要调整样式，让它看起来像我们想要的那样。编写大型样式表确实很有压力。同时保持样式选择器的范围，以避免样式以非预期的方式工作，这会让人很快就疲惫。

即使引入了 `CSS` 变量可以减少声明的重复，但是它也存在一些问题，使用预处理程序就可以处理。比如说，变量名非常的长。

即使有了 `CSS3` 的出现，我们仍然需要依靠技术 (基本上是黑客) 来设计用户界面的样式。我们还可以注意到，在编写 `HTML` 时，有一个清晰的嵌套和视觉层次，而编写时普通的 `CSS` 时则没有这样层次。

让我们来看看对 `CSS` 中所缺少的功能的 "解决方案"。

## 什么是 CSS 预处理器

CSS 预处理器基本上可以被认为是这样一个程序，它让你从它们自己特定的语法中生成 `CSS`。`CSS` 预处理器一般会增加一些纯 `CSS` 中不存在的功能，如 [mixin](https://sass-lang.com/documentation/at-rules/mixin)、[嵌套选择器](http://thesassway.com/beginner/the-inception-rule)、[继承选择器](https://github.com/webplatform/webplatform.github.io/blob/master/docs/tutorials/inheritance_and_cascade/index.html)。同时也给我们提供了一种非常结构化的编写样式的方式。CSS 预处理器的例子包括 [LESS](http://lesscss.org), [stylus](http://stylus-lang.com), [Sass](https://sass-lang.com), [PostCSS](https://postcss.org). 如前所述，本文主要讨论 Sass 这个预处理程序。

![csspreprocessors](/images/csspreprocessor/csspreprocessor-nocdn.webp)

## SASS 还是 SCSS？

现在，如果你是 `Sass` 的新手，你可能会想，'如果 `Sass` 是预处理器，那么 `SCSS` 是什么'？好吧，因为 `Sass` 在使用时会让人产生混乱，因为没有分号 `;` 和大括号 `{}`，取而代之的是使用标签和空格。

在 `Sass` 的[版本 3](https://sass-lang.com/documentation)中， `SCSS` 语法被引入作为 `Sass` 的主要语法，它包含了 `CSS` 的所有特性，但允许使用 `Sass` 的特性。在我看来，无论哪种语法都可以用来做样式，而且并不比其他的好。对 `SCSS` 的需求是为了使 `Sass` 的学习曲线和实现更快，并且不犯错误。

Sass:

```scss
$font-stack: Helvetica, sans-serif
$primary-color: #333
body
  font: 100% $font-stack
  color: $primary-color
```

SCSS:

```scss
$font-stack: Helvetica, sans-serif;
$primary-color: #333;
body {
  font: 100% $font-stack;
  color: $primary-color;
}
```

在上面的代码例子中，我们注意到 `Sass` 和 `SCSS` 写作风格的不同。请注意，它们都使用 `$` 来声明一个变量。

## SCSS 中的概念

### 嵌套和范围

当对 `HTML` 文件进行样式设计时，`SCSS` 使你在样式文件中拥有与 `HTML` 相同的视觉层次，这样你就能以一种更易理解的方式实际映射出你的样式设计。例如，对这个 `index.html` 进行造型。

```html
<nav class="sidebar">
  <ul>
    <li><a> </a></li>
  </ul>
</nav>
```

普通 CSS:

```css
nav ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
nav li {
  display: inline-block;
}
nav a {
  display: block;
  padding: 6px 12px;
  text-decoration: none;
}
```

SCSS:

```scss
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }
  li {
    display: inline-block;
  }
  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```

我们可以从上面的 `SCSS` 代码例子推断出 `HTML` 文件的结构，同时保持实现的简短和简单。这样做的另一个好处是，它有助于避免拼写错误，另外你可以看到，我们对一些规则进行了范围化处理，使它们只适用于 `nav` 内部。

在 SCSS 中应用的是子元素样式规则:

```scss
.container {
  .left-area { 
    ...
  }
}
```

这意味着只有 `.container` 中具有 `.left-area` 类的子元素会受到这些规则的影响。基本的 CSS 选择器仍然适用于 SCSS，例如。

## 直接后裔 (>)

```scss
.container{
  > .left-area{ ... }
}
```

现在，只有 `.contaner` 的直接子代的 `.left-area` 类才会得到样式。

## 父选择器 (&)

如果我们想通过添加一个类来修改一个类，可以利用父选择器，它主要用于添加二级样式改变元素的样式的情况, 起到修改器的作用。

```scss
.container {
  & .right-area {
    background-color: #0000;
  }
}
```

我们也可以使用父选择器将样式应用到另一个类上，像这样。

```scss
button {
  color: #349;
  .theme-dark & {
    color: #fff;
  }
}
```

从代码的例子来看，由于父选择器的存在，颜色 `#fff` 只适用于 `.theme-dark` 类。

## 变量

通常，在 `CSS` 中，我们通过使用 `@import` 将各种样式文件连接起来，引入到主 `CSS` 文件中。这对用户来说意味着什么呢？这将意味着用户必须下载额外的 CSS 文件。

使用 `SCSS` 将所有这些输入文件编译成为单个的 `CSS` 文件，那会怎么样呢？CSS 中变量的概念来自于 JavaScript 的方法。

请注意，SCSS 中的 `@import` 是用来将其他文件中的导入到 SCSS 文件的，但它们不会被编译成 `CSS` 文件。一般它们的文件名以 `_` 开头。

## 使用 SCSS 的变量

**全局变量：** 顾名思义，这些变量可以在一个 `CSS` 块中被访问。如果你熟悉 JavaScript 中的范围，你会理解[全局变量](https://funprogramming.org/50-What-are-global-and-local-variables.html)。

SCSS 中的变量总是以符号 `$` 开头。

```scss
$color: #f002 .color {
  $text_color: #ddd;
  background-color: $color;
  color: $text_color;
  text-shadow: 0 0 2px darken($text_color, 40%);
}
```

从上面的代码中，我们注意到 `$text_color` 只能在代码块中访问。

## 混合函数 `Mixin`

SCSS 的另一个神奇的功能是它能够将可重复使用的样式打包在一起，并允许在需要时导入到另一个样式块中，以减少代码的冗余度。

## 声明

创建一个 `mixin `很简单，只需在样式块前添加一个 `@mixin`，后面跟上 `mixin` 的名字，就像这样。

```scss
@mixin {insert name} { // write CSS code here }
```

## 使用

要在代码块中使用混合函数，我们必须使用 `@include`，后面跟着混合函数的名称，然后是分号。

```scss
.nav {
  @include {mixin name}
}
```

另一种使用混合函数的方法是使用参数，就像 `JavaScript` 中的函数一样，我们可以声明一个全局变量并将其设置为混合函数的参数。

```scss
@mixin text-color($color) {
  background-color: $color;
  color: white;
}
//import
.name {
  @include text-color(orange);
}

.background {
  @include text-color(white);
}
```

现在想象一下，如果我们想要一个 `mixin` 参数有默认值，并且要在不同的代码块中改变或重新分配这个值，我们会使用一些参数。为了说明这一点，我将重构之前的代码例子。

```scss
@mixin text-color($color: #fff) {
  background-color: $color;
  color: white;
}
//import
.name {
  @include text-color(orange);
}

.background {
  @include text-color($color: white);
}
```

这样做的目的是，它为 `mixin` 设置了一个默认的颜色，同时它可以通过重新赋值来进行修改。我们也可以将该值设置为 `null`，以便在混合器中只使用我们需要的参数。

```scss
@mixin text-color($color: null) {
  background-color: $color;
  color: #038;
}
//import
.name {
  @include text-color();
}
.background {
  @include text-color(#fff);
}
```

这样使 `@include` 导入的代码块中值为 `null` 的属性没有值, 在最后生成的 `CSS` 中不存在，但其他值不变。

在 `mixin` 中重新定义一个样式块。

```scss
@mixin text-color($color) {
  color: $color;
  .extra {
    @content;
  }
}
//import
.name {
  @include text-color(#fff) {
    color: blue;
  }
}
```

这个代码块基本上让我们能够通过对父类的样式来保持简单的结构，同时也能够定义内部类的样式。

## 值得注意的信息

- 混合函数不会在 CSS 文件中存在，它们被设计为被只在 `@include` 中使用，所以你会在 SCSS 中看到它们是 [partials](https://dev.to/sarah_chima/using-sass-partials-7mh)
- 混合函数不需要在前面加上点.

## 函数

`SCSS` 中的函数是 `SASS` 功能的一个重要部分，它们允许你定义复杂的操作，并在整个样式表中重复使用。有很多内置的 `SASS` 函数可以帮助你。请查看[文档](https://sass-lang.com/documentation/at-rules/function)以了解更多信息。

下面是一些你应该熟悉的函数的列表

- [lighten($color, $amount)](http://sass-lang.com/documentation/Sass/Script/Functions.html#lighten-instance_method) : 使一个颜色变浅
- [darken($color, $amount)](http://sass-lang.com/documentation/Sass/Script/Functions.html#darken-instance_method) : 使一个颜色变深
- [adjust-hue($color, $degrees)](http://sass-lang.com/documentation/Sass/Script/Functions.html#adjust_hue-instance_method) : 改变一个颜色的色调
- [mix($color1, $color2, $weight)](http://sass-lang.com/documentation/Sass/Script/Functions.html#mix-instance_method) : 混合颜色+第一种颜色的权重/还有下面的颜色的返回值，以用于条件反射
- [hue($color)](http://sass-lang.com/documentation/Sass/Script/Functions.html#hue-instance_method) : 获取一个颜色的色调成分
- [saturation($color)](http://sass-lang.com/documentation/Sass/Script/Functions.html#saturation-instance_method) : 获取颜色的饱和度分量
- [lightness($color)](http://sass-lang.com/documentation/Sass/Script/Functions.html#lightness-instance_method) : 获取颜色的亮度分量

我们也可以编写自己的 Sass 函数。

```scss
$width: 4px;
@function double($x) {
  @return 2\* $x;
}
.thin-border {
  border-width: $width;
}
.thick-border {
  border-width: double($width);
}
```

上面的函数只是在以正常宽度为参数时，将宽度加倍。

## 使用条件语句构造样式

这个功能非常出色，因为它允许我们根据另一个样式的值，使用 `@if` 和 `@else `声明来交替定义一个特定的样式。例如，对于不同的 `font-size` 值，动态地增加行高。

```scss
@mixin modify($size) {
  font-size: $size;
  @if $size > 18 {
    line-height: $size;
  }
}
//import
.name {
  @include modify(24px);
}
```

## 其他一些需要注意的特点

- 注释 - SCSS 中的多行注释与 CSS 中的显示方式相同，但内联会注释被删除。它们可以用于字符串插值（知道在 CSS 中实际显示的是什么值）。
- Sass 还配备了`@for`用于迭代和[控制流](https://sass-lang.com/documentation/at-rules/control)，这可以用于混合函数和函数。

## 总结

在这篇文章中，我们试图了解用 `SCSS` 编写功能性 `CSS` 的基础知识，同时也看了一些 `Sass/SCSS` 的一般原则。我希望我们能将这些实践用于为我们的应用程序编写更多无压力和优化的样式。编码愉快!😄
