# `docker-compose.yml`

`docker-compose.yml` 是掌握 Docker 编排的关键。  
把这个文件分为三个部分：**顶层定义** (`version`, `services`, `networks`, `volumes`) 和 **服务内定义**（每个服务下的具体配置）。

---
## 一、顶层定义 (全局配置)

这部分定义了整个文件的基本信息和共享资源。

*   `version: '3.8'`
    *   **含义**: 指定了此文件遵循的 `docker-compose` 文件格式版本。

*   `services:`
    *   **含义**: 这是文件的核心部分，所有你想运行的独立应用/组件（我们称之为“服务”）都在这个关键字下面定义。你文件中的 `backend`, `frontend`, `mysql-db` 等都是服务。

*   `networks:`
    *   **含义**: 在这里统一定义服务可以连接的虚拟网络。
    *   `dev-network:`: 你为这个网络起的名字。
    *   `driver: bridge`: `bridge` 是最常见的网络驱动模式。它会创建一个私有的内部网络，所有加入这个网络的容器都可以通过它们的服务名相互通信（例如，后端服务可以通过主机名 `mysql-db` 访问到数据库服务）。

*   `volumes:`
    *   **含义**: 在这里声明“命名的卷(Named Volumes)”。这些是由 Docker 管理的、用于持久化存储数据的特殊文件夹。
    *   `mysql-data:` 和 `redis-data:`: 你声明了两个命名的卷。即使你删除了 MySQL 或 Redis 的容器，这些卷里的数据（你的数据库文件等）也会被保留下来，下次启动容器时会重新挂载，从而保证数据不丢失。

---
## 二、服务内定义 (针对每个服务的详细配置)

下面我们来逐一解释每个服务（如 `backend`, `mysql-db`）内部的配置字段是什么意思。

#### `build`
*   **含义**: 指示 Docker Compose 如何**构建**一个自定义的 Docker 镜像。这通常用于你自己的应用程序（后端、前端）。
*   **`context: ./backend`**: 指定 Dockerfile 所在的**上下文目录**。Docker 会把这个目录下的所有文件发送到 Docker 守护进程以开始构建。`./backend` 指的是与 `docker-compose.yml` 文件同级的 `backend` 文件夹。
*   **`dockerfile: Dockerfile`**: 指定在上下文目录中用于构建镜像的 Dockerfile 文件名。

#### `image`
*   **含义**: 指示 Docker 使用一个**已经存在**的、预先构建好的镜像。通常从 Docker Hub（一个公共的镜像仓库）上拉取。这用于标准软件，如数据库、缓存等。
*   **例子**: `image: mysql:8.0` 告诉 Docker 去拉取官方的 MySQL 8.0 版本的镜像。

> **`build` vs `image`**: 一个服务要么使用 `build` 来创建自己的镜像，要么使用 `image` 来拉取现成的镜像，两者通常不同时使用。

#### `container_name`
*   **含义**: 为启动后的容器指定一个**固定的、易于记忆的名字**。
*   **好处**: 如果不指定，Docker 会分配一个随机的名字（如 `my-project_backend_1`）。指定后，你可以很容易地通过名字来管理这个容器，例如 `docker logs my-backend-app`。

#### `ports`
*   **含义**: 设置**端口映射**，在你的电脑（主机）和容器之间建立一个“管道”。
*   **格式**: `"<主机端口>:<容器端口>"`
*   **例子**:
    *   `"8080:8080"`: 将你电脑的 8080 端口收到的请求，转发到容器内部的 8080 端口。
    *   `"8081:8080"`: 将你电脑的 8081 端口的请求，转发到容器内部的 8080 端口（这样做是为了避免前端和后端同时占用主机的 8080 端口）。

#### `volumes`
*   **含义**: 将主机的文件/文件夹或命名的卷**挂载**到容器内部。这是实现**数据持久化**和**代码热重载**的关键。
*   **两种形式**:
    1.  **代码同步 (Bind Mount)**: `主机路径:容器内路径`
        *   **例子**: `- ./backend/src:/app/src`
        *   **作用**: 将你电脑上的 `backend/src` 文件夹直接映射到容器内的 `/app/src` 目录。你在主机上修改代码，容器内的文件会**立即同步更新**，`spring-boot-devtools` 检测到变化后就会自动重启应用。
    2.  **数据持久化 (Named Volume)**: `命名卷:容器内路径`
        *   **例子**: `- mysql-data:/var/lib/mysql`
        *   **作用**: 将我们之前在顶层定义的 `mysql-data` 卷挂载到 MySQL 容器存放数据的标准目录 `/var/lib/mysql`。这样，所有的数据库文件都会保存在这个由 Docker 托管的卷里，而不是容器里，从而实现了数据的持久化。
*   **特殊用法**: `- /app/node_modules` (在 `frontend` 服务中)
    *   这是一个巧妙的技巧。它创建了一个匿名的卷并挂载到容器的 `/app/node_modules`。这可以防止主机上的 `node_modules` 文件夹（可能是空的）覆盖掉在 `Dockerfile` 中通过 `npm install` 安装好的 `node_modules` 目录。

#### `environment`
*   **含义**: 在容器内部设置**环境变量**。
*   **作用**: 这是一种将配置信息（如数据库密码、主机名等）注入到应用中的最佳实践，避免了将敏感信息或配置硬编码在代码里。
*   **例子**: `SPRING_DATASOURCE_USERNAME=root` 会在后端容器内创建一个名为 `SPRING_DATASOURCE_USERNAME` 的环境变量，值为 `root`。Spring Boot 应用会自动读取这个变量来配置数据库连接。

#### `networks`
*   **含义**: 将此服务连接到指定的网络。
*   **例子**: `networks: - dev-network` 表示将该服务加入到我们定义的 `dev-network` 网络中，使其能够与其他同样在该网络中的服务通信。

#### `depends_on`
*   **含义**: 定义服务的**启动依赖关系**。
*   **作用**: 确保一个服务在它所依赖的服务**启动之后**才启动。
*   **例子**: `backend` 服务中的 `depends_on: - mysql-db ...` 意味着 `docker-compose` 会先启动 `mysql-db`, `redis-cache` 和 `kafka-broker` 容器，然后再启动 `backend` 容器。
*   **注意**: `depends_on` 只保证启动顺序，并不保证被依赖的应用已经完全准备好接受请求（例如，MySQL 数据库可能还在初始化）。但在大多数开发场景下，这已经足够了。