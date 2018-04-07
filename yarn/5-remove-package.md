# 移除包

#### `yarn remove <package...>`

移除后, 此项目上工作的其他开发者可以运行 yarn install 来同步他们的 node_modules 目录为新的依赖集。

当你移除一个包时，它被从所有类型的依赖里移除：dependencies、devDependencies 等等。

> 注意：yarn remove 总是会更新你的 package.json 和 yarn.lock。 这可以确保不同的开发人员在同一个项目上得到相同的依赖集。 不可能禁止这个行为。
>
>注意：`yarn remove <package> --<flag>` 使用与 yarn install 命令相同的 flag。
