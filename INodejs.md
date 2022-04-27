nodejs
===



## nodejs

### 基本安装

```sh
# Using Ubuntu
curl -fsSL https://deb.nodesource.com/setup_current.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 使用nvm

安装nvm

```
git clone https://github.com/nvm-sh/nvm.git
mv nvm ~/.nvm
```

add code to .profile

```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" 
```



### nodejs

```
nvm install v16.13.1
nvm install --lts
```

### npm

```
nvm install-latest-npm
```



## YARN

### install

```sh
npm install --global yarn
```

add config to .profile

```sh
#nodejs
NPM_HOME=/TOOLS/npm
PATH=$NPM_HOME/node_modules/bin:$PATH
```

```sh
yarn config set cache-folder /TOOLS/yarn/cache
```

### 基本命令

#### yarn add

`yarn add <package...>`

This will install one or more packages in your [`dependencies`](https://classic.yarnpkg.com/en/docs/dependency-types#toc-dependencies).

`yarn add <package...> [--dev/-D]`

Using `--dev` or `-D` will install one or more packages in your [`devDependencies`](https://classic.yarnpkg.com/en/docs/dependency-types#toc-dev-dependencies).

`yarn add <package...> [--peer/-P]`

Using `--peer` or `-P` will install one or more packages in your [`peerDependencies`](https://classic.yarnpkg.com/en/docs/dependency-types#toc-peer-dependencies).

`yarn add <package...> [--optional/-O]`

Using `--optional` or `-O` will install one or more packages in your [`optionalDependencies`](https://classic.yarnpkg.com/en/docs/dependency-types#toc-optional-dependencies).

`yarn add <package...> [--exact/-E]`

Using `--exact` or `-E` installs the packages as exact versions. The default is to use the most recent release with the same major version. For example, `yarn add foo@1.2.3` would accept version `1.9.1`, but `yarn add foo@1.2.3 --exact` would only accept version `1.2.3`.

`yarn add <package...> [--tilde/-T]`

Using `--tilde` or `-T` installs the most recent release of the packages that have the same minor version. The default is to use the most recent release with the same major version. For example, `yarn add foo@1.2.3 --tilde` would accept `1.2.9` but not `1.3.0`.

`yarn add <package...> [--ignore-workspace-root-check/-W]`

Using `--ignore-workspace-root-check` or `-W` allows a package to be installed at the workspaces root. This tends not to be desired behaviour, as dependencies are generally expected to be part of a workspace. For example `yarn add lerna --ignore-workspace-root-check --dev` at the workspaces root would allow lerna to be used within the scripts of the root package.json.

`yarn add <alias-package>@npm:<package>`

This will install a package under a custom alias. Aliasing, allows multiple versions of the same dependency to be installed, each referenced via the *alias-package* name given. For example, `yarn add my-foo@npm:foo` will install the package `foo` (at the latest version) in your [`dependencies`](https://classic.yarnpkg.com/en/docs/dependency-types#toc-dependencies) under the specified alias `my-foo`. Also, `yarn add my-foo@npm:foo@1.0.1` allows a specific version of `foo` to be installed.

`yarn add <package...> --audit`



#### [yarn audit](https://classic.yarnpkg.com/en/docs/cli/audit)

#### [yarn autoclean](https://classic.yarnpkg.com/en/docs/cli/autoclean)

#### [yarn bin](https://classic.yarnpkg.com/en/docs/cli/bin)

- yarn cache

- [yarn check](https://classic.yarnpkg.com/en/docs/cli/check)

- [yarn config](https://classic.yarnpkg.com/en/docs/cli/config)

#### [yarn create](https://classic.yarnpkg.com/en/docs/cli/create)

- [yarn dedupe](https://classic.yarnpkg.com/en/docs/cli/dedupe)

- [yarn generate-lock-entry](https://classic.yarnpkg.com/en/docs/cli/generate-lock-entry)

- [yarn global](https://classic.yarnpkg.com/en/docs/cli/global)

- [yarn help](https://classic.yarnpkg.com/en/docs/cli/help)

- [yarn import](https://classic.yarnpkg.com/en/docs/cli/import)

- [yarn info](https://classic.yarnpkg.com/en/docs/cli/info)

- [yarn init](https://classic.yarnpkg.com/en/docs/cli/init)

- [yarn install](https://classic.yarnpkg.com/en/docs/cli/install)

- [yarn licenses](https://classic.yarnpkg.com/en/docs/cli/licenses)

- [yarn link](https://classic.yarnpkg.com/en/docs/cli/link)

- [yarn list](https://classic.yarnpkg.com/en/docs/cli/list)

- [yarn lockfile](https://classic.yarnpkg.com/en/docs/cli/lockfile)

- [yarn login](https://classic.yarnpkg.com/en/docs/cli/login)

- [yarn logout](https://classic.yarnpkg.com/en/docs/cli/logout)

- [yarn outdated](https://classic.yarnpkg.com/en/docs/cli/outdated)

- [yarn owner](https://classic.yarnpkg.com/en/docs/cli/owner)

- [yarn pack](https://classic.yarnpkg.com/en/docs/cli/pack)

- [yarn policies](https://classic.yarnpkg.com/en/docs/cli/policies)

- [yarn prune](https://classic.yarnpkg.com/en/docs/cli/prune)

- [yarn publish](https://classic.yarnpkg.com/en/docs/cli/publish)

- [yarn remove](https://classic.yarnpkg.com/en/docs/cli/remove)

- [yarn run](https://classic.yarnpkg.com/en/docs/cli/run)

- [yarn self-update](https://classic.yarnpkg.com/en/docs/cli/self-update)

- [yarn tag](https://classic.yarnpkg.com/en/docs/cli/tag)

- [yarn team](https://classic.yarnpkg.com/en/docs/cli/team)

- [yarn test](https://classic.yarnpkg.com/en/docs/cli/test)

- [yarn unlink](https://classic.yarnpkg.com/en/docs/cli/unlink)

- [yarn upgrade](https://classic.yarnpkg.com/en/docs/cli/upgrade)

- [yarn upgrade-interactive](https://classic.yarnpkg.com/en/docs/cli/upgrade-interactive)

- [yarn version](https://classic.yarnpkg.com/en/docs/cli/version)

- [yarn versions](https://classic.yarnpkg.com/en/docs/cli/versions)

- [yarn why](https://classic.yarnpkg.com/en/docs/cli/why)

- [yarn workspace](https://classic.yarnpkg.com/en/docs/cli/workspace)

- [yarn workspaces](https://classic.yarnpkg.com/en/docs/cli/workspaces)

- 

## NPM

### install

```
sudo apt install npm
npm config set prefix /TOOLS/npm
npm config set cache /TOOLS/npm/node_cache
npm config set registry https://registry.npm.taobao.org --global
npm config ls
```

npm 升级

```
npm install -g npm@8.1.4
```

注：

如果使用nvm安装的npm，则config配置应该修改`${VNMHOME}/versions/node/v16.13.1/lib/node_modules/npm/npmrc`文件

### 基本命令

```
npm config ls -d
npm install xxxx -g -d
npm init ..
```



### 目录结构

#### .bin

通过`npm`启动的脚本，会默认把`node_modules/.bin`加到`PATH`环境变量中

当某个模块配置了 `bin` 定义时，就会被安装的时候，自动软链了过去

#### packge.json

`package.json` 文件是项目的清单。 它可以做很多完全互不相关的事情。 例如，它是用于工具的配置中心。 它也是 `npm` 和 `yarn` 存储所有已安装软件包的名称和版本的地方。



#### 脚本

npm 允许在`package.json`文件里面，使用`scripts`字段定义脚本命令。

```json
{
  "scripts": {
    "build": "node build.js"
  }
}
```

```shell
$ npm run build
# 等同于执行
$ node build.js
```

### node_modules

#### public node_modules

#### scope(@)

所有npm模块都有name，有的模块的name还有scope。scope的命名规则和name差不多，同样不能有url非法字符或者下划线点符号开头。scope在模块name中使用时，以@开头，后边跟一个/ 。package.json中，name的写法如下：

> @somescope/somepackagename

scope是一种把相关的模块组织到一起的一种方式，也会在某些地方影响npm对模块的处理。

npm公共仓库支持带有scope的的模块，同时npm客户端对没有scope的模块也是向后兼容的，所以可以同时使用两者。

### 基本命令

```
npm install <Module Name>
```

```sh
npm install express          # 本地安装
npm install express -g   # 全局安装
npm config set proxy null
npm list -g
npm uninstall express
npm update express
npm init
```

## NPX

npx 主要用于命令行的寻址等辅助功能上，而 npm 是管理依赖的。

```
npx create-react-app my-app
```



## Yarn

### install

```

```



### weith npm

```text
npm install === yarn 
npm install taco --save === yarn add taco
npm uninstall taco --save === yarn remove taco
npm install taco --save-dev === yarn add taco --dev
npm update --save === yarn upgrade
```

## 开发

### vscode

#### create launch.json



#### 快捷键

| 快捷键       | 说明                |      |
| ------------ | ------------------- | ---- |
| ctrl+F5      | 运行当前文件        |      |
| F5           | debug当前文件       |      |
| Ctrl+Shift+P | **Command Palette** |      |

#### 基本命令

| npm install |      |
| ----------- | ---- |
|             |      |
|             |      |
|             |      |



## 组件

### storybook

