##### 1.为什么要有docker

​	跨平台

1. 快速构建应用
2. 快速分享应用
3. 快速运行应用

##### 2.docker如何工作

![image-20240624192149652](./assets/image-20240624192149652.png)

![image-20240624221535756](./assets/image-20240624221535756.png)



##### 3.镜像命令

检索: docker search

下载: docker pull

列表: docker images

删除: docker rmi



##### 4.容器命令

运行: docker run  *

```sh
docker run 
				   -d 
					 --name mynginx 
					 -p 80:80
					 nginx
```

查看: docker ps

停止: docker stop

启动: docker start

重启: docker restart

状态: docker stats

日志: docker logs

进入: docker exec  *

```sh
docker exec
						-it
						mynginx /bin/bash
```

删除: docker rm





##### 5.分享命令

提交: docker commit

保存: docker save

加载: docker load

登录: docker login

命名: docker tag

推送: docker push





##### 6.命令的丰富使用

强制删除所有容器: docker rm -f $(docker ps -aq)

目录挂载: docker run -d -p 80:80 -v /app/nghtml:/usr/share/nginx/html

卷映射: 





















