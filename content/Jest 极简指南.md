---
title: Jest 极简指南
tags:
  - 工程化
  - 测试
---
# 准备测试

安装 jest
```shell
pnpm add jest -D
```

要测试的函数
```js
const englishCode = "en-US";
const spanishCode = "es-ES";

function getAboutUsLink(language) {
  switch (language.toLowerCase()) {
    case englishCode.toLowerCase():
      return "/about-us";
    case spanishCode.toLowerCase():
      return "/acerca-de";
  }
  return "";
}

module.exports = getAboutUsLink;
```

测试示例，用 *.spec.js 或 *.test.js 后缀
```js
const getAboutUsLink = require("./index");

test("Returns about-us for english language", () => {
  expect(getAboutUsLink("en-US")).toBe("/about-us");
});
```

# 运行测试

安装测试运行工具
```shell
pnpm add jest-cli -D
```

配置 package.json 脚本
```json
{
  "scripts": {
    "test": "jest"
  }
}
```

跑测试
```shell
pnpm test
```

# 覆盖率

查看测试是否完善，追求 100%，但不一定要达到

配置脚本
```json
{
  "scripts": {
    "test": "jest --coverage"
  }
}
```

运行之后会生成 coverage 文件夹，打开 coverage/locv-report/index.html 可以看到测试覆盖情况

![image](https://github.com/ReinerLau/testing-guide/assets/103234074/560934d8-de98-4d35-9921-b0d97a2c7808)

# 拓展
安装[拓展](https://marketplace.visualstudio.com/items?itemName=Orta.vscode-jest)可以方便运行测试，每次保存文件都会重新运行一次测试

![image](https://github.com/ReinerLau/testing-guide/assets/103234074/293bc225-5503-4d86-a377-a6c37c0eb76d)

cmd + shift + p 搜索 toggle coverage 并选择会打开开关，在文件中高亮显示测试未覆盖到的代码行

![image](https://github.com/ReinerLau/testing-guide/assets/103234074/4d18bab6-f6f2-4865-9c1d-176169aebfa8)