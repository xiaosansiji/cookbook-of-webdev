# Docker 

## 常用指令

查看镜像id：`docker images -q` 
删除镜像：`docker rmi image_id` 
删除所有镜像：`docker rmi $(docker images -q)` 
创建容器：`docker run --name <container_name> image_i`
查看所有容器：`docker ps -a` 
查看运行容器：`docker ps` 
查看容器id：`docker ps -q` 
进入容器：`docker exec -it <container_id> bash` ，基于 alpine 的镜像没有 /bin/bash 可用，可以使用 `docker exec -it <container_id> sh`
退出容器：`exit` 
删除容器：`docker rm <container_id>` 
删除所有容器：`docker rm $(docker ps -aq)` 

根据dockerfile打镜像：`docker build -t <image_name> [-f <dockerfile位置，可以是相对路径>]  <路径>`
给镜像改名字：`docker tag <old_name>:<new_name>`
发布镜像：`docker push <image_name>`