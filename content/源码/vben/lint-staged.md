# 校验暂存区

## lint-staged

[前面](https://reinerlau.github.io/blog/sourcecode/vben/commitLint.html#husky) 提到过 husky 的概念，同理可以像校验提交信息一样在代码提交时校验代码规范，只不过要考虑一个问题，就是 eslint 校验时需要传递要校验的文件夹或文件作为参数，也就是说，有可能校验的文件可能和上一次一样没有做任何更改，白白浪费了校验的时间，正常情况下，暂存区中的文件其实就发生了变更的文件，那有没有办法只对暂存区文件做校验呢，有的，就是 lint-staged

## 配置

### 安装依赖

```shell
pnpm add --save-dev lint-staged
```

### 配置

`package.json`

```json
{
  "lint-staged": {
    "*.{js,ts}": "eslint"
  }
}
```

`key` 是匹配对应文件的语法，`value` 是对匹配文件的校验命令

### hook

在配置了 husky 的前提下

创建 `.husky/pre-commit` hook

```shell
#!/bin/sh
pnpm exec lint-staged
```

这样就实现提交代码时只对暂存区文件做代码规范校验

## 完整配置

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["prettier --write", "eslint --fix"],
    "{!(package)*.json,*.code-snippets,.!(browserslist)*rc}": [
      "prettier --write--parser json"
    ],
    "package.json": ["prettier --write"],
    "*.vue": ["prettier --write", "eslint --fix", "stylelint --fix"],
    "*.{scss,less,styl,html}": ["prettier --write", "stylelint --fix"],
    "*.md": ["prettier --write"]
  }
}
```
