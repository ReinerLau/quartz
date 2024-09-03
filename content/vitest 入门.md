---
title: Vitest 入门
tags:
  - 工程化
  - 测试
---
## 安装

```shell
pnpm add -D vitest
```

## 测试示例

`sum.ts`

```ts
export function sum(a, b) {
  return a + b;
}
```

`sum.test.ts`

```ts
import { expect, test } from "vitest";
import { sum } from "./sum";

test("adds 1 + 2 to equal 3", () => {
  expect(sum(1, 2)).toBe(3);
});
```

## 脚本

```json
{
  "scripts": {
    "test": "vitest"
  }
}
```

## 运行

```shell
pnpm test
```

## 拓展

vitest 和 jest 一样有对应的[VSCode拓展](https://marketplace.visualstudio.com/items?itemName=vitest.explorer)方便运行测试，查看测试运行情况

