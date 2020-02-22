---
title: "React + Formik + Material-UI = 😭 (Part 1)"
date: 2020-01-23T23:47:40+08:00
categories: ["Coding"]
tags: ["React", "Formik", "Material-UI", "JavaScript"]
draft: true
---

2019 年的最後一個月在翻公司的就專案專案，由於是整個打掉重練，於是想改用 React 重新撰寫，以及用一些新的 Library ，例如 Material-UI ，以及 Formik 這種 Form Library 看看是否能減少上一個專案開發中出現的痛點？但在在過程中還是遇到了不少靈異現象。

# 先理解自己要做什麼？再動手下去做！

在用某個套件前，最好還是先閱讀官方文件了解套件的功能，評估自己需求會用到功能是否該套件有提供，可以的話思考~~通靈~~一下未來可能出現的需求這個套件是否能夠提供，再動手下去用。~~雖然是這樣講但我大部分時間是且戰且走。~~

{{< figure src="/2019-react-project/dab0a7c4a11752c29b5993efc02e0bcc.jpg" title="使用前請詳讀文件" >}}

## create-react-app

之前的專案是由台北資深的同事做好整個架構，然後再給我和新竹的同事們一起接手做下去。開發期間也有為了升級 Webpack 到 4.x 還有最佳化的需求去修改 Webpack （還有像是去改動 babel 的設定但這不是很重要），不過 Webpack 的設定檔很複雜又有分環境而有不同的設定檔，所以那個時候改得很辛苦。新的專案由於有時程壓力，不想花太多時間在改這些東西，加上新專案的環境只有 development 和 production 兩個，因此我最後使用了 [create-react-app](https://create-react-app.dev) 產生專案。

{{< figure src="/2019-react-project/create-react-app.png" title="Create React App offers a modern build setup with no configuration." >}}

產生專案很簡單，安裝完 `create-react-app` 後執行以下指令，馬上就能得到一個可以執行的 SPA: 

```bash
npx create-react-app my-app
cd my-app
npm start
```

馬上就可以進行開發了，[官方文件](https://create-react-app.dev/docs/getting-started)寫得十分詳細，舉凡 Styles (CSS Modules 、 SASS)、建制專案常用的套件、測試、部署應有盡有真的很方便。 Create React App 分成兩個部分：

* create-react-app: 產生專案 command line tool
* react-scripts: 開發相關的 dependency (Webpack 等等)

如果想要獲得新的 feature ，可以看官方 [Change Log](https://github.com/facebook/create-react-app/blob/master/CHANGELOG.md) 的更新指示。比方說我很想用 [Optional Chaining 和 Nullish Coalescing Operators](https://github.com/TC39/proposal-optional-chaining) 但我現在用的是 react-scripts 3.2.0 ，我升級到 3.3.0 就可以享受到這個功能了，根據官方文件，我只要更新 react-scripts 到 3.3.0 即可：

```bash
npm install --save --save-exact react-scripts@3.3.0
```

如果想要進一步掌控整個專案的話，也可以 [eject](https://create-react-app.dev/docs/available-scripts#npm-run-eject) 出所有設定檔自己改（但是這個指令一下下去就回不去了！），或者參考官方的[另外一套方法](https://auth0.com/blog/how-to-configure-create-react-app/)。

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

## Material-UI
先前的專案使用 React-Bootstrap ，不過我覺得它的 component 沒有那麼豐富也沒那麼漂亮，所以我花了很多時間在修~~魔改~~ UI 。當時看到 Material-UI 還蠻好看的就下定決心下個專案一定要用它，新專案也沒想那麼多就直接用下去了。用了才發現缺點也不少。

第一個缺點 component 的封裝不統一。比方說 `Chip` 有 `size` 屬性可以控制大小，但 `Avatar` 卻沒有，因此如果你想要得到一個比較小的 `Avatar` 就必須自己寫 CSS ：

```jsx
import React from "react";
import { makeStyles } from "@material-ui/core/styles";
import Avatar from "@material-ui/core/Avatar";

const useStyles = makeStyles(theme => ({
  root: {
    display: "flex",
    "& > *": {
      margin: theme.spacing(1)
    }
  },
  small: {
    width: theme.spacing(3),
    height: theme.spacing(3)
  },
  large: {
    width: theme.spacing(7),
    height: theme.spacing(7)
  }
}));

export default function ImageAvatars() {
  const classes = useStyles();

  return (
    <div className={classes.root}>
      <Avatar
        alt="Remy Sharp"
        src="/static/images/avatar/1.jpg"
        className={classes.small}
      />
      <Avatar alt="Remy Sharp" src="/static/images/avatar/1.jpg" />
      <Avatar
        alt="Remy Sharp"
        src="/static/images/avatar/1.jpg"
        className={classes.large}
      />
    </div>
  );
}
```

雖然 Material-UI 的 component 功能包山包海，但是這些功能需要寫更多的程式碼來實現（比方說 [Desponsive Drawer](https://material-ui.com/components/drawers/#responsive-drawer)），不過官方文件的範例程式碼很完整，只要好好當一個 Copy & Paste 工程師就可以實作出複雜的功能。

另外一個缺點則是 Form 的排版，其實官方文件並沒有寫要怎麼排版，所以我都是依靠 `Grid` 來達成，但是 Material-UI 的 `Grid` 並沒有 Row 、 Column 的概念，因此如果我想要達到以下的排版的話就需要花更多 Code 來達成：

```jsx
<form>
  <Grid container spacing={2}>
    <Grid item sm={4}>
      <TextField fullWidth helperText="Enter email" label="Email" />
    </Grid>
  </Grid>
  <Grid container spacing={2}>
    <Grid item sm={12}>
      <TextField
        fullWidth
        helperText="字數上限: 255 個字"
        label="Password"
        type="password"
      />
    </Grid>
  </Grid>
  <Button variant="contained" color="primary" type="submit">
    Submit
  </Button>
</form>
```

相對於 Material-UI ， React Bootstrap 對 Form 的封裝更好：

{{< figure src="/2019-react-project/react-bootstrap-form.png" title="React Bootstrap 的排版" >}}

我們可以學習 React Bootstrap 的做法將 Layout 封裝起來，增加程式碼的可讀性：

```jsx
import React from "react";
import Grid from "@material-ui/core/Grid";
import Button from "@material-ui/core/Button";
import TextField from "@material-ui/core/TextField";

function Form(props) {
  return <form {...props} />;
}

function FormRow(props) {
  return <Grid container spacing={2} {...props} />;
}

function FormCol(props) {
  return <Grid item {...props} />;
}

Object.assign(Form, {
  Row: FormRow,
  Col: FormCol
});

function MyForm() {
  return (
    <Form>
      <Form.Row>
        <Form.Col sm={4}>
          <TextField fullWidth helperText="Enter email" label="Email" />
        </Form.Col>
      </Form.Row>
      <Form.Row>
        <Form.Col sm={12}>
          <TextField
            fullWidth
            helperText="字數上限: 255 個字"
            label="Password"
            type="password"
          />
        </Form.Col>
      </Form.Row>
      <Button variant="contained" color="primary" type="submit">
        Submit
      </Button>
    </Form>
  );
}
```

## Formik


# 版本需要正確，不正確的話可能會有問題
