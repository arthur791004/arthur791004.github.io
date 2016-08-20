---
title: webpack-react-scss-eslint-material
date: 2016-08-20 14:49:17
tags: [ webpack, react ]
---
## Goal
* 本篇的目的是建立一個完整的 react 開發流程並包含 production 的部分
* 完整的開發流程目前包含
  * webpack 建立一個 hot reload 的 debug server
  * react with es6
  * 使用 scss 打造 css
  * 使用 url-loader 處理 images, fonts, ...等檔案
  * 使用 eslint 統一 coding style 和檢查容易犯的錯誤
  * 使用 material-ui
* 之後會以此延伸，再加入以下事項
  * redux for single source of truth
  * mocha for auto testing
  * Circle CI for continuous integration
  * PureRender
  * ...

## webpack-dev-server
* 首先第一步，是使用 webpack-dev-server，來打造一個具有 hot reload 的 server，方便看執行結果。
* npm install --save-dev webpack webpack-dev-server html-webpack-plugin clean-webpack-plugin
  * html-webpack-plugin
    * generate index.html from template
  * clean-webpack-plugin
    * clean build before complie
* webpack.config.js
  * webpack 的設定檔，用來建置 webpack-dev-server 和 build code
  * hash v.s chunkhash
    * [hash] is replaced by the hash of the compilation. (timestamp)
    * [chunkhash] is replaced by the hash of the chunk.  (md5)
  * eval
    * 官方說是速度最快的，無論是第一次 compile 或是 re-compile
  * Example
```js
    var path = require('path');
    var HtmlWebpackPlugin = require('html-webpack-plugin');

    module.exports = {
      output: {
        path: path.resolve(__dirname, 'build'),
        filename: '[name].[chunkhash].js',
      },
      plugins: [
        new CleanPlugin('build'),
        new HtmlWebpackPlugin({
          template: path.resolve(__dirname, 'app/index.html'),
          filename: 'index.html'
        })
      ],
      devtool: 'eval',
      devServer: {
        contentBase: 'build',
        host: '0.0.0.0'
      }
    };
```

* app/index.html
  * 簡易的 index.html sample
```
    <!DOCTYPE html>
    <html>
      <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>React Hello World</title>
      </head>
      <body>
        <div id="root"></div>
      </body>
    </html>
```
* package.json
  * 加入 dev 指令，方面使用
  * Example
```json
    "scripts": {
      "dev": "webpack-dev-server --devtool eval --progress --colors,
      ...
    }
```
* test
  * npm run dev
  * hot reload with iframe-mode (more simple than inline-mode)
    * http://<host>:<port>/webpack-dev-server/index.html
* build
  * webpack
  * build/
    * index.html
* 當然除了 webpack-dev-server，也可以透過其他方法達到 hot reload 的效果
  * webpack-dev-middleware + webpack-hot-middleware

## react
* 接著就可以把 react 加進來，建置一個 hello world 的 Example
* npm install --save react react-dom
* app/components/Hello.jsx
  * 建立一個 hello 的 component
```js
  import React, { Component, PropTypes } from 'react';

  class Hello extends Component {
    constructor(props) {
      super(props);
    }

    render() {
      return <div>Hello World</div>
    }
  }

  Hello.propTypes = {

  };

  export default Hello;
```
* app/app.jsx
```js
  import React, { Component } from 'react';
  import ReactDOM from 'react-dom';

  import Hello from './components/Hello';

  class App extends Component {
    render() {
      return <Hello />;
    }
  }

  ReactDOM.render(
    <App />,
    document.getElementById('root')
  );
```
* 重新 npm run dev 就可以看到 Hello World 了！

## webpack settings
* 在使用 react 時，必須使用 babel-loader 和搭配轉碼規則
  * ES2015 轉碼規則
```bash
    npm install --save-dev babel-preset-es2015
```
  * react 轉碼規則
