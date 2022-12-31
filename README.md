# 准备工作：

> 所有工作都在服务器上执行!
> 客户端推荐使用 FinalShell 

### 如果系统是  openSUSE , 则先执行 prepare.sh
```
./prepare.sh
```

### 安装 docker 
```
cat <<EOF > /etc/docker/daemon.json
{
"registry-mirrors":[
    "https://hub-mirror.c.163.com",
    "https://registry.aliyuncs.com"
],
"graph":"/var/lib/docker",
"graph":"/var/lib/docker",
"insecure-registries":["localhost:8000"],
"features": { "buildkit": true }
}
EOF
```

执行：
`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`

如果是阿里云服务器，执行 
```
dnf config-manager --add-repo=https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

dnf -y install dnf-plugin-releasever-adapter --repo alinux3-plus
dnf -y install docker-ce --nobest

```

### 拉取 akstream:v2
```
docker pull registry.cn-hangzhou.aliyuncs.com/jgcx/akstream:v2
docker tag registry.cn-hangzhou.aliyuncs.com/jgcx/akstream:v2 akstream:v2
```


### Git Clone
获取 github.com  的IP，通过： https://tool.chinaz.com/dns  查询获得。
```
cd /root
docker run --rm -it --add-host  github.com:140.82.113.3   -v /root:/git/repo  wuliangxue/git:0.1 git clone https://github.com/iamnewsea/AKStreamDocker.git
```

### 安装 mysql

```
docker run --name mysql  -v /root/AKStreamDocker/mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=AKStream2021@  --net=host -d mysql:5.7.35
```

### 确定服务器IP：
```
ifconfig 
找出 eth0 IP： 一般是 10.x.x.x
```

# 启动 akstream:v2 容器
```
./start.sh
```
或：
```
docker run --privileged=true -d --name akstream_dev -v $PWD/mysql_data:/var/lib/mysql -v $PWD/video_record:/root/AKStreamKeeper/record -v /sys/fs/cgroup:/sys/fs/cgroup:ro -v /etc/timezone:/etc/timezone:ro -v /etc/localtime:/etc/localtime:ro --tmpfs /run:rw --net=host akstream:v2 /usr/sbin/init
```


# 进入容器， 启动服务


`docker exec -it akstream_dev bash`

### 修改文件

在docker 内部执行：

`vi /root/AKStreamKeeper/Config/AKStreamKeeper.json`
修改 IpV4Address， AkStreamWebRegisterUrl

`vi /root/AKStreamWeb/Config/AKStreamWeb.json`
修改数据库连接字符串中的 localhost 为 10.x.x.x

```
vi /root/AKStreamNVR/public/env-config.js IP 为 10.x.x.x

window._env_ = {
REACT_APP_API_HOST: "xx.xx.xx.xx",
AKSTREAM_WEB_API:"xx.xx.xx.xx:5800",
}
```

### 执行：
在docker 内部执行：

``` 
cd ~/AKStreamWeb
dotnet AKStreamWeb.dll > /dev/null &

执行回车

cd ~/AKStreamKeeper
dotnet AKStreamKeeper.dll > /dev/null &

执行回车

cd ~/AKStreamNVR

npm run start &

```


# 其它问题参考 [原 readme.md](./ori-readme.md)


