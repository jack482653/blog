---
title: "React 2019 Late: 建置專案利器 create-react-app"
date: 2020-02-01T15:09:32+08:00
categories: ["Coding"]
tags: ["React", "JavaScript"]
draft: false
---

2019 年的最後一個月在翻公司的舊專案，改用 React 重新撰寫。最近終於完成了，趁著記憶猶新來記錄我這次開發專案的心得。

自從上個專案到現在過了蠻長一段時間， React 從 16.3 一口氣來到 16.8 （更新的速度真可怕），所以我決定一鼓作氣全部用最新版。反正新專案嘛，沒有包袱 ✌️。順便也用了一些新的 Library ，例如 Material-UI ，以及 Formik 這種 Form Library 看看是否能減少上一個專案開發中出現的痛點？但在在過程中還是遇到了不少靈異現象。

# Get Started!

之前的專案是由台北資深的同事做好整個架構，然後再給我和新竹的同事們一起接手做下去。開發期間也有為了升級 Webpack 到 4.x 還有最佳化的需求去修改 Webpack （還有像是去改動 babel 的設定但這不是很重要），不過 Webpack 的設定檔很複雜又有分環境而有不同的設定檔，所以那個時候改得很辛苦 😭。

順帶一提，去年年中參加 JCConf ，看到大神[龍哥 Josh Long](https://twitter.com/starbuxman) 用 [Spring Initializr](https://start.spring.io) 三兩下就把 Spring 專案產生出來了。可以選擇開發 Java 版本、 Spring 版本、相依的套件等等，真的是太讚了。下次有新的後端專案我也想用這個。

拉回來，新的專案由於有時程壓力，不想花太多時間在改這些東西，加上新專案的環境只有 development 和 production 兩個，因此我最後使用了 [create-react-app](https://create-react-app.dev) 產生專案。

{{< figure src="/2019-react-project/create-react-app.png" title="Create React App offers a modern build setup with no configuration." >}}

# create-react-app

產生專案很簡單，安裝完 `create-react-app` 後執行以下指令，馬上就能得到一個可以執行的 SPA: 

```bash
npx create-react-app my-app
cd my-app
npm start
```

馬上就可以進行開發了。產生的專案結構如下：

```
my-app
├── README.md
├── node_modules
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── logo512.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    ├── serviceWorker.js
    └── setupTests.js
```

`npm run build` 後產生的檔案會放在 `my-app/build` 資料夾下。

[官方文件](https://create-react-app.dev/docs/getting-started)寫得十分詳細，舉凡 Styles (CSS Modules 、 SASS)、建制專案常用的套件、測試、部署應有盡有真的很方便。

## 升級

Create React App 分成兩個部分：

* create-react-app: 產生專案 command line tool
* react-scripts: 開發相關的 dependency (Webpack 等等)

如果想要獲得新的 feature ，可以看官方 [Change Log](https://github.com/facebook/create-react-app/blob/master/CHANGELOG.md) 的更新指示。比方說我很想用 [Optional Chaining 和 Nullish Coalescing Operators](https://github.com/TC39/proposal-optional-chaining) 但我現在用的是 react-scripts 3.2.0 ，我升級到 3.3.0 就可以享受到這個功能了，根據官方文件，我只要更新 react-scripts 到 3.3.0 即可：

```bash
npm install --save --save-exact react-scripts@3.3.0
```

這個比我自己在那邊 npm 一個一個更新 package 強大太多了， npm 的套件升級真的很痛。

如果想要進一步掌控整個專案的話，也可以 [eject](https://create-react-app.dev/docs/available-scripts#npm-run-eject) 出所有設定檔自己改（但是這個指令一下下去就回不去了！），或者參考官方的[另外一套方法](https://auth0.com/blog/how-to-configure-create-react-app/)。

## 不同環境的 profile

最後到部署發現一個問題， `npm run build` 永遠只會產生 production 的版本，我沒辦法指定要 development 還是 production 的 config 。最後我的解決方案是參考 [issue 4071](https://github.com/facebook/create-react-app/issues/4071) 的方式額外弄一個環境變數：

```bash
REACT_APP_DIST_ENV=$ENV npm run build || exit 1
```

這樣一來我可以在 `config/index.js` 決定要載入哪個環境的 config ：

```js
// config/index.js
let config = {};

switch (process.env.REACT_APP_DIST_ENV) {
  case 'production':
    config = require('./production').default;
    break;
  default:
    config = require('./development').default;
    break;
}

export default config;
```

```js
// config/development.js
import baseConfig from './base';

const config = {
  backendApiRoot: 'https://beta-api.backend/rest',
  appEnv: 'development'
};

export default Object.freeze(Object.assign({}, baseConfig, config));
```

```js
// config/production.js
import baseConfig from './base';

const config = {
  backendApiRoot: 'https://prod-api.backend/rest',
  appEnv: 'production'
};

export default Object.freeze(Object.assign({}, baseConfig, config));
```

接著在其他程式碼 import config 後直接使用：

```js
import config from 'config';

export const api = (options) => _api(config.backendApiRoot, options);

// your _api() code ...
```

這樣一來，我可以在 `production.js` 和 `development.js` 指定不同的 `backendApiRoot` 的值，然後根據環境讓前端打 API 到 開發環境或是生產環境的伺服器。