### SonarQube

- 安裝

        docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest

- 登入

    <http://localhost:9000/>

    login: admin
    password: admin

    ![Alt text](/assets/image-69.png)

- 需要修改密碼

    password: mitacadmin

    ![Alt text](/assets/image-70.png)

- 兩個參數

    sonar_host: <http://127.0.0.1:9000/>
    sonar_token: sqa_2d932888615a4ca75bf80e889a422a9bde711ec3

    User > My Account > Security
    ![Alt text](/assets/image-71.png)

- 與 Drone 整合

  - 建立兩個 secret

        sonar_host: <http://127.0.0.1:9000/>
        sonar_token: sqa_2d932888615a4ca75bf80e889a422a9bde711ec3

  - .drone.yml 範例 (要放在 Package 後)

  - name: code-analysis
        /assets/image: aosapps/drone-sonar-plugin
        settings:
        sonar_host:
            from_secret: sonar_host
        sonar_token:
            from_secret: sonar_token
        commands:
    - sonar-scanner -Dsonar.projectKey=docker-demo
            -Dsonar.tests=src/test/java
            -Dsonar.sources=src/main/java
            -Dsonar.java.libraries=./target/classes
            -Dsonar.java.binaries=./target/classes
            -Dsonar.host.url=<http://172.31.93.122:9000/>
            -Dsonar.login=sqa_2d932888615a4ca75bf80e889a422a9bde711ec3