```
    npm install --save-dev babel-preset-react
```
  * ES7 不同階段語法提案的轉碼規則（共有4個階段），選一個裝
```
    npm install --save-dev babel-preset-stage-0
    npm install --save-dev babel-preset-stage-1
    npm install --save-dev babel-preset-stage-2
    npm install --save-dev babel-preset-stage-3
```
* 修改 webpack.config.js，加入 loaders 和 resolve 的部分
```js
  var path = require('path');
  var HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: path.resolve(__dirname, 'app/index.jsx'),
    ...
    module: {
      loaders: [
        {
          test: /\.jsx$/,
          exclude: /node_modules/,
          loader: 'babel-loader',
          query: {
            presets: ['es2015', 'stage-0', 'react']
          }
        }
      ]
    },
    resolve: {
      extensions: ['', '.js', '.jsx']
    },
    ...
  };
```

## 處理 vendor.js
* 這邊有一個問題，就是我們每次對 jsx 檔有任何改變，它轉換的時間太長，明明我們只有2個檔案而已，不應該花到 100ms 左右的速度。
* 會發生這個原因是因為我們使用 react 的 jsx 轉換時，每次改變 jsx 檔，就需要重新 bundle 一次，但每次 bundle 都會把 require('react') 一起 bundle 進來，所以我們應該把 react 獨立出變成一個 vendor ，每次 bundle 時就只要重新編譯其 有改變的部分就好。
* webpack.config.js
  * entry 分成 app 和 vendor
```
    entry: {
      app: path.resolve(__dirname, 'app/app.jsx'),
      vendor: ['react', 'react-dom']
    },
```
  * plugins 增加 webpack.optimize.CommonsChunkPlugin
```
  plugins: [
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'app/index.html'),
      filename: 'index.html'
    }),
    new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.[chunkhash].js')
  ]
```

## 樣式loaders (css, scss, images, fonts, ...)
* 再來要處理 scss + autoprefix, images, fonts, ...等檔案的部分
* npm install --save-dev style-loader css-loader node-sass sass-loader autoprefixer-loader file-loader url-loader
  * sass-loader 需要安裝 node-sass，不然會出現 ERROR in Cannot find module 'node-sass'
* 首先是 scss + autoprefix 的部分
  * 包含： style-loader, css-loader, sass-loader, autoprefixer-loader
  * add loaders in webpack.config.js
```js
    {
      test: /\.scss$/,
      loader: 'style!css?modules!autoprefixer!sass?sourceMap'
    }
```
  * app/app.scss
```css
    body {
      background-color: yellow;
    }
    
    .app {
      background-color: blue;
    }
```
  * app/app.jsx
```js
      import styles from './app.scss';
      ...
      <div className={ styles.app }>
        <Hello />
      </div>
```

* url-loader
  * 幫我們偵測字型檔、圖片檔、woff(2) 等這些來源如果在多少大小以內，就幫轉成base64
  * hash by file-loader
    *  the hash of the content, hex-encoded md5 by default
  * add loaders in webpack.config.js
```js
    {
      test: /\.(jpg|gif|png)$/,
      loader: 'url?limit=8192&name=images/[name].[hash:20].[ext]',
      exclude: /node_modules/,
    },
    {
      test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
      loader: 'url?limit=8192&name=images/[name].[hash:20].[ext]'
    },
    {
      test:  /\.woff(2)?(\?v=[0-9]\.[0-9]\.[0-9])?$/,
      loader: 'url?limit=8192&minetype=application/font-woff2&name=images/[name].[hash:20].[ext]'
    }
```
  * add images in app/app.jsx
```js
    import imgPikachu from './images/pikachu.jpg';
        ...
       <div className="app">
         <Hello />
         <img src={ imgPikachu } />
       </div>
       ...
```

* file-loader
  * 用來讀取字型檔、圖片檔用的，但此處不需特別指定，因為 url-loader 也具備 file-loader 的效果，且會將指定 size 以內的檔案轉成 base64

