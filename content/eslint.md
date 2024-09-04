# 如何校验代码规范

## 手动校验代码

vben 提到要手动校验代码可以执行下面命令：

```shell
yarn run lint:eslint
```

上面命令说明执行了 `package.json` 中的 `lint:eslint` 脚本，看看该脚本执行了什么了命令

```shell
eslint --cache --max-warnings 0  \"{src,mock}/**/*.{vue,ts,tsx}\" --fix
```

`eslint` 静态代码检查工具，发现和修复代码问题

`--cache` 将检查过的文件[缓存](https://eslint.org/docs/v8.x/use/command-line-interface#--cache)起来，下次检查时只检查更改的文件，加快检查速度

`max-warning 0` 超过指定警告数量则检查失败，0 则表示不允许有任何警告

`\"{src,mock}/**/*.{vue,ts,tsx}\"`

- `{src,mock}` 检查这两个文件夹
- `/**/*` 递归检查文件夹下所有子文件夹和文件
- `.{vue,ts,tsx}` 检查指定文件拓展名

`--fix` 自动修复能修复的错误和警告，不能自动修复的输出到终端提示开发者手动修复

## 安装依赖

要执行上面命令的前提自然是安装对应命令行工具

```shell
pnpm add --save-dev eslint@8
```

因为 vben 使用的 eslint 版本是 8，但 eslint 的最新版是 9，两个版本的配置方式不太一样，所以这里安装 8

能使用命令行工具的前提是安装成功后在 `node_modules/.bin` 下出现对应的 shell 脚本可执行文件

但是这里出现了一个疑问，为什么 vben 的 `package.json` 中并没有对 `eslint` 的直接依赖，却出现了可执行文件？

这里涉及到[依赖提升](https://pnpm.io/zh/next/npmrc#%E4%BE%9D%E8%B5%96%E6%8F%90%E5%8D%87%E8%AE%BE%E7%BD%AE)的概念

因为某些直接依赖间接依赖了 `eslint`，可以通过在 `.npmr` 下配置 `public-hoist-pattern` 实现将间接依赖提升到根目录 `node_modules` 下，同理适用于可执行文件

```
public-hoist-pattern[]=*eslint*
```

`[]` 表示这个配置可以接收多个值

不过还是建议显式地手动安装依赖

## 配置

eslint 8 的[配置文件](https://eslint.org/docs/v8.x/use/configure/configuration-files#configuration-file-formats)有多种格式，vben 则通过 `.eslintrc.cjs` 配置, 看看配置

```js
module.exports = {
  root: true,
  extends: ["@vben"],
  rules: {
    "no-undef": "off",
  },
};
```

使用 `Commonjs` 模块规范

`root` 标记了 `eslint` 搜索配置时的停止点，`eslint` 检查某个文件时优先使用最近的配置文件，再向上一层搜索，同时会合并途中找到的所有配置文件，直到到达带有 `root: true` 的配置文件或根目录

`extends` 使用预设的配置，不需要手动配置规则，使用带有 `eslint-config-` 前缀的 `npm` 包时，可以省略前缀，即此处 `@vben` 预设的完整名称是 `@vben/eslint-config` 的 npm 包，还有两个自带的预设 `eslint:recommended` 和 `eslint:all`

`rules` 配置具体每个校验规则, 此处 `"no-undef": "off"` 表示关闭了对未声明变量的校验，但是为什么这里要关掉呢？正常来说开启不是很正常吗？

试试开启后校验会不会报错

```js{5}
module.xports = {
  root: true,
  extends: ["@vben"],
  rules: {
    "no-undef": "error",
  },
};
```

```shell
pnpm lint:eslint
```

校验后确实报错了，报错信息都来自一些类型声明，这些类型声明都没有通过 import 导入就直接使用，cmd 单击就会跳转到 `types/global.d.ts` 下，那么问题来了，为什么定义在 `global.d.ts` 可以直接使用？

## 全局类型

声明全局类型一种方式就是创建一个 `.d.ts` 声明文件，变量则使用 `declare` 声明，`interface` 和 `type` 则不需要 `declare`，同时要在 `tsconfig.json` 的 `include` 中包含该声明文件, 这样就能全局使用该声明文件定义的类型，不需要 `import` 和 `export`

这里需要注意的是，一旦声明文件中包含 `import` 和 `export`，该声明文件就被视为模块文件，所有 `declare`、`interface` 和 `type` 定义的类型无法全局共享，这时候就需要 `declare global` 语法，将原本需要全局共享的类型放进去就能重新实现全局共享的效果，有没有 `decalre` 都行

## @vben/eslint-config

前面提到 `@vben` 的完整名称应该是 `@vben/eslint-config` 的依赖

看看 `package.json`，就能发现该依赖来自 pnpm workspace，只要在当前项目中搜索一下 `name` 为 `@vben/eslint-config` 的 `package.json` 文件就找到对应源码

源码放在 `.internal/eslint-config` 下，表示属于该项目内部才使用到的包

根据[eslint 官网](https://eslint.org/docs/v8.x/extend/shareable-configs#creating-a-shareable-config)介绍每个预设其实就只是导出一个配置对象，那么就需要找到导出配置对象的入口文件，查看 `package.json` 就可以看到 `exports` 字段指定了 `index.ts` 的打包文件作为模块的主入口文件

完整配置如下

```ts
export default {
  env: {
    browser: true,
    node: true,
    es6: true,
  },
  parser: "vue-eslint-parser",
  parserOptions: {
    parser: "@typescript-eslint/parser",
    ecmaVersion: 2020,
    sourceType: "module",
    jsxPragma: "React",
    ecmaFeatures: {
      jsx: true,
    },
    project: "./tsconfig.*?.json",
    createDefaultProgram: false,
    extraFileExtensions: [".vue"],
  },
  plugins: ["vue", "@typescript-eslint", "import"],
  extends: [
    "eslint:recommended",
    "plugin:vue/vue3-recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended",
  ],
  rules: {
    "no-unused-vars": "off",
    "no-case-declarations": "off",
    "no-use-before-define": "off",
    "space-before-function-paren": "off",

    "import/first": "error",
    "import/newline-after-import": "error",
    "import/no-duplicates": "error",

    "@typescript-eslint/no-unused-vars": [
      "error",
      {
        argsIgnorePattern: "^_",
        varsIgnorePattern: "^_",
      },
    ],
    "@typescript-eslint/ban-ts-ignore": "off",
    "@typescript-eslint/ban-ts-comment": "off",
    "@typescript-eslint/ban-types": "off",
    "@typescript-eslint/explicit-function-return-type": "off",
    "@typescript-eslint/no-explicit-any": "off",
    "@typescript-eslint/no-var-requires": "off",
    "@typescript-eslint/no-empty-function": "off",
    "@typescript-eslint/no-use-before-define": "off",
    "@typescript-eslint/no-non-null-assertion": "off",
    "@typescript-eslint/explicit-module-boundary-types": "off",
    "vue/script-setup-uses-vars": "error",
    "vue/no-reserved-component-names": "off",
    "vue/custom-event-name-casing": "off",
    "vue/attributes-order": "off",
    "vue/one-component-per-file": "off",
    "vue/html-closing-bracket-newline": "off",
    "vue/max-attributes-per-line": "off",
    "vue/multiline-html-element-content-newline": "off",
    "vue/singleline-html-element-content-newline": "off",
    "vue/attribute-hyphenation": "off",
    "vue/require-default-prop": "off",
    "vue/require-explicit-emits": "off",
    "vue/html-self-closing": [
      "error",
      {
        html: {
          void: "always",
          normal: "never",
          component: "always",
        },
        svg: "always",
        math: "always",
      },
    ],
    "vue/multi-word-component-names": "off",
  },
  globals: { defineOptions: "readonly" },
};
```

上面配置的重点是用到了三个插件 `eslint-plugin-vue`、`eslint-plugin-import` 和 `@typescript-eslint/eslint-plugin`，这些插件提供了额外规则校验 vue、模块导入语法和 typescript，但要辅以 `vue-eslint-parser` 和 `@typescript-eslint/parser` 解析器对非 js 文件进行解析才能校验

要注意的是，要关闭和 prettier 和 eslint 的冲突，需要 `eslint-plugin-prettier` 和 `eslint-config-prettier`
