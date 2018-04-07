# 安装依赖包

```
yarn add [package-name]
```

命令默认使用 ^ 范围。

会安装一个包, 并且声明在 `dependencies` 字段下面.

这个命令会更新`package.json`和`yarn.lock`文件，以便使本项目的其他开发者可以使用`yarn`或者`yarn install`来安装相同的依赖。

**指定版本号添加**

- `yarn add package-name` 会安装 latest 最新版本。
- `yarn add package-name@1.2.3` 会从 registry 里安装这个包的指定版本。
- `yarn add package-name@tag` 会安装某个 “tag” 标识的版本（比如 beta、next 或者 latest）。

**从不同路径添加**

- `yarn add package-name` 从 npm registry 里安装包，除非你在 package.json 指定了其它 registry。


- `yarn add file:/path/to/local/folder` 从本地文件系统里安装一个包，可以用这种方式来测试还未发布的包。


- `yarn add file:/path/to/local/tarball.tgz` 安装一个 gzipped 压缩包，此格式可以用于在发布之前分享你的包。


- `yarn add <git remote url>` 从远程 git repo 里安装一个包。


- `yarn add <git remote url>#<branch/commit/tag> `从一个远程 git 仓库指定的 git 分支、git 提交记录或 git 标签安装一个包。


- `yarn add https://my-project.org/package.tgz` 用一个远程 gzipped 压缩包来安装。

#### 全局安装

```
yarn global add <package...>
```

## 命令

```
yarn add <package...>

yarn add <package...> [--dev/-D]

yarn add <package...> [--peer/-P]

yarn add <package...> [--optional/-O]

yarn add <package...> [--exact/-E]

yarn add <package...> [--tilde/-T]
```

#### `yarn add <package...>`
安装包, 声明在 `dependencies` 里.


#### `yarn add <package...> [--dev/-D]`
安装包, 声明在 `devDependencies` 里.

#### `yarn add <package...> [--peer/-P]`
安装包, 声明在 `peerDependencies` 里.
这个包是 需要开发者预先安装的.

#### `yarn add <package...> [--optional/-O]`
安装包, 声明在 `optionalDependencies` 里.
这个包是选装的.

#### `yarn add <package...> [--exact/-E]`
安装包, 以后这个包只接受当前所指定的精确版本.

#### `yarn add <package...> [--tilde/-T]`
安装包的次要版本里的最新版.
yarn add foo@1.2.3 --tilde 会接受 1.2.9，但不接受 1.3.0。