## extract-text-webpack-plugin
* 這地方是把 css 的部分從 js 抽取出來，獨立成一個檔案
* npm install --save-dev extract-text-webpack-plugin
* webpack.config.js
```
var ExtractPlugin = require('extract-text-webpack-plugin');
    ...
    {
      test: /\.scss$/,
      loader: ExtractPlugin.extract('style', 'css!sass')
    }
    ...
    plugins: [
      new ExtractPlugin('[name].[hash].css', { allChunks: true })
    ]
```
* app.jsx
```js
  ...
  <div className="app">
  ...
```

## eslint
* npm install --save-dev eslint eslint-loader eslint-plugin-react 
* .eslintrc
  * eslint --init
* add preloader in webpack.config.js
```jsx
  preLoaders: [
    {
      test: /\.jsx$/,
      loader: 'eslint-loader',
    }
  ],
```

## using material-ui
* npm install --save material-ui
* add material-ui to vendor in webpack.config.js
```js
  vendor: ['react', 'react-dom', 'material-ui'],
```
* app/app.jsx
```
  import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
  import RaisedButton from 'material-ui/RaisedButton';
        ...
       <MuiThemeProvider>
        <div className="app">
         <Hello /> 
         <RaisedButton label="Default" />
         <img src={ imgPikachu } />
        </div>
       </MuiThemeProvider>
       ...
```

## material icons
* 這部分有兩種方法
  * svg icon
    * import material-ui 事先 build 好的 svg-icons
      * import ActionHome from 'material-ui/svg-icons/action/home';
  * font icon
    * 需要使用 material-design-icons
    * npm install --save material-design-icons
    * http://www.material-ui.com/#/components/font-icon
* app/app.jsx
```
  import MuiThemeProvider from 'material-ui/styles/MuiThemeProvider';
  import RaisedButton from 'material-ui/RaisedButton';
  import ActionHome from 'material-ui/svg-icons/action/home';
  import { red500, greenA200 } from 'material-ui/styles/colors';
      ...
      <MuiThemeProvider>
        <div className="app">
         <Hello /> 
         <RaisedButton label="Default" />
         <ActionHome color={ red500 } hoverColor={ greenA200 } />
         <img src={ imgPikachu } />
        </div>
      </MuiThemeProvider>
      ...
```

## for production
* 最後是 for production 的部分
* package.json 加入 prod 指令，方便日後使用
```json
  // add in scripts
  "prod": "PROD=1 wepback",
```
* webpack.config.js 加入 UglifyJsPlugin 和 DefinePlugin，以下為完整的檔案
```js
  var path = require('path');
  var CleanPlugin = require('clean-webpack-plugin');
  var ExtractPlugin = require('extract-text-webpack-plugin');
  var HtmlWebpackPlugin = require('html-webpack-plugin');
  var webpack = require('webpack');

  var plugins = [
    new CleanPlugin('build'),
    new ExtractPlugin('[name].[hash].css', { allChunks: true }),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'app/index.html'),
      filename: 'index.html'
    }),
    new webpack.optimize.CommonsChunkPlugin('vendor', 'vendor.[chunkhash].js'),
    new webpack.optimize.OccurenceOrderPlugin()
  ];

  if (process.env.PROD) {
    plugins.push(
      new webpack.optimize.UglifyJsPlugin({
        include: /\.min\.js$/,
        compress: {
          warnings: false,
          dead_code: true,
        }
      }),
      new webpack.DefinePlugin({
        'process.env': {
          'NODE_ENV': JSON.stringify('production'),
        },
      })
    );
  }

  module.exports = {
    entry: {
      app: path.resolve(__dirname, 'app/app.jsx'),
      vendor: ['react', 'react-dom', 'material-ui'],
    },
    output: {
      path: path.resolve(__dirname, 'build'),
      filename: '[name].[chunkhash].js',
    },
    module: {
      preLoaders: [
        {
          test: /\.jsx$/,
          loader: 'eslint-loader',
        }
      ],
      loaders: [
        {
          test: /\.jsx$/,
          exclude: /node_modules/,
          loader: 'babel-loader',
          query: {
            presets: ['es2015', 'stage-0', 'react']
          }
        },
        {
          test: /\.scss$/,
          loader: ExtractPlugin.extract('style', 'css!sass')
        },
        {
          test: /\.(jpg|git|png)$/,
          loader: 'url?limit=8192&name=images/[name].[hash:20].[ext]'
        },
        {
          test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
          loader: 'url?limit=8192&name=images/[name].[hash:20].[ext]'
        },
        {
          test:  /\.woff(2)?(\?v=[0-9]\.[0-9]\.[0-9])?$/,
          loader: 'url?limit=8192&minetype=application/font-woff2&name=images/[name].[hash:20].[ext]'
        }
      ]
    },
    resolve: {
      extensions: ['', '.js', '.jsx']
    },
    plugins: plugins,
    devtool: 'eval',
    devServer: {
      contentBase: 'build',
      host: '0.0.0.0',
    },
  };
```

