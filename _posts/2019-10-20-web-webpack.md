---
layout:     post
title:      "web开发-webpack使用"
subtitle:   "webpack使用"
date:       2019-10-22 17:10
author:     "LQFGH"
header-img: "img/webpack-banner.png"
header-mask: 0.3
catalog:    true
tags:

  - WEB
---

# Ⅰ安装



#### 一：全局安装

```shell
npm install --global webpack
```

>  *不推荐全局安装 webpack。这会将你项目中的 webpack 锁定到指定版本，并且在使用不同的 webpack 版本的项目中，可能会导致构建失败。* 

 如果你使用 webpack 4+ 版本，你还需要安装 CLI。 

```shell
npm install --save-dev webpack-cli
```



#### 二：本地安装

 要安装最新版本或特定版本，请运行以下命令之一： 

```shell
npm install --save-dev webpack
npm install --save-dev webpack@<version>
```

 如果你使用 webpack 4+ 版本，你还需要安装 CLI。 

```shell
npm install --save-dev webpack-cli
```



------

# Ⅱ基本打包

#### 一：手动打包

```shell
webpack src/js/entry.js -o dist/js/bundle.js
```

![1571751998801](/img/in-post/1571751998801.png)

> 根目录下多了 `dist` 文件,里面有 `bundle.js` 文件

控制台打印的日志

![1571752282572](/img/in-post/1571752282572.png)

time：打包花费的时间

Asset：打包完成的静态资源

Size：静态资源的大小

最后一行是源文件及其大小

#### 二：使用配置文件打包

根目录下创建一个配置文件  `**webpack.config.js**` 

```js
const path = require('path'); // node内置的模块用来设置路径

module.exports = {
    entry: './src/js/entry.js', // 入口
    output: {                // 出口
        filename: 'bundle.js', // 输出的文件名
        path: path.resolve(__dirname, 'dist/js/') // __dirname：node 中跟目录，此处path为根目录拼接后面的 dist
    }
};
```

接下来就可以直接在控制台使用 `webpack` 命令执行打包，不需要加参数

![1571753899768](/img/in-post/1571753899768.png)



------

# Ⅲ打包 css 和图片

#### 一：加载 CSS

 为了从 JavaScript 模块中 `import` 一个 CSS 文件，你需要在 `module` 配置中 安装并添加 `style-loader` 和 `css-loader`

```shell
npm install css-loader style-loader --save-dev
```

 **webpack.config.js** 

```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
+   module: {
+     rules: [
+       {
+         test: /\.css$/,
+         use: [
+           'style-loader',
+           'css-loader'
+         ]
+       }
+     ]
+   }
  };
```

> *webpack 根据正则表达式，来确定应该查找哪些文件，并将其提供给指定的 loader。在这种情况下，以* `.css` *结尾的全部文件，都将被提供给* `style-loader` *和* `css-loader`*。* 

