# 如何校验提交信息

## husky

vben 文档里写到用 commitlint 校验提交信息，但没有提到用什么命令校验，什么时候校验，不过提到了如何关闭校验

> 在 .husky/commit-msg 内注释以下代码即可

husky 这个工具会在 git 流程触发对应 hook 脚本，也就是说，要校验提交信息需要 git 的 `commit-msg` hook 中进行，项目中的校验命令如下

```shell
#!/bin/sh
npx --no-install commitlint --edit "$1"
```

`#!/bin/sh` 特殊注释行，使用指定 shell 解释器执行脚本，加上了 vscode 里才有 shell 脚本的图标

`--edit` 指定一个提交信息文件来进行检查，例如：

```shell
commitlint --edit .git/COMMIT_EDITMSG
```

`$1` 表示脚本接收到的第一个参数，在 husky 的 `commit-msg` hook 中指的就是 `.git/COMMIT_EDITMSG`

## husky 配置

### 安装依赖

```shell
pnpm add --save-dev husky
```

### 初始化

```shell
pnpm exec husky init
```

## 配置 commitlint

### 安装依赖

```shell
pnpm add --save-dev @commitlint/{cli,config-conventional}
```

### 配置

根目录 `.commitlintrc.cjs`

```js
module.exports = {
  extends: ["@commitlint/config-conventional"],
};
```

## 完整配置

```js
const fs = require("fs");
const path = require("path");
const { execSync } = require("child_process");

const scopes = fs
  .readdirSync(path.resolve(__dirname, "src"), { withFileTypes: true })
  .filter((dirent) => dirent.isDirectory())
  .map((dirent) => dirent.name.replace(/s$/, ""));

const scopeComplete = execSync("git status --porcelain || true")
  .toString()
  .trim()
  .split("\n")
  .find((r) => ~r.indexOf("M  src"))
  ?.replace(/(\/)/g, "%%")
  ?.match(/src%%((\w|-)*)/)?.[1]
  ?.replace(/s$/, "");

/** @type {import('cz-git').UserConfig} */
module.exports = {
  ignores: [(commit) => commit.includes("init")],
  extends: ["@commitlint/config-conventional"],
  rules: {
    "body-leading-blank": [2, "always"],
    "footer-leading-blank": [1, "always"],
    "header-max-length": [2, "always", 108],
    "subject-empty": [2, "never"],
    "type-empty": [2, "never"],
    "subject-case": [0],
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "perf",
        "style",
        "docs",
        "test",
        "refactor",
        "build",
        "ci",
        "chore",
        "revert",
        "wip",
        "workflow",
        "types",
        "release",
      ],
    ],
  },
  prompt: {
    alias: {
      f: "docs: fix typos",
      r: "docs: update README",
      s: "style: update code format",
      b: "build: bump dependencies",
      c: "chore: update config",
    },
    customScopesAlign: !scopeComplete ? "top" : "bottom",
    defaultScope: scopeComplete,
    scopes: [...scopes, "mock"],
    allowEmptyIssuePrefixs: false,
    allowCustomIssuePrefixs: false,
    typesAppend: [
      { value: "wip", name: "wip:      work in process" },
      { value: "workflow", name: "workflow: workflow improvements" },
      { value: "types", name: "types:    type definition file changes" },
    ],
  },
};
```

## 自动生成提交信息

手动输入规范的提交信息十分繁琐，需要借助工具自动生成提交信息

过去使用 `commitizen` 交互式地生成提交信息，通过适配器改变交互方式，`cz-conventional-changelog` 就是最常用的一种适配器，但 vben 使用了一种叫 `cz-git` 的适配器，提供了更丰富灵活的配置，比如添加 emoji

使用 comitizen 的情况下是这样配置 cz-git 适配器的

`package.json`

```json
{
  "config": {
    "commitizen": {
      "path": "node_modules/cz-git"
    }
  }
}
```

配置

`.commitlintrc.js`

```js
/** @type {import('cz-git').UserConfig} */
module.exports = {
  prompt: {
    useEmoji: true,
  },
};
```

但是出现自带 `cz-git` 的提交信息生成工具 `czg`，属于 `commitizen` 的替代方案，不需要额外配置适配器，该项目就是使用了这个工具，生成命令如下

```shell
pnpm commit
```