## 備註
* CommonsChunkPlugin config
  * filename: 
  * children: true
  * minSize: 10 * 1000 // 10k
  * minChunks: 5
* No Errors Plugin
  * 此 Plugin 主要功能是在程式碼出錯時，不會將有錯誤的程式碼打包以確保打包完後的程式碼語法上的正確性。比較容易會碰到的問題是，當有使用 eslint-loader 來為 ES6 做語法檢查時，不管是 warning 或是 error 都會使 No Error Plugin 停止打包。
  * 同時使用此 Plugin 與 eslint-loader 的設定有好有壞，好處是你的程式碼必須乾淨到連 warning 都沒有才能正常打包。壞處則是檢查太嚴格，有些開發者可能不習慣這樣的模式。當你想要保有檢查 es6 語法功能卻又不想被這麼嚴苛的對待時，就把 No Errors Plugin 拿掉吧。
* Extract Text Plugin
  * 大家都知道在 webpack 中 CSS 是可以被 require() 的，webpack 會自動生成一個 style 標籤並加入到 html 的 head 中。
  * 但是在發佈時，我們可能只希望有一個被打包過後的 css 檔。這時 Extract Text Plugin 就能幫助我們完成這項任務
* OccurenceOrderPlugin 
* Multiple lazy loaded entries
* UglifyJsPlugin
  * UglifyJsPlugin 的功能正如其名，它會壓縮 javascript 檔。
  * 在 webpack -p 也就是 production mode 時會自動執行。
  * 這邊特別提出來的原因在於有時會在壓縮時有一些多餘的警告 (Side effects in initialization of unused variable define) 跑出來，如果你也像我一樣龜毛不喜歡看到警告的話，你可以在 webpack.config.js 內做以下設定：
```js
  plugins: [
    new Webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
```
  * webpack -p
* DefinePlugin
* PureRender

## Reference
* http://qiita.com/uryyyyyyy/items/6d7d29499efbca8c618e
* https://rhadow.github.io/2015/04/02/webpack-workflow/
* http://www.ruanyifeng.com/blog/2016/01/babel.html
* https://segmentfault.com/a/1190000005614604
* https://wohugb.gitbooks.io/webpack/content/plugins/html_webpack_plugin.html
* https://rhadow.github.io/2015/05/30/webpack-loaders-and-plugins
* http://blog.kkbruce.net/2015/10/webpack.html#.V7MUdFt94dU
* http://gold.xitu.io/entry/5767a975df0eea0062ffe193
* https://zhuanlan.zhihu.com/p/20914387
* [【webpack】的基本工作流程](https://medium.com/html-test/webpack-%E7%9A%84%E5%9F%BA%E6%9C%AC%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B-585f2bc952b9#.gop5lmgwk)
