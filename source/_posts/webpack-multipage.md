---
title: webpack react 多页面
date: 2020-07-07
tags: webpack
seotitle: webpack react 多页面
---

webpack 单页面 大家都挺熟悉的了，本文讲一讲 webpack 的多页面

#### 安装依赖

- webpack 相关

```bash
  npm i webpack webpack-cli webpack-dev-server webpack-merge -D
```

- js 相关

```bash
  npm i @babel/core @babel-loader @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators @babel/plugin-transform-runtime @babel/preset-env @babel/preset-react terser-webpack-plugin -D
```

- css 相关

```bash
  npm i css-loader style-loader sass-loader sass postcss-loader autoprefixer -D
```

- image 相关

```bash
  npm i url-loader file-loader html-withimg-loader -D
```

<!-- more -->

#### 主要讲讲 js 部分

- 首先是目录结构

```bash
npm init && touch webpack.config.js && mkdir src && cd src && mkdir pages && cd pages && mkdir saber && cd saber && touch saber.html saber.js && cd .. && mkdir archer && cd archer && touch archer.html archer.js
```

- 其次是生成入口和模板文件的文件，touch webpack.utils.js

```js
const path = require("path")
const glob = require("glob")
const HtmlWebpackPlugin = require("html-webpack-plugin") // html 模板

const whiteList = ["options"] // 入口白名单，在名单内的 不作为打包入口

const prefixRoot = "./src/pages/"
const JSFiles = glob.sync(`${prefixRoot}**/*.js`) // ['./src/pages/fate/fate.js']
const HtmlFiles = glob.sync(`${prefixRoot}**/*.html`) // ['./src/pages/fate/fate.html']

const fitlerJSFiles = (data) => {
  if (!data.length) return data

  return data.filter((v) => {
    const basename = path.basename(v, ".js") // 截取 ./src/pages/fate/fate.js => fate
    return !whiteList.includes(basename) // 过滤出白名单以外的 入口
  })
}

// 获取入口文件
const entry = fitlerJSFiles(JSFiles).reduce((obj, filePath) => {
  const entryChunkName = filePath
    .replace(path.extname(filePath), "") // ./src/pages/fate/fate
    .replace(prefixRoot, "") // fate/fate
  obj[entryChunkName] = filePath // obj['fate/fate'] = ./src/pages/fate/fate.js
  return obj
}, {})

// 根据对应模板生成对应的 html
const HtmlPlugins = HtmlFiles.map((filePath) => {
  const fileName = filePath.replace(prefixRoot, "")
  return new HtmlWebpackPlugin({
    chunks: [fileName.replace(path.extname(fileName), ""), "vendor"],
    template: filePath,
    filename: fileName,
    minify: {
      removeAttributeQuotes: true, // 移除属性的引号
      collapseWhitespace: false, // 文件压缩变成一行
    },
    hash: true,
  })
})

module.exports = {
  entry,
  HtmlPlugins,
}
```

#### webpack.config.js

```js
const path = require("path")
const distAbsolutePath = path.resolve(__dirname, "dist")
const { entry, HtmlPlugins } = require("./webpack.utils")

module.exports = {
  mode: "development", // development production
  entry: entry,
  output: {
    filename: "[name].js", // bundle.[hash:8].js
    path: distAbsolutePath, // 推荐绝对路径
  },
  module: {
    rules: [
      {
        test: /\.js$/i,
        include: path.resolve(__dirname, "src"), // 解析 src 目录下的
        exclude: /node_modules/, // 不解析 第三方包
        use: {
          loader: "babel-loader",
          options: {
            presets: [
              [
                "@babel/preset-env",
                {
                  useBuiltIns: "entry", // 用于解析  'xxxx'.includes('xx')
                  corejs: { version: 3, proposals: true },
                },
              ],
              "@babel/preset-react",
            ],
            plugins: [
              ["@babel/plugin-proposal-decorators", { legacy: true }], // 装饰器
              ["@babel/plugin-proposal-class-properties", { loose: true }], // 类的使用
              "@babel/plugin-transform-runtime", // 如 generator 那些，class 那些多次使用，统一走这个 runtime 生成的 helper 方法
            ],
          },
        },
      },
    ],
  },
  plugins: [...HtmlPlugins],
}
```

[github](https://github.com/aslkami/webpack/tree/multipage)
[webpack 路径问题](https://chenoge.github.io/2018/04/10/webpack-%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98/)
