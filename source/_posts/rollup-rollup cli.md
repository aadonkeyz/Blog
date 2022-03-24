---
title: rollup cli
categories:
  - FE tools
  - rollup 2.69.2
abbrlink: de3a6b85
date: 2022-03-13 22:19:01
---

# 读取 command

```ts
import argParser from 'yargs-parser';

const command = argParser(process.argv.slice(2), {
  alias: commandAliases,
  configuration: { 'camel-case-expansion': false },
});
```

我的指令是配置在 package.json 中，为 `"build": "rollup -c"`，它对应的 `command` 变量为

```ts
command = {
  _: [],
  c: true,
  config: true,
};
```

# 生成 options

```ts
const { options } = await getConfigs(command);

/**
 * 分界线
 **/

async function getConfigs(
  command: any
): Promise<{ options: MergedRollupOptions[]; warnings: BatchWarnings }> {
  if (command.config) {
    // getConfigPath 在项目根目录下寻找 rollup.config.(mjs | cjs | ts | js)，然后返回对应文件的绝对路径
    const configFile = await getConfigPath(command.config);
    const { options, warnings } = await loadAndParseConfigFile(
      configFile,
      command
    );
    return { options, warnings };
  }
  return await loadConfigFromCommand(command);
}

async function loadAndParseConfigFile(
  fileName: string,
  commandOptions: any = {}
): Promise<{ options: MergedRollupOptions[]; warnings: BatchWarnings }> {
  // config 数组，我只配置了 rollup.config.ts，所以数组内容只有一项
  const configs = await loadConfigFile(fileName, commandOptions);
  const warnings = batchWarnings();
  try {
    const normalizedConfigs: MergedRollupOptions[] = [];
    // 对配置进行默认处理
    for (const config of configs) {
      const options = mergeOptions(config, commandOptions, warnings.add);
      await addCommandPluginsToInputOptions(options, commandOptions);
      normalizedConfigs.push(options);
    }
    return { options: normalizedConfigs, warnings };
  } catch (err) {
    warnings.flush();
    throw err;
  }
}
```

# 根据 options 进行 build

```ts
const { options, warnings } = await getConfigs(command);
for (const inputOptions of options) {
  await build(inputOptions, warnings, command.silent);
}

/**
 * 分界线
 **/

async function build(
  inputOptions: MergedRollupOptions,
  warnings: BatchWarnings,
  silent = false
): Promise<unknown> {
  const outputOptions = inputOptions.output;
  const bundle = await rollup(inputOptions as any);
  await Promise.all(outputOptions.map(bundle.write));
  await bundle.close();
}
```

很明显是先调用 `rollup` 生成 `bundle`，然后调用 `bundle.write` 生成产物。
