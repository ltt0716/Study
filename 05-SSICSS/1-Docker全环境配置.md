# 一键启动全栈开发环境 

1. **安装Docker Desktop**
   - 访问官网下载[Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)。
   - 运行安装程序,启用 WSL 2 (Windows Subsystem for Linux 2)。
   - 安装完毕后重启电脑。
   - 运行Docker Desktop  
    如遇到 **WSL 需要更新**(WSL needs updating)，进行如下操作：
     1. 以管理员身份打开 PowerShell;
     2. 运行更新命令:  
        ```powershell
        wsl --update
        ```
        
2. 创建后端 `Dockerfile` 文件   
   - 进入 Backend/ 目录，创建一个名为 Dockerfile 的文件（无后缀），并填入以下内容。这个文件专门为开发模式设计。
   - 文件路径: /my-project/backend/Dockerfile
    ```Dockerfile
    # 使用一个已经内置了 Maven 和 Java 8 (OpenJDK 8) 的官方镜像
    FROM maven:3.8-openjdk-8

    # 在容器内创建一个工作目录
    WORKDIR /app

    # 复制 pom.xml 文件。这一步是为了利用Docker的缓存机制。
    # 只有当 pom.xml 变化时（例如增加了新依赖），才会重新执行下载依赖的步骤。
    COPY pom.xml .

    # 下载所有依赖项并存起来，以便后续快速构建
    RUN mvn dependency:go-offline

    # 将你的项目源代码全部复制到容器的工作目录中
    COPY src ./src

    # 声明 Spring Boot 应用的端口，以及远程调试端口(可选，但非常有用)
    EXPOSE 8082
    EXPOSE 5005

    # 容器启动时要执行的命令
    # 1. 使用 mvn spring-boot:run 来启动应用，它会监控文件变化并自动重启
    #    (前提是你的 pom.xml 中有 spring-boot-devtools 依赖)
    # 2. 开启远程调试功能，让你可以从IDE连接到容器进行Debug
    CMD ["mvn", "spring-boot:run", "-Dspring-boot.run.jvmArguments=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005"]```

3. 前端创建 `Dockerfile` 文件
   - 进入 `Frontend/` 目录，创建 `Dockerfile` 文件（无文件后缀）。
   - **文件路径: `/my-project/frontend/Dockerfile`**
    ```dockerfile
    # 使用一个轻量级的 Node.js 镜像
    FROM node:18-alpine

    # 在容器内设置工作目录
    WORKDIR /app

    # 同样利用缓存，先只复制 package.json 相关文件
    COPY package*.json ./

    # 安装所有前端依赖
    RUN npm install

    # 复制所有前端源代码到容器中
    COPY . .

    # 暴露 Vue 开发服务器的端口
    EXPOSE 8080

    # 启动开发服务器
    # --host 0.0.0.0 参数是关键，它让容器外的请求可以访问到这个服务 
    CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
    ```

4. 创建 `docker-compose.yml` (总指挥文件)  
   - 这是整个系统的核心。在你的项目总目录 (/my-project/) 下创建 `docker-compose.yml` 文件。
   - 文件路径: `/my-project/docker-compose.yml`
 
    ```yml
 
   version: '3.8'   
   services:
     # 后端服务 (Java 8)
     Backend:
       build:
         context: ./backend
         dockerfile: Dockerfile
       container_name: my-backend-app
       ports:
         - "8080:8080" # 映射应用端口: 主机:容器
         - "5005:5005" # 映射远程调试端口
       volumes:
         - ./backend/src:/app/src # 将本地的 src 目录实时同步到容器内
       environment:
         - SPRING_DATASOURCE_URL=jdbc:mysql://mysql-db:3306/mydatabase?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
         - SPRING_DATASOURCE_USERNAME=root
         - SPRING_DATASOURCE_PASSWORD=rootpassword
         - SPRING_REDIS_HOST=redis-cache
         - SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka-broker:9092
       networks:
         - dev-network
       depends_on: # 确保这些服务先于后端启动
         - mysql-db
         - redis-cache
         - kafka-broker

     # 前端服务
     frontend:
       build:
         context: ./frontend
         dockerfile: Dockerfile
       container_name: my-frontend-app
       ports:
         - "8081:8080" # 将容器的8080映射到主机的8081，避免和后端冲突
       volumes:
         - ./frontend:/app # 实时同步整个前端项目
         - /app/node_modules # 防止本地node_modules覆盖容器内的
       networks:
         - dev-network

     # MySQL 数据库服务
     mysql-db:
       image: mysql:8.0
       container_name: mysql-db
       ports:
         - "3306:3306"
       environment:
         - MYSQL_ROOT_PASSWORD=rootpassword
         - MYSQL_DATABASE=mydatabase
       volumes:
         - mysql-data:/var/lib/mysql # 将数据持久化，防止容器重启后数据丢失
       networks:
         - dev-network

     # Redis 服务
     redis-cache:
       image: redis:6.2-alpine
       container_name: redis-cache
       ports:
         - "6379:6379"
       volumes:
         - redis-data:/data # 持久化Redis数据
       networks:
         - dev-network

     # Zookeeper 服务 (Kafka 依赖)
     zookeeper:
       image: confluentinc/cp-zookeeper:7.0.1
       container_name: zookeeper
       ports:
         - "2181:2181"
       environment:
         ZOOKEEPER_CLIENT_PORT: 2181
         ZOOKEEPER_TICK_TIME: 2000
       networks:
         - dev-network

     # Kafka 服务
     kafka-broker:
       image: confluentinc/cp-kafka:7.0.1
       container_name: kafka-broker
       ports:
         - "9092:9092"
       depends_on:
         - zookeeper
       environment:
         KAFKA_BROKER_ID: 1
         KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
         KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker:9092
         KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
       networks:
         - dev-network

   # 定义一个共享网络，让所有容器可以互相通信
   networks:
     dev-network:
       driver: bridge

   # 声明数据卷，用于持久化存储
   volumes:
     mysql-data:
     redis-data:
    ```
5. 一键启动！  
   - 打开一个终端 (PowerShell 或 Windows Terminal)，cd 进入到你的项目总目录 (/my-project/)。  
   - 运行命令：
    ```bash
    docker-compose up --build
    ```
    >- 第一次运行会需要一些时间，因为它需要从网上下载 MySQL, Redis, Java, Node 等所有基础环境的镜像。
    >- 后续运行会非常快，因为镜像已经存在本地了。如果你没有修改 Dockerfile，甚至可以直接运行 docker-compose up。
    >- 当你在终端看到所有服务的日志开始稳定输出时，代表你的全栈开发环境已经成功运行！

6. 访问和使用开发环境
   - 访问前端页面: 打开浏览器，访问 http://localhost:8081
   - 访问后端API: 你的API的根路径是 http://localhost:8080
   - 连接数据库:
     * 工具：Navicat
     * 主机: localhost
     * 端口: 3306
     * 用户: root
     * 密码: rootpassword
   - 连接Redis:
     * 工具：Redis Desktop Manager
     * 主机: localhost
     * 端口: 6379

7. 日常开发流程
   - 用 VS Code 或 IntelliJ IDEA 同时打开 backend 和 frontend 文件夹。
   - 修改后端 .java 代码后保存，容器内的 Spring Boot 会自动重启。
   - 修改前端 .vue 代码后保存，浏览器里的页面会自动热更新。
   - 一天工作结束后，在终端里按 Ctrl + C，然后运行 docker-compose down 来关闭所有服务。你的数据库数据会因为 volumes 的设置而被保留下来。