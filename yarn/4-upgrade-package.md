# 升级包

#### 1. `yarn upgrade [package | package@tag | package@version | @scope/]... [--ignore-engines] [--pattern]`

指定包名称时，将只升级这些包。未指定包名称时，将升级所有依赖项。

**[package]:** 升级到最新版本

**[package@tag]:** 将升级到该标签的版本, 标签会出现在 package.json 里.

**[package@version]:** 将升级到指定版本.你可以使用任何[语义版本](https://yarnpkg.com/zh-Hans/docs/dependency-versions#toc-semantic-versioning)的版本号或版本范围。

**--ignore-engines:** 可用于跳过引擎检查

**yarn upgrade --pattern <pattern>** 将升级所有匹配此模式的包。

如:
```
yarn upgrade --pattern gulp
yarn upgrade left-pad --pattern "gulp|grunt"
yarn upgrade --latest --pattern "gulp-(match|newer)"
```

#### 2. `yarn upgrade [package]... --latest|-L [--caret | --tilde | --exact] [--pattern]`

**-L:** 忽略版本范围. 升级到最新版本.

package.json 会从新更新版本范围限定。

默认情况下，package.json中的现有范围说明符 (以下之一：^，〜，<=，>或确切版本) 将被重用.

否则，它将被更改为插入符号（^）。其中一个标志--caret，--tilde或--exact可用于明确指定范围。

**--caret:** 版本限定为

**--tilde:** 版本限定为 **次版本**

**--exact:** 版本限定为 **精确版本**

#### 3. `yarn upgrade (--scope|-S) @scope [--latest] [--pattern]`

**--scope @scope/:** 当指定范围时，只有以该范围开头的包才会升级。范围必须以“@”开头。

**--latest:** 忽略版本范围, 安装到最新版本
