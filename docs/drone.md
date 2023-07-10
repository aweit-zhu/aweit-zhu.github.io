# Drone 筆記

### 1.透過 ngrok，建立公開IP，綁定 localhost:8082  (重啟、重開機就沒了)

![Alt text](/assets/image-51.png)

![Alt text](/assets/image-52.png)

### 2.綁定github

- 前往 settings -> Developer Settings
![Alt text](/assets/image-53.png)

- 建立一個 OAuth APP
Homepage URL：填剛剛用 ngrok 提共的 公開 IP  <https://b5c9-58-115-111-122.ngrok-free.app>
![Alt text](/assets/image-54.png)

- 產生 Client ID 與 Client Secret

![Alt text](/assets/image-55.png)

Client ID： 3ab12bcea99814fd59df
Client Secret： 2ae999f7e3e8bc29c1c5fec1918d1d1ba13a2067

### 3.透過 Docker Compose 建立 drone server

- 開啟 CMD，輸入 WSL。

    ![Alt text](/assets/image-49.png)

- 切到 home 路徑。

        cd ~

- 建立 drone-server 資料夾

        mkdir drone-server
        cd drone-server

- 建立 docker-compose.github.yml

        nano docker-compose.github.yml

  貼上以下

    version: '2'
    services:
    drone-server:
        /assets/image: drone/drone:1
        ports:
        - 8082:80
        volumes:
        - ./:/data
        restart: always
        environment:
        - DRONE_SERVER_HOST=${DRONE_SERVER_HOST}
        - DRONE_SERVER_PROTO=${DRONE_SERVER_PROTO}
        - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
        - DRONE_GITHUB_SERVER=<https://github.com>
        - DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}
        - DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}
        - DRONE_LOGS_PRETTY=true
        - DRONE_LOGS_COLOR=true
        - DRONE_USER_CREATE=username:aweit-zhu,admin:true

# runner for docker version

    drone-runner:
        /assets/image: drone/drone-runner-docker:1
        restart: always
        depends_on:
        - drone-server
        volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        environment:
        - DRONE_RPC_HOST=${DRONE_RPC_HOST}
        - DRONE_RPC_PROTO=${DRONE_RPC_PROTO}
        - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}
        - DRONE_RUNNER_CAPACITY=3

- 建立 .env

        nano .env

貼上以下

    我們目前有的資訊為：
    公開 IP  <https://b5c9-58-115-111-122.ngrok-free.app>
    Client ID： 3ab12bcea99814fd59df
    Client Secret： 2ae999f7e3e8bc29c1c5fec1918d1d1ba13a2067

    DRONE_SERVER_HOST=<https://b5c9-58-115-111-122.ngrok-free.app>
    DRONE_SERVER_PROTO=https
    DRONE_RPC_SECRET=123
    DRONE_RPC_HOST=drone-server
    DRONE_RPC_PROTO=http
    DRONE_GITHUB_CLIENT_ID=3ab12bcea99814fd59df
    DRONE_GITHUB_CLIENT_SECRET=2ae999f7e3e8bc29c1c5fec1918d1d1ba13a2067

![Alt text](/assets/image-56.png)

- 啟動容器

    docker-compose -f docker-compose.github.yml up -d

![Alt text](/assets/image-57.png)

![Alt text](/assets/image-58.png)

- 打開 <https://b5c9-58-115-111-122.ngrok-free.app>

點選 Visit Site

![Alt text](/assets/image-59.png)

點選 Authorize aweit-zhu

![Alt text](/assets/image-60.png)

- Drone Server 的操作

選擇 aweit-zhu/docker-demo 進行 Activate
選擇 Trusted
點選 Save
![Alt text](/assets/image-61.png)

- 回到 github 做設定

將 Payload URL 修改為
修改為 <https://b5c9-58-115-111-122.ngrok-free.app> /hook

![Alt text](/assets/image-62.png)

### 4. 撰寫 .drone.yml 檔

- 建立 ssh_password_master secret

![Alt text](/assets/image-63.png)

- 建立 ssh_password secret

![Alt text](/assets/image-64.png)

- 解析 .drone.yml (略)

        kind: pipeline
        type: docker
        name: docker-demo-drone
        steps:

        - name: test
            /assets/image: maven:3-jdk-8
            volumes:
        - name: maven-cache
                path: /root/.m2
        - name: maven-build
                path: /app/build
            commands:
        - mvn install -DskipTests=true -Dmaven.javadoc.skip=true -B -V
        - mvn test -B

        - name: package
            /assets/image: maven:3-jdk-8
            volumes:
        - name: maven-cache
                path: /root/.m2
        - name: maven-build
                path: /app/build
            commands:
        - mvn clean package
        - cp target/docker-demo.jar /app/build/docker-demo.jar
        - cp Dockerfile /app/build/Dockerfile
        - cp docker-k8s-demo-deployment.yaml /app/build/docker-k8s-demo-deployment.yaml
        - cp deploymentservice.yaml /app/build/deploymentservice.yaml
        - cp run.sh /app/build/run.sh

        - name: scp files
            /assets/image: appleboy/drone-scp
            settings:
            host: 192.168.0.17
            username: root
            password:
                from_secret: ssh_password_master
            port: 22
            command_timeout: 2m
            target: /mydata/maven/build
            source: ./*

        - name: build-start01
            /assets/image: appleboy/drone-ssh
            settings:
            host: 172.31.93.122
            username: root
            password:
                from_secret: ssh_password
            port: 2222
            command_timeout: 5m
            script:
                - cd /mydata/maven/build
                - chmod +x run.sh
                - ./run.sh

        - name: build-start02
            /assets/image: appleboy/drone-ssh
            settings:
            host: 192.168.0.17
            username: vboxuser
            password:
                from_secret: ssh_password_master
            port: 22
            command_timeout: 5m
            script:
                - cd /mydata/maven/build
                - microk8s.kubectl delete deploy docker-k8s-demo-deployment
                - microk8s.kubectl apply -f ./docker-k8s-demo-deployment.yaml

        volumes:

        - name: maven-build
            host:
            path: /mydata/maven/build
        - name: maven-cache
            host:
            path: /mydata/maven/cache

- 只要 push 到遠端，就會觸發 drone 執行。

![Alt text](/assets/image-65.png)

![Alt text](/assets/image-66.png)

- 驗證

![Alt text](/assets/image-67.png)

打開瀏覽器，輸入URL：<http://192.168.0.17:60000/>

![Alt text](/assets/image-68.png)
