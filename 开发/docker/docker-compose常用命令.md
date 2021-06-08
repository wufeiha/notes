| docker-compose up -d    | 推荐使用，，，-d代表后台运行                                 |
| ----------------------- | ------------------------------------------------------------ |
| docker-compose ps       | 列出所有运行中的容器,类似docker ps                           |
| docker-compose logs web | 查看web服务日志                                              |
| docker-compose stop     | Stop a running container by sending SIGTERM and then SIGKILL after a grace period |
| docker-compose down     | 停止并移除容器、网络.比stop更彻底                            |
| docker-compose restart  | 重启YAML文件中定义的服务                                     |
| docker-compose kill     | Kill a running container using SIGKILL or a specified signal |
| docker-compose rm       | 删除指定已经停止服务的容器                                   |
| docker-compose build    | 构建或重建服务                                               |
| docker-compose images   | 列出所有镜像                                                 |
| docker-compose pull     | 拉去并下载指定服务镜像                                       |
| docker-compose push     | push服务镜像                                                 |
| docker-compose top      | 显示各个容器内运行的进程                                     |

Docker-compose大多命令作用与docker类似

**up，down和start, stop,restart的区别**
docker-compose restart不会加载新的docker-compose相关文件的新的改动（如，docker-compose.yml文件）docker-compose down ; docker-compose up 一定会是全部最新的内容

如果只是改动代码和配置文件等，docker-compose restart是没有问题的，如果改动了docker-compose.yml里的内容，则必须down之后再up （restart是无法自动加载新变动的内容的）,如果修改了dockerfile需要重建镜像，要携带**--build**参数

**docker restart 和docker-compose restart 的区别**
docker restart是使用的是容器全名，docker-compose restart使用的是docker-compose.yml里定义的别名