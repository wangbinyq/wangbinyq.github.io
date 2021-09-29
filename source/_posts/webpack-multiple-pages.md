---
title: Webpack 多页面配置
date: 2021-09-29 12:22:16
tags:
  - Webpack
  - 多页面
---

本文将基于 `Webpack 5` 配置出一套多页面的前端开发工作流. 适用于传统的后端模板渲染情景. 示例代码请点击[链接](https://github.com/wangbinyq/webpack-multiple-pages-example).

<!-- more -->

## 基础配置

首先我们创建一个空文件夹, 然后创建一个基础的 `Node` 项目: `yarn init -y`. 添加 `Webpack` 依赖 `yarn add -D webpack webpack-cli`.

创建一个基本 `webpack.conf.js` 配置文件:

```js
module.exports = {

}
```

## 添加多页面

接下来我们添加多页面 `a.html` 和 `b.html` 在 `src` 中. 这里 `a.html` 是一个完成的 `html`, `b.html` 是一个 `html` 片段, 因为大多数后端模板有 `layout` 机制, 不需要每个模板文件都是完整的 `html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div class="a">
    This is a.html
  </div>
</body>
</html>
```

```html
<div class="b">
  This is b.html
</div>
```

然后我们添加脚本和样式文件, 我们将使用 `TypeScript` 和 `Babel` 来编译成 `js` 文件, 使用 `sass` 编译 `css` 文件. 具体文件内容参考示例.

现在我们的文件依赖结构是

![文件依赖](/images/webpack-multiple-pages/inital-depend.png)

## Build 多页面

接下来我们添加依赖

```sh
yarn add -D typescript babel-loader @babel/core @babel/preset-env @babel/preset-typescript sass sass-loader css-loader mini-css-extract-plugin html-webpack-plugin
```

生成 `tsconfig.json` 文件:

```sh
yarn tsc --init
```

修改 `webpack.config.js`:

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin")

// 定义了我们的页面和 chunks
var pages = ['a', 'b']

module.exports = {
  mode: 'production',
  // 通过 ts 定义页面入口文件
  entry: pages.reduce((e, p) => ({...e, [p]: `./src/${p}.ts`}), {}),
  output: {
    filename: '[name].[contenthash:8].js',
    clean: true,
  },
  module: {
    rules: [
      {
        test: /\.[jt]sx?$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: [
              "@babel/preset-typescript",
            ]
          }
        }
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              filename: '[name].[chunkhash:8].css'
            }
          },
          'css-loader',
          'sass-loader'
        ]
      }
    ],
  },
  plugins: [
    // css 文件单独抽取出来, 参考 https://webpack.js.org/plugins/mini-css-extract-plugin/
    new MiniCssExtractPlugin(
      {
        filename: '[name].[contenthash:8].css'
      }
    ),
    // 生成 html, 参考 https://github.com/jantimon/html-webpack-plugin
    ...pages.map(p => new HtmlWebpackPlugin({
      // 不自动注入 chunks
      inject: false,
      template: `./src/${p}.html`,
      // 生成的 html 文件名
      filename: `${p}.html`,
      // 引用入口文件定义的 chunks
      chunks: [p]
    }))
  ],
  resolve: {
    extensions: ['.ts', 'tsx', '.js']
  },
  // 自动进行代码分割, 参考 https://webpack.js.org/plugins/split-chunks-plugin/
  optimization: {
    splitChunks: {
      chunks: 'all'
    }
  }
}
```

现在我们执行 `yarn webpack`, 可以发现在 `dist` 文件下有以下这些文件:

![文件生成](/images/webpack-multiple-pages/webpack-1.png)

查看 `a.html`, 发现它并没有引用我们希望的 `a.xxxxx.js` 和 `a.xxxxx.css`, 这是因为我们设置了 `inject: false`, 不让 `webpack` 自动注入, 我们希望手动注入这些脚本和样式文件:

通过

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <%= htmlWebpackPlugin.tags.headTags %>
</head>
<body>

  <div class="a">
    This is a.html
  </div>
</body>
</html>
```

```html
<%= htmlWebpackPlugin.tags.headTags %>

<div class="b">
  This is b.html
</div>
```

再次执行 `yarn webpack`, 可以看到生成的 `dist/a.html` 文件 `head` 部分已经包含了我们脚本和样式文件.