#### 二：加载图片

 假想，现在我们正在下载 CSS，但是我们的背景和图标这些图片，要如何处理呢？使用 [file-loader](https://www.webpackjs.com/loaders/file-loader)，我们可以轻松地将这些内容混合到 CSS 中： 

```shell
npm install --save-dev file-loader
```

 **webpack.config.js** 

```js
const path = require('path'); // node内置的模块用来设置路径

module.exports = {
    entry: './src/js/entry.js', // 入口
    output: {                // 出口
        filename: 'bundle.js', // 输出的文件名
        path: path.resolve(__dirname, 'dist/js/') // __dirname：node 中跟目录，此处path为根目录拼接后面的 dist
    },
      module: {
          rules: [
              {
                  test: /\.css$/,
                  use: [
                      'style-loader',
                      'css-loader'
                  ]
              },
              {
                  test: /\.(png|svg|jpg|gif)$/,
                  use: [
                         'file-loader'
                  ]
              }
          ]
      }
};
```

> 这里需要注意的是当页面引入图片路径的问题



------

# Ⅳ自动编译打包

#### 一：安装

```shell
npm install --save-dev webpack-dev-server
```

 **webpack.config.js** 

```js
const path = require('path'); // node内置的模块用来设置路径

module.exports = {
    entry: './src/js/entry.js', // 入口
    output: {                // 出口
        filename: 'bundle.js', // 输出的文件名
        path: path.resolve(__dirname, 'dist/js/') // __dirname：node 中跟目录，此处path为根目录拼接后面的 dist
    },
      module: {
          rules: [
              {
                  test: /\.css$/,
                  use: [
                      'style-loader',
                      'css-loader'
                  ]
              },
              {
                  test: /\.(png|svg|jpg|gif)$/,
                  use: [
                      {
                          loader : 'url-loader',
                          options : {
                              limit : 8192
                          }
                      }
                  ]
              }
          ]
      },
      devServer: {
       contentBase: './dist'
      }
};
```

> 下面多了一个 `devServer` 的配置

**package.json**

```json
{
  "name": "webpack_test",
  "version": "1.0.0",
  "main": "webpack.config.json",
  "scripts": {
    "start": "webpack-dev-server --open"
  },
  "devDependencies": {
    "css-loader": "^3.2.0",
    "file-loader": "^4.2.0",
    "style-loader": "^1.0.0",
    "url-loader": "^2.2.0",
    "webpack": "^4.41.2",
    "webpack-cli": "^3.3.9",
    "webpack-dev-server": "^3.8.2"
  }
}
```

>  这个文件中添加了一个配置 `scripts`,其中添加了 `start` 配置

接下来在控制台中输入 `npm start` 就可以启动， 如果现在修改和保存任意源文件，web 服务器就会自动重新加载编译后的代码

但是打开的页面我们发现是这样的页面

![1571757142147](/img/in-post/1571757142147.png)

接下来添加配置解决该问题

**webpack.config.js**

```js
const path = require('path'); // node内置的模块用来设置路径

module.exports = {
    entry: './src/js/entry.js', // 入口
    output: {                // 出口
        filename: 'bundle.js', // 输出的文件名
        path: path.resolve(__dirname, 'dist/js/') // __dirname：node 中跟目录，此处path为根目录拼接后面的 dist
    },
      module: {
          rules: [
              {
                  test: /\.css$/,
                  use: [
                      'style-loader',
                      'css-loader'
                  ]
              },
              {
                  test: /\.(png|svg|jpg|gif)$/,
                  use: [
                      {
                          loader : 'url-loader',
                          options : {
                              limit : 8192
                          }
                      }
                  ]
              }
          ]
      },
      devServer: {
       contentBase: './dist' // webpack 默认服务于根目录下index.html页面，不配置的话可能找不到index.html页面
      }
};
```

> `devServer` 为添加的配置，配置到 `index.html` 所在的地址



------



# Ⅴ插件

#### 一：安装

```shell
npm install --save-dev html-webpack-plugin
```

#### 二：配置

```js
const path = require('path'); // node内置的模块用来设置路径
const HtmlWebpackPlugin = require('html-webpack-plugin'); //自动生成 html 文件的插件
// const CleanWebpackPlugin = require('clean-webpack-plugin'); // 清除之前打包的文件

module.exports = {
    entry: './src/js/entry.js', // 入口
    output: {                // 出口
        filename: 'bundle.js', // 输出的文件名
        path: path.resolve(__dirname, 'dist/js/') // __dirname：node 中跟目录，此处path为根目录拼接后面的 dist
    },
    module: {
          rules: [
              {
                  test: /\.css$/,
                  use: [
                      'style-loader',
                      'css-loader'
                  ]
              },
              {
                  test: /\.(png|svg|jpg|gif)$/,
                  use: [
                      {
                          loader : 'url-loader',
                          options : {
                              limit : 8192
                          }
                      }
                  ]
              }
          ]
      },
    devServer: {
       contentBase: './dist' // webpack 默认服务于根目录下index.html页面，不配置的话可能找不到index.html页面
      },
    plugins : [
        new HtmlWebpackPlugin({template: './index.html'}), // 根据index.html 生成新的html文件
        // new CleanWebpackPlugin(['dist']), //清楚的目录
    ]

};
```

> 新增了第2行和第36行

