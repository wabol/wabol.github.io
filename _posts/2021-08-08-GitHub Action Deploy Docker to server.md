---
layout: post
tag: docker
category: 
title: GitHub Action Deploy Docker to server
---

### 1. 写在前面的 

接触GitHub Action有一段时间了，发现这是一个神器，现在的环境是通过Action自动部署到k8s，部分环境因为客户的服务器还是用的直接部署或者docker环境，就想能否通过Action自动部署docker，然后就有了下面的实践过程和结论。

### 2. 参考内容

```xml
      - name: Deploy
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.HOST_USERNAME }}
          password: ${{ secrets.HOST_PASSWORD }}
          port: ${{ secrets.HOST_PORT }}
          script: |
            docker stop $(docker ps --filter ancestor=${{ secrets.DOCKER_REPOSITORY }} -q)
            docker rm -f $(docker ps -a --filter ancestor=${{ secrets.DOCKER_REPOSITORY }}:latest -q)
            docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }}:latest -q)
            docker login --username=${{ secrets.DOCKER_USERNAME }} --password ${{ secrets.DOCKER_PASSWORD }} registry.cn-hangzhou.aliyuncs.com
            docker pull ${{ secrets.DOCKER_REPOSITORY }}:latest
            docker run -d -p 8000:4000 ${{ secrets.DOCKER_REPOSITORY }}:latest
```

原网址，https://blog.csdn.net/alangrady/article/details/108241799 ，感谢原作者。

### 3. 实践过程

我希望每次发布的时候能保留历史版本，而上面示例只保留唯一最新版本，所以，需要进行一些调整。同时，docker从打包，推送（推送到阿里云仓空或者docker官方仓库，或者其他仓空都行）等流程基于之前k8s的，都已经畅通，所以最终修改完的内容如下

```xml
    # Deploy to server   
    - name: Deploy to server using ssh key
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.KEY }}
        port: ${{ secrets.PORT }}
        script: |
          docker stop $(docker ps --filter name=${{ secrets.DOCKER_NAME }} -q)
          docker rm -f $(docker ps -a --filter name=${{ secrets.DOCKER_NAME }} -q)
          docker rmi -f $(docker images ${{ secrets.DOCKER_REPOSITORY }} -q)
          docker login --username=${{ secrets.REGISTRY_USERNAME }} --password ${{ secrets.REGISTRY_PASSWORD }} registry.cn-shanghai.aliyuncs.com
          docker pull ${{ secrets.DOCKER_REPOSITORY }}:${{ github.sha }}
          docker run -d --name=${{ secrets.DOCKER_NAME }} -p 4000:80 ${{ secrets.DOCKER_REPOSITORY }}:${{ github.sha }}
```

注：在push images到仓库的时候，我添加了commit作为版本，所以在这里pull image的时候也同样如此，这样在仓库里就有了完整的版本管理。

### 4. 几个坑

>下面这两个坑浪费了好长时间，请务必注意
{: .prompt-danger }

4.1 private key必须有`-----BEGIN OPENSSH PRIVATE KEY-----`和`-----END OPENSSH PRIVATE KEY-----`内容，生成key的方式建议在Linux环境下，通过`ssh-keygen -t rsa -C "xxx@xxx.com"`进行生成。我用putty在win10下生成的key，通过putty可以登录，通过命令提示行登录总是失败，导致我最初用putty生成的私钥进行测试的时候，ssh一直不通，为此我换了几个ssh的Action，非常浪费时间。

4.2 环境变量的问题，我在`/github/workflows/deploy.yml`中配置的env在执行Deploy to server的过程中未生效，后来我索性改为secretes方式，然后问题解决。env不生效的原因，猜测是因为登录到了新的服务器中，而不是在Action环境里面。
