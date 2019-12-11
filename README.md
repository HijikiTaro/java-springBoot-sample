# java-springBoot-sample

- [java-springBoot-sample](#java-springboot-sample)
- [環境](#%e7%92%b0%e5%a2%83)
- [実行手順](#%e5%ae%9f%e8%a1%8c%e6%89%8b%e9%a0%86)
  - [ソースのclone](#%e3%82%bd%e3%83%bc%e3%82%b9%e3%81%aeclone)
  - [buildに必要な依存関係用コンテナの作成](#build%e3%81%ab%e5%bf%85%e8%a6%81%e3%81%aa%e4%be%9d%e5%ad%98%e9%96%a2%e4%bf%82%e7%94%a8%e3%82%b3%e3%83%b3%e3%83%86%e3%83%8a%e3%81%ae%e4%bd%9c%e6%88%90)
  - [javaをbuildしたコンテナの作成](#java%e3%82%92build%e3%81%97%e3%81%9f%e3%82%b3%e3%83%b3%e3%83%86%e3%83%8a%e3%81%ae%e4%bd%9c%e6%88%90)
  - [jarファイルのみを配置したコンテナの作成](#jar%e3%83%95%e3%82%a1%e3%82%a4%e3%83%ab%e3%81%ae%e3%81%bf%e3%82%92%e9%85%8d%e7%bd%ae%e3%81%97%e3%81%9f%e3%82%b3%e3%83%b3%e3%83%86%e3%83%8a%e3%81%ae%e4%bd%9c%e6%88%90)
  - [アプリの実行](#%e3%82%a2%e3%83%97%e3%83%aa%e3%81%ae%e5%ae%9f%e8%a1%8c)

# 環境
- Windows 10 Home (64bit)
- Visuali Sutdio Code
- Docker Toolbox
  - ポート：8090
- java 8
- Spring Boot v2.2.0.RELEASE

※「Windows => VirtualBox(8090) => Docker(8080)」となるようにVirtualBoxのネットワーク設定（ポートフォワーディング）を行っている

# 実行手順
下記のgithubを参考
- [参考にしたgithub](https://github.com/yoshioterada/k8s-Azure-Container-Service-AKS--on-Azure)

コンテナ作成の考え方として、k8s勉強会で教えて頂いた「コンテナは軽量に」をベースとしている。
最終的なコンテナは、jarファイルのみを配置したコンテナとなるようにいくつかのコンテナを作成している。
（実際にはもっと整理し、めんどくさくないCI/CDが可能な構成を考えることは必要）

※コンテナイメージのバージョンは必ず着ける

## ソースのclone
githubからソースをcloneする
```
git clone https://github.com/HijikiTaro/java-springBoot-sample.git
```

## buildに必要な依存関係用コンテナの作成
コンテナイメージをbuildする
```
docker build -t maven-include-localrepo:1.0 . -f 1-Dockerfile-for-Maven 
```

上記コマンドで作成されるコンテナイメージ。

「maven」は「maven-include-localrepo」のもとイメージとなるコンテナイメージ。
```
# docker image
$ docker image ls
REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
maven-include-localrepo   1.0                 0b52344f73f0        2 minutes ago       373MB
maven                     3.6.1-jdk-8-slim    96a712cf9211        3 months ago        301MB
```

## javaをbuildしたコンテナの作成
コンテナイメージをbuildする
```
docker build -t build . -f 2-Dockerfile-for-Build 
```

上記コマンドで作成されるコンテナイメージ。
```
# docker image
$ docker image ls
REPOSITORY                TAG                 IMAGE ID            CREATED              SIZE
build                     latest              8c7d696d888c        About a minute ago   392MB
maven-include-localrepo   1.0                 0b52344f73f0        7 minutes ago        373MB
maven                     3.6.1-jdk-8-slim    96a712cf9211        3 months ago         301MB
```

## jarファイルのみを配置したコンテナの作成
コンテナイメージをbuildする
```
docker build -t java/java-springboot-sample:1.0 . -f 3-Dockerfile-for-App  
```

上記コマンドで作成されるコンテナイメージ。

「`mcr.microsoft.com/java/jdk`」は「java/java-springboot-sample」のもとイメージとなるコンテナイメージ。
```
# docker image
$ docker image ls
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
java/java-springboot-sample   1.0                 ce3bc5fbf162        10 minutes ago      256MB
build                         latest              8c7d696d888c        31 minutes ago      392MB
maven-include-localrepo       1.0                 0b52344f73f0        38 minutes ago      373MB
maven                         3.6.1-jdk-8-slim    96a712cf9211        3 months ago        301MB
mcr.microsoft.com/java/jdk    8u212-zulu-alpine   ffc8446208f6        7 months ago        237MB
```

## アプリの実行
コンテナを起動する
```
$ docker run -p 8080:8080 -it java/java-springboot-sample:1.0

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.0.RELEASE)

2019-12-11 07:53:05.859  INFO 1 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication v0.0.1-SNAPSHOT on fb93275130f7 with PID 1 (/app/app.jar started by java in /app)
2019-12-11 07:53:05.871  INFO 1 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2019-12-11 07:53:08.731  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2019-12-11 07:53:08.759  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2019-12-11 07:53:08.762  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.27]
2019-12-11 07:53:08.927  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-12-11 07:53:08.928  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 2899 ms
2019-12-11 07:53:10.225  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2019-12-11 07:53:10.721  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2019-12-11 07:53:10.727  INFO 1 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 5.791 seconds (JVM running for 6.938)
```

アプリへ接続する
```
http://localhost:8090/hello
```




