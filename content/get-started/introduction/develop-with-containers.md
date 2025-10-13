---
title: 使用容器进行开发
keywords: concepts, build, images, container, docker desktop
description: 本页将教你如何使用容器进行开发。
summary: |
  学习如何运行你的第一个容器，亲身体验 Docker 的强大能力。
  本文将展示如何在容器化环境中对后端与前端代码进行实时修改，
  实现顺畅的一体化开发与测试体验。
weight: 2
aliases:
 - /guides/getting-started/develop-with-containers/
---

{{< youtube-embed D0SDBrS3t9I >}}

## 说明

现在你已经安装了 Docker Desktop，可以开始进行应用开发了。你将完成以下任务：

1. 克隆并启动一个开发项目
2. 修改后端与前端代码
3. 立即看到变更效果

## 试一试

在这个动手指南中，你将学习如何使用容器进行开发。


## 启动项目

1. 首先，将项目克隆到本地，或[下载 ZIP 包](https://github.com/docker/getting-started-todo-app/archive/refs/heads/main.zip)。

    ```console
    $ git clone https://github.com/docker/getting-started-todo-app
    ```

    克隆完成后，进入项目目录：

    ```console
    $ cd getting-started-todo-app
    ```

2. 获取项目后，使用 Docker Compose 启动开发环境。


    使用 CLI 启动项目，运行以下命令：

   ```console
   $ docker compose watch
   ```

   你将看到镜像拉取、容器启动等输出。此时不必完全理解所有内容，稍等片刻，环境就会稳定并启动完成。


3. 打开浏览器访问 [http://localhost](http://localhost) 即可查看正在运行的应用。应用启动可能需要几分钟。该应用是一个简单的待办应用，你可以添加条目、标记完成或删除条目。

    ![Screenshot of the getting started to-do app after its first launch](images/develop-getting-started-app-first-launch.webp)


### 环境中包含什么？

环境已经运行起来了，它实际包含哪些内容？从高层看，它由多个容器（或进程）组成，为应用提供不同的能力：

- React 前端——一个运行 React 开发服务器的 Node 容器，使用 [Vite](https://vitejs.dev/)。
- Node 后端——提供获取、创建与删除待办项的 API。
- MySQL 数据库——用于存储待办项列表。
- phpMyAdmin——用于与数据库交互的 Web 界面，可通过 [http://db.localhost](http://db.localhost) 访问。
- Traefik 代理——[Traefik](https://traefik.io/traefik/) 是一个应用代理，将请求路由到正确的服务：把对 `localhost/api/*` 的请求发往后端，把对 `localhost/*` 的请求发往前端，把对 `db.localhost` 的请求发往 phpMyAdmin。这样你即可通过 80 端口访问所有应用（而无需为每个服务使用不同端口）。

在这个环境中，作为开发者的你无需安装或配置任何服务、初始化数据库结构或配置数据库凭据。你只需要 Docker Desktop，其余的一切都能正常工作。


## 修改应用

环境运行后，你可以开始修改应用，体验 Docker 如何提供快速的反馈循环。

### 修改欢迎语

页面顶部的欢迎语由 `/api/greeting` 接口返回。目前它总是返回 "Hello world!"。现在我们将其修改为在三条可自定义的消息中随机返回一条。

1. 在文本编辑器中打开 `backend/src/routes/getGreeting.js` 文件。该文件提供该 API 端点的处理函数。

2. 将文件顶部的变量改为一个欢迎语数组。可以使用以下示例或自定义。并更新端点逻辑，从该列表中随机返回一条。

    ```js {linenos=table,hl_lines=["1-5",9],linenostart=1}
    const GREETINGS = [
        "Whalecome!",
        "All hands on deck!",
        "Charting the course ahead!",
    ];

    module.exports = async (req, res) => {
        res.send({
            greeting: GREETINGS[ Math.floor( Math.random() * GREETINGS.length )],
        });
    };
    ```

3. 保存文件。刷新浏览器即可看到新的欢迎语。多次刷新可以看到列表中的不同消息。

    ![Screenshot of the to-do app with a new greeting](images/develop-app-with-greetings.webp)


### 修改占位文本

应用中的输入框占位文本目前是 "New Item"。我们将其改得更具描述性、更有趣一点。同时也会对样式做一些调整。

1. 打开 `client/src/components/AddNewItemForm.jsx` 文件。该组件用于在待办清单中新增条目。

2. 修改 `Form.Control` 元素的 `placeholder` 属性为你希望展示的文字。

    ```js {linenos=table,hl_lines=[5],linenostart=33}
    <Form.Control
        value={newItem}
        onChange={(e) => setNewItem(e.target.value)}
        type="text"
        placeholder="What do you need to do?"
        aria-label="New item"
    />
    ```

3. 保存文件并回到浏览器。你会看到改动已经通过热更新生效。如需微调，可反复修改直至满意。

![Screenshot of the to-do app with an updated placeholder in the add item text field"](images/develop-app-with-updated-placeholder.webp)


### 修改背景色

在认为应用已定稿之前，我们先优化一下配色。

1. 打开 `client/src/index.scss` 文件。

2. 将 `background-color` 修改为你喜欢的颜色。示例代码使用了与 Docker 航海主题相配的柔和蓝色。

    如果你使用 IDE，可通过内置取色器选择颜色；也可以使用在线[取色器](https://www.w3schools.com/colors/colors_picker.asp)。

    ```css {linenos=table,hl_lines=2,linenostart=3}
    body {
        background-color: #99bbff;
        margin-top: 50px;
        font-family: 'Lato';
    }
    ```

    每次保存后都能在浏览器中立即看到效果。不断调整直到你满意为止。


    ![Screenshot of the to-do app with a new placeholder and background color"](images/develop-app-with-updated-client.webp)

到这里就完成了，恭喜你更新了网站。


## 回顾

在继续之前，先回顾一下你完成了哪些事情：

- 以零安装成本启动了一个完整的开发项目。容器化环境提供了所需的一切，无需在本机安装 Node、MySQL 或其他依赖。你只需要 Docker Desktop 与代码编辑器。

- 修改代码后即可立即看到效果。这得益于：1）各容器内的进程会监听并响应文件变更；2）代码文件与容器化环境共享。

Docker Desktop 让这一切成为可能，且不止于此。一旦以容器化思维进行开发，你几乎可以创建任意环境，并轻松与团队共享。

## 下一步

应用已更新完毕，接下来你将学习如何将其打包为容器镜像并推送到仓库，具体来说是 Docker Hub。

{{< button text="构建并推送你的第一个镜像" url="build-and-push-first-image" >}}

