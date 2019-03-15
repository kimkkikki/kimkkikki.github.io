---
layout: post
title:  "Flask에 React Router 적용하기"
date:   2019-03-15 11:48:17 +0900
author: kimkkikki
categories: flask
comments: true
---
Flask에 React Router를 적용해봅시다. Flask는 기본적으로 Jinja2 Template을 지원하고 있지만, 저는 그저 REST API 용도로만 Flask를 활용하고, View는 전부 React에 맡기고 싶었습니다.

View에서 일어나는 모든일은 React에 맡기고 Flask는 REST API만 담당하여 둘을 완전히 분리해봅시다.

일단 기본적인 프로젝트 구조는 다음과 같이 구성할 예정입니다.

{% highlight shell %}
template/
    index.html
static/
api/
    api.py
web/
    app.js
    /containers
        First.js
        Second.js
app.py
package.json
webpack.config.js
requirements.txt
{% endhighlight %}

아주 기본적인 내용만 담고 있는 프로젝트 구조입니다. 하나씩 살펴봅시다. 먼저 **app.py**에 기본적인 Flask Application 코드를 작성합니다.

{% highlight python %}
from flask import Flask, render_template


app = Flask(__name__)


@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def web(path):
    return render_template('index.html')


if __name__ == '__main__':
    app.run(host='127.0.0.1')
{% endhighlight %}

어떠한 호출이 들어와도 index.html을 반환해주는 심플한 코드입니다. React Router를 사용하기 위한 준비작업이라고 보시면 됩니다. 이제 React Router를 설정해줍니다.

**package.json**을 다음과같이 구성해줍니다.
{% highlight json %}
{
  "name": "flask_react_router",
  "version": "1.0.0",
  "author": "kimkkikki",
  "devDependencies": {
    "@babel/core": "^7.2.2",
    "@babel/plugin-proposal-class-properties": "^7.3.0",
    "@babel/plugin-proposal-decorators": "^7.3.0",
    "@babel/polyfill": "^7.2.5",
    "@babel/preset-env": "^7.3.1",
    "@babel/preset-react": "^7.0.0",
    "babel-loader": "^8.0.5",
    "css-loader": "^2.1.0",
    "file-loader": "^3.0.1",
    "mini-css-extract-plugin": "^0.5.0",
    "postcss-loader": "^3.0.0",
    "sass-loader": "^7.1.0",
    "style-loader": "^0.23.1",
    "webpack": "^4.29.0",
    "webpack-cli": "^3.2.1"
  },
  "dependencies": {
    "react": "^16.7.0",
    "react-dom": "^16.7.0",
    "react-router-dom": "^4.3.1"
  }
}
{% endhighlight %}

package.json을 보면 현재 프로젝트에는 필요없는 dependency들이 있는데요. 언젠가 쓸일이 있을거니까 넘어갑시다.
**webpack.config.js**를 만들어 webpack 설정을 해줍니다.

{% highlight javascript %}
const path = require("path")
const MiniCssExtractPlugin  = require("mini-css-extract-plugin")

module.exports = {
    mode: "development",
    entry: [
        '@babel/polyfill',
        "./web/app.js"
    ],
    output: {
        filename: "flask_react_router.js",
        path: path.resolve(__dirname, "static"),
        publicPath: "/static/"
    },
    module: {
        rules: [
            {
                test: /\.(sa|sc|c)ss$/,
                use: [
                    MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            },
            {
                test: /\.m?js$/,
                exclude: /(node_modules|bower_components)/,
                use: [{
                    loader: "babel-loader",
                    options: {
                        presets: ["@babel/preset-env", "@babel/preset-react"],
                        plugins: [
                            ['@babel/plugin-proposal-decorators', {legacy: true}],
                            ['@babel/plugin-proposal-class-properties', {loose: true}]
                        ]
                    }
                }]
            },
            {
                test: /\.(png|jpg|ico|svg|eot|woff|woff2|ttf)$/,
                use: ['file-loader']
            }
        ]
    },
    plugins: [
        new MiniCssExtractPlugin({
            filename: "flask_react_router.css",
            chunkFilename: "[id].css"
        })
    ]
}
{% endhighlight %}

webpack.config.js를 보시면 css를 별도 파일로 나누고 있고, 빌드 완료된 js 파일을 /static/ 폴더에 넣는것을 볼 수 있습니다. 저는 es7을 사용하고 있으므로 관련 babel loader를 추가해 주었습니다.

이제 생성될 js와 css파일을 **index.html**에 넣어줍시다.

{% highlight html %}
{% raw %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="author" content="kimkkikki">
    <title>Flask React Router</title>

    <link rel="stylesheet" href="{{ url_for('static', filename='flask_react_router.css') }}">
</head>
<body>
<div id="root"></div>
</body>
<script type="text/javascript" src="{{ url_for('static', filename='flask_react_router.js') }}"></script>
</html>
{% endraw %}
{% endhighlight %}

이제 거의다 왔습니다. **app.js**를 다음과같이 구성합니다.

{% highlight jsx %}
import '@babel/polyfill'
import React from 'react'
import ReactDOM from 'react-dom'
import { BrowserRouter, Route } from 'react-router-dom'
import { First, Second } from './containers'

ReactDOM.render(
    <BrowserRouter>
        <Route exact path='/first' component={ First } />
        <Route exact path='/second' component={ Second } />
    </BrowserRouter>
    , document.getElementById('root')
);
{% endhighlight %}

uri가 /first와 일치하면 First Component를 불러주고, /second와 일치하면 Second Component를 불러주게 됩니다.

**First.js**와 **Second.js**를 다음과 같이 구성해줍니다.

{% highlight jsx %}
// First.js
import React, { Component } from 'react'

export default class First extends Component {
    render() {
        return (
            <div>Hello First</div>
        )
    }
}

// Second.js
import React, { Component } from 'react'

export default class Second extends Component {
    render() {
        return (
            <div>Hello Second</div>
        )
    }
}
{% endhighlight %}

이제 완성입니다.
터미널을 열고 webpack 빌드를 한 후 Flask를 실행해 줍시다.

{% highlight bash %}
webpack
python app.py
{% endhighlight %}

이제 브라우저로 localhost:5000/first, localhost:5000/second 로 접속하여 결과를 확인하면 됩니다.
