---
title: rollup 源码解析
categories:
  - FE tools
  - rollup 2.69.2
abbrlink: '408e8927'
date: 2022-04-18 21:56:17
---

首页是对外暴露的 `rollup` 方法，它调用的是 `rollupInternal` 方法。

```ts
export default function rollup(
  rawInputOptions: GenericConfigObject
): Promise<RollupBuild> {
  return rollupInternal(rawInputOptions, null);
}
```

至于 `rollupInternal`，它负责生成 `graph`，然后调用 `graph.build()` 开始对源文件进行解析，最后生成 `result`。在 `result` 上具有 `generate` 和 `write` 两个输出编译后代码的方法。

```ts
export async function rollupInternal(
  rawInputOptions: GenericConfigObject,
  watcher: RollupWatcher | null
): Promise<RollupBuild> {
  const { options: inputOptions, unsetOptions: unsetInputOptions } =
    await getInputOptions(rawInputOptions, watcher !== null);

  const graph = new Graph(inputOptions, watcher);

  const useCache = rawInputOptions.cache !== false;

  await graph.build();

  const result: RollupBuild = {
    cache: useCache ? graph.getCache() : undefined,
    async close() {
      // xxx
    },
    closed: false,
    async generate() {
      // xxx
    },
    watchFiles: Object.keys(graph.watchFiles),
    async write() {
      // xxx
    },
  };

  return result;
}
```

实际上 `generate` 和 `write` 的实现细节基本一致，都是调用的 `handleGenerateWrite` 函数，只是所传第一个参数不同罢了。

```ts
async function handleGenerateWrite(
  isWrite: boolean,
  inputOptions: NormalizedInputOptions,
  unsetInputOptions: ReadonlySet<string>,
  rawOutputOptions: GenericConfigObject,
  graph: Graph
): Promise<RollupOutput> {
  const bundle = new Bundle(
    outputOptions,
    unsetOptions,
    inputOptions,
    outputPluginDriver,
    graph
  );

  const generated = await bundle.generate(isWrite);

  if (isWrite) {
    await Promise.all(
      Object.values(generated).map((chunk) =>
        writeOutputFile(chunk, outputOptions)
      )
    );
  }

  return createOutput(generated);
}
```

如上所见可以看到 rollup 的运行是从创建 `Graph` 实例开始的，它上面会挂载三个重要属性：

- `pluginDriver`：管理 plugin。
- `acornParser`：生成 ast 语法树。
- `moduleLoader`：管理文件。

`Graph.generateModuleGraph` 方法会从入口文件开始，通过对源代码进行分析，从而梳理文件之间的依赖关系，逐一的为每个文件生成一个 `Module` 实例，然后登记在 `graph.moduleLoader.modulesById` 上。

在对源代码进行分析时，就是通过 `graph.acornParse` 生成 ast 语法树，然后再基于 rollup 自己定义在 src/nodes/shared 下的节点生成对应的实例，保存在 `module.ast` 上。在这一系列动作之后，已经知道了当前源文件依赖哪些其他文件，然后就可以顺着依赖关系再对其他文件进行同样的解析。

当然，在这一过程中，会在关键的节点调用 `graph.pluginDriver` 上的合适方法来抛出对应的钩子，触发用户声明要在编译过程中执行的 plugins。
