---
title: docker java api使用
date: 2018-05-30 09:20:28
tags: [docker api]
---

## 使用场景

一个内部系统的业务需要使用到各种运行环境（例如jdk6,jdk7,jdk8都需要），使用宿主机上的环境，一方面维护复杂，另一方面使用也比较麻烦（需要针对不同业务流设置不同环境变量），因此考虑使用调用宿主机上的docker来完成。

## 使用方法

### 第一步 配置宿主机的docker服务

#编辑配置：/etc/systemd/system/docker.service.d/docker.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
#刷新docker配置并重启
systemctl daemon-reload
systemctl restart docker

### 第二步 使用java api远程调用 

```java
 public static void main(String[] args) {
        DockerClientConfig config = DefaultDockerClientConfig.createDefaultConfigBuilder()
                .withDockerHost("tcp://172.26.X.XX:2375")//172.26.X.XX为宿主机ip
                .withDockerTlsVerify(false)
                .withDockerConfig("/home/user/.docker")
                .withApiVersion("1.23")
                .withRegistryUrl("https://registrindex.docker.io/v1/")
                .withRegistryUsername("dockeruser")
                .withRegistryPassword("ilovedocker")
                .withRegistryEmail("dockeruser@github.com")
                .build();

        DockerClient dockerClient = DockerClientBuilder.getInstance(config).build();
        Info info = dockerClient.infoCmd().exec();
        CreateContainerResponse container = dockerClient.createContainerCmd("maven:jdk7")
                .withBinds(new Bind("/home/user/Demo", new Volume("/Demo")))//挂载宿主机上的文件夹
                .withCmd("mvn","compile","-f","/Demo/pom.xml")
                .exec();
        dockerClient.startContainerCmd(container.getId()).exec();
        dockerClient.waitContainerCmd(container.getId()).exec(new WaitContainerResultCallback()).awaitStatusCode();
        dockerClient.removeContainerCmd(container.getId()).exec();
    }
```

