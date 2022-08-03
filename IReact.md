React
===

## Install

1. install npm
2. install nvm
3. install npx
4. install nodejs
5. install vscode

## create-react-app

### npm init

使用如下命令创建项目

```sh
npm init react-app my-react-app --verbose
```

或者使用

```
npx create-react-app my-app
```

注：react-app 是facebook开源的穿件react项目的项目

[参考地址](https://github.com/facebook/create-react-app)

### run

```sh
npm start
```

使用该命令执行的时候，剧本热加载的能力，不需要再另外配置webpack



## sample

### Welcome to React[#](https://code.visualstudio.com/docs/nodejs/reactjs-tutorial#_welcome-to-react)

We'll be using the `create-react-app` [generator](https://reactjs.org/docs/create-a-new-react-app.html#create-react-app) for this tutorial. To use the generator as well as run the React application server, you'll need [Node.js](https://nodejs.org/) JavaScript runtime and [npm](https://www.npmjs.com/) (Node.js package manager) installed. npm is included with Node.js which you can download and install from [Node.js downloads](https://nodejs.org/en/download/).

> **Tip**: To test that you have Node.js and npm correctly installed on your machine, you can type `node --version` and `npm --version` in a terminal or command prompt.

You can now create a new React application by typing:

```
npx create-react-app my-app
```

where `my-app` is the name of the folder for your application. This may take a few minutes to create the React application and install its dependencies.

> **Note**: If you've previously installed `create-react-app` globally via `npm install -g create-react-app`, we recommend you uninstall the package using `npm uninstall -g create-react-app` to ensure that npx always uses the latest version.

Let's quickly run our React application by navigating to the new folder and typing `npm start` to start the web server and open the application in a browser:

```
cd my-app
npm start
```

You should see the React logo and a link to "Learn React" on [http://localhost:3000](http://localhost:3000/) in your browser. We'll leave the web server running while we look at the application with VS Code.

To open your React application in VS Code, open another terminal or command prompt window, navigate to the `my-app` folder and type `code .`:

```
cd my-app
code .
```

### Markdown preview[#](https://code.visualstudio.com/docs/nodejs/reactjs-tutorial#_markdown-preview)

In the File Explorer, one file you'll see is the application `README.md` Markdown file. This has lots of great information about the application and React in general. A nice way to review the README is by using the VS Code [Markdown Preview](https://code.visualstudio.com/docs/languages/markdown#_markdown-preview). You can open the preview in either the current editor group (**Markdown: Open Preview** Ctrl+Shift+V) or in a new editor group to the side (**Markdown: Open Preview to the Side** Ctrl+K V). You'll get nice formatting, hyperlink navigation to headers, and syntax highlighting in code blocks.

## vscode

## 重要组件

#### antd

#### flux

#### redux

#### dispatcher



## webpack

[官方资料](https://webpack.js.org/guides/getting-started/)



## React

### 基本概念

#### useState

#### useEffect

#### Promise

#### route

#### props

### 用法

#### 数据绑定

#### 异步请求

#### 事件

## 组件

### redux

#### useDispath

#### createStore

### 父子组件传值



## TypeScript

#### 声明文件

#### interface

```typescript
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}
```

#### 函数

基本定义方法

```typescript
let mySum = function (x: number, y: number): number {
    return x + y;
};
function sum(x, y) {
    return x + y;
}
let mySum = function (x, y) {
    return x + y;
};
let mySum: (x: number, y: number) => number = function (x: number, y: number): number {
    return x + y;
};
```

接口定义方法

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
    return source.search(subString) !== -1;
}
```

可选参数

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName) {
        return firstName + ' ' + lastName;
    } else {
        return firstName;
    }
}
let tomcat = buildName('Tom', 'Cat');
let tom = buildName('Tom');
```

默认参数

```ts
function buildName(firstName: string, lastName: string = 'Cat') {
    return firstName + ' ' + lastName;
}
```

剩余参数

```ts
function push(array, ...items) {
    items.forEach(function(item) {
        array.push(item);
    });
}

let a: any[] = [];
push(a, 1, 2, 3);
```



#### type



