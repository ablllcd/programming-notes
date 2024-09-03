## React 项目启动

```
// 1. 创建vite项目
npm create vite@4.1.0

// 2. 选择框架和语言
create-vite@4.1.0
Ok to proceed? (y) y
√ Project name: ... vite-react
√ Select a framework: » React
√ Select a variant: » TypeScript

// 3. 进入当前目录下创建的项目
cd vite-react

// 4. 下载依赖的包
npm install

// 5. 启动项目
npm run dev

```

### 项目结构

1. node_modules: 存储依赖包
2. public: 存放静态资源，例如图片，字体等
3. src/assets：也是存放静态资源，此处存放的资源会被打包，而public中的不会
4. src/*.jsx：javascript xml文件，也是写组件的文件
5. src/*.css：风格文件
6. index.html：入口文件
7. package.json：项目文件，类似于pom文件

## React 组件

React通过一个个组件来构建成组件树，也就是 Virtual Dom。然后 React Dom 包来负责将组件数来转为浏览器理解的DOM文件。

![alt text](pic/Theory.png)


### 什么是组件

react中每个组件都是一个tsx文件，其中格式为：

```
function ListGroup() {
    return (<ul className="list-group">
        <li className="list-group-item">An item</li>
        <li className="list-group-item">A second item</li>
        <li className="list-group-item">A third item</li>
        <li className="list-group-item">A fourth item</li>
        <li className="list-group-item">And a fifth one</li>
    </ul>);
}

export default ListGroup;
```

需要注意，每个组件中只能返回一个HTML ELEMENT。