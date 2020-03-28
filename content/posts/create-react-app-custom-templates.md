---
title: "create-react-app 進階功能: Custom Templates"
date: 2020-03-28T22:10:47+08:00
categories: ["Coding"]
tags: ["React", "JavaScript"]
draft: false
---

之前提到 create-react-app 可以幫助開發者快速產生一個 React 專案，而 Custom Templates 功能可以根據樣板讓新的專案長出你希望的架構和安專好相依性的套件。雖然不是常常有機會開發新專案，但是如果能馬上產生立即可以開發的專案，花點時間寫 template 是很有幫助的。

# Get Start!

首先這個功能從 `react-scripts@3.3.0` 以後才有，所以使用前請先注意你的 create-react-app 版本（如果是直接用 npx 執行就不用煩惱這個問題）。

當我們在執行 `npx create-react-app my-app` 時，其實是用官方預設的 template 去產生專案。官方的 template 有以下這兩個：

* [cra-template](https://github.com/facebook/create-react-app/tree/master/packages/cra-template)
* [cra-template-typescript](https://github.com/facebook/create-react-app/tree/master/packages/cra-template-typescript)

你可以參考官方的 template 去建構出自己的 template 。

# Structure

template 基本架構如下：

```
cra-template-[template-name]/
  README.md (for npm)
  template.json
  package.json
  template/
    README.md (for projects created from this template)
    gitignore
    public/
      index.html
    src/
      index.js (or index.tsx)
```

其中最重要的有 template.json 檔案跟 template 資料夾：

## template 資料夾
在這邊放的檔案和資料夾會在執行 `npx create-react-app my-app` 的過程中複製到 `my-app` 專案。這個功能相當好用，我可以不用事後花老半天新增資料夾和檔案。例如我習慣將 `.jsx` 根據他是 container component 還是 presentational component 分別放在 `src/containers` 和 `src/components` ，然後 `components` 又可以根據該 component 的顆粒度分別放在 `elements` 、 `blocks` 、 `segments` 、 `layouts` 和 `pages` 等。還有一些基本的設定檔案等等我都可以放在 template 資料夾。以下是我的範例 template 資料夾目錄：

```
template
├── README.md
├── deploy.sh
├── gitignore
├── jsconfig.json
├── public
│   ├── favicon.ico
│   ├── index.html
│   ├── logo192.png
│   ├── manifest.json
│   └── robots.txt
└── src
    ├── components
    │   ├── blocks
    │   ├── elements
    │   ├── layouts
    │   ├── pages
    │   └── segments
    ├── config
    │   ├── base.js
    │   ├── development.js
    │   ├── index.js
    │   ├── production.js
    │   └── stage.js
    ├── constants
    ├── containers
    │   └── application
    │       ├── App.css
    │       ├── index.jsx
    │       ├── index.test.js
    │       ├── logic.js
    │       ├── selector.js
    │       └── slice.js
    ├── images
    │   ├── logo.svg
    ├── index.js
    ├── reducers.js
    ├── routes.js
    ├── serviceWorker.js
    ├── setupTests.js
    ├── store.js
    └── utils
```

## template.json
template 的設定檔。目前只支援 `package` 這個 key ，在 `package` 底下可以寫任何你想要加到新專案底下的 `package.json` 的設定，比方說 `dependencies` 或是 `scripts` 等等。 而且 `dependencies` 和 `devDependencies` 的套件會在你初始化專案的時候一並安裝。以下是我的範例 template.json ：

```json
{
  "package": {
    "dependencies": {
      "@testing-library/react": "^9.3.2",
      "@testing-library/jest-dom": "^4.2.4",
      "@testing-library/user-event": "^7.1.2",
      "@reduxjs/toolkit": "^1.2.1",
      "axios": "^0.19.0",
      "prettier": "^1.19.1",
      "react-router-dom": "^5.1.2",
      "react-redux": "^7.1.3",
      "redux": "^4.0.5",
      "redux-thunk": "^2.3.0",
      "reselect": "^4.0.0"
    },
    "scripts": {
      "lint": "eslint ./src",
     "prettier": "prettier --check src/**/*.{js,jsx,ts,tsx,json,css,scss,md}"
    },
    "eslintConfig": {
      "extends": "react-app"
    },
    "husky": {
      "hooks": {
        "pre-commit": "lint-staged"
      }
    },
    "lint-staged": {
      "src/**/*.{js,jsx,ts,tsx,json,css,scss,md}": "prettier --write",
      "src/**/*.{js,jsx,ts,tsx}": "eslint ./src --fix-dry-run"
    },
    "devDependencies": {
      "chai": "^4.2.0",
      "eslint": "^6.8.0",
      "eslint-config-prettier": "^6.10.1",
      "eslint-plugin-prettier": "^3.1.2",
      "eslint-plugin-react": "^7.19.0",
      "husky": "^4.2.3",
      "lint-staged": "^10.0.9",
      "pre-commit": "^1.2.2",
      "redux-devtools": "^3.5.0"
    }
  }
}
```

以上簡單安裝了 redux 、 axios 、 prettier 、 eslint 等等，以及我希望所有開發者在 commit 前可以自動先將 code format 過一次，以及進行 eslint 檢查，這樣我們的 coding style 才會一致。

# Create Project from Custom Templates

執行以下指令就能使用你的 custom template 產生新專案：

```bash
npx create-react-app my-app --template file:/path/to/your/template
```

如果你的 custom template 有上傳到 npm ，可以改用以下指令：

```bash
npx create-react-app my-app --template [template-name]
```

# Reference

* [Custom Templates](https://create-react-app.dev/docs/custom-templates/)