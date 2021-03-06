---
title: module
date: 2021-09-13 02:09:42
category: javascript
thumbnail: { thumbnailSrc }
draft: false
---

# module

Javascript에 모듈 시스템은 참 다양하다.
node.js 환경에서 사용가능한 모듈 시트템과 브라우저에서 사용가능한 모듈시스템이
구분되며, 이에 따라 헷갈리기 쉬운 모듈 개념을 정리하는 시간을 가져보려고 한다.

## 1. node 환경의 모듈 시스템

### 1.1. commonJS

ecmascrip의 표준 모듈은 아니지만 node.js 환경에서 채택한 **기본** 모듈 시스템이다.

#### module.exports

```js
function add(a, b) {
  return a + b;
}

module.exports = add;
```

모듈을 외부에서 사용하려면 "module" 전역 네임스페이스의 exports 프로퍼티에 함수를 할당한다.

#### require

```js
const add = require('./add');
console.log(add(1, 2));
```

require 함수와 인수로 경로를 입력하면 모듈을 불러올 수 있다.(확장자 생략 가능)

### 1.2. ESM

ES6(ECMA2015+) 부터 지원하는 표준 모듈 시스템이다.
node 환경에서는 package.json 파일에서 `type: "module"` 프로퍼티를 설정하면 그 때 부터는 모듈 시스템이 import export로 변경된다. 이때 commonJS 방식은 사용할 수 없다.

#### export

```js
export function add(a, b) {
  return a + b;
}
```

ESM은 모듈을 외부에서 사용하기 위해 export 키워드를 사용한다.

**default export**
파일 내에 단 하나의 함수만 기본 모듈로 제공하고 싶은 경우 default 키워드를 사용할 수 있다.

```js
export function plus(a, b) {
  return a + b;
}

export function minus(a, b) {
  return a - b;
}

export default function add(a, b) {
  return a + b;
}
```

#### import

```js
import add, { plus, minus } from './add.js';
...
```

import 키워드로 모듈을 불러올 수 있다.
단 from 뒤의 모듈 경로는 반드시 파일의 **확장자 단위까지 포함한 완전한 형태**로 입력해야 한다. 생략이 불가능하다.

## 2. 브라우저 환경의 모듈 시스템

브라우저 환경에서는 ESM 방식만을 지원한다. 하지만 현대에는 React와 같은 프레임워크와 더불어 test 환경을 node.js 에서 구동시키는데, 이 같은 경우 브라우저와 node.js 환경에서 지원하는 모듈 시스템이 다르기 때문에 문제가 발생할 수 있다.

이 때는 package.json 에서 `type: module` 프로퍼티를 설정해주면 문제가 해결되지만, ESM은 모든 모듈에 경로를 확장자까지 포함하여 기입해야함으로 불편한 점이 많다.

편의성을 위해 webpack은 다음과 같은 문법을 ESM 방식으로 변환한다.

```js
import add from './add'; // import add from './add.js'
import add from './directory/'; // import add from './directory/index.js'
```

## 3. 응용

### 3.1. webpack 설정

#### ESM 방식(비권장)

webpack.config.js 파일을 root 경로에 만들어주고 다음과 같이 입력한다.

```js
import path from 'path';
const __dirname = path.resolve(); // ESM 방식은 __dirname이 없다.

export default {
  // module.exports
  entry: ['./test.js'],
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
  },
};
```

**주의할 점**

- package.json의 모듈 타입 설정에 따라 webpack 설정도 export 혹은 module.exports 문법을 따라야한다.
- commonJS 문법의 경우 \_\_dirname을 기본적으로 제공하지만, ESM 방식은 지원하지 않기 때문에 따로 설정해 주어야 한다.
- ESM 문법으로 설정할 경우, 무조건 **ESM 문법만을 따라야 하기 때문에 모듈문법을 유연하게 사용할 수 있는 webpack의 장점을 살릴 수 없다**.

### 권장 설정

1. node.js 의 기본 모듈 설정인 **commonJS**를 따른다.
2. import export 문법을 사용하여 모듈을 유연하게 로드한다.
3. webpack은 js를 번들할 때, 내부적으로 브라우저가 사용가능한 모듈 문법으로 변환해준다.
4. babel을 사용하여 크로스 브라우징을 한다.

### 3.2. jest 설정

위와같이 webpack 모듈 방식을 사용할 경우, require문을 사용하지 않으면 jest 실행시 SyntaxError가 발생한다. jest가 node 환경에서 실행되니 당영한 결과다.

이럴 경우에는, 루트경로에 .babelrc 파일을 따로 만들어서 node.js 환경에서 실행될 때는 별도의 모듈 시스템이 적용되도록 분리할 수 있다.

```js
{
  "presets": [
    ["@babel/preset-env", {
      "loose": true,
      "modules": "commonjs", // <= 이부분이 중요하다. auto 혹은 commonjs로 설정
      "targets": [
        "defaults",
        "> 1% in KR",
        "not dead", // 1년이상 브라우저 관리가 되지 않은 것 dead라고 판단.
        "ie 10"
      ]
    }]
  ]
}
```
