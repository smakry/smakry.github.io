---
layout: post
title: 'python调用docker'
date: 2022-04-18
author: smakry
tags: python docker
---

> 相互作用

### 环境
- python3
- docker

``` py
一、创建客户端：要与Docker守护程序进行通信，您首先需要实例化一个客户端。
1.最简单的方法是调用函数from_env()

import docker
docker.from_env()

2.也可以通过实例化一个DockerClient类来手动配置它。

import docker
docker_client = docker.DockerClient(**params)
 
参数params详解：
base_url (str): 指定链接路径，可以通过socket或者tcp方式链接unix:///var/run/docker.sock 或者 tcp://0.0.0.0:2375
version (str): 指定API使用的版本(docker=2.0.0默认的api版本是1.24,最低支持1.21,docker1.9+的api是1.21),因此在使用python的docker模块时一定要注意docker的api以及docker模块的api是否兼容。当然如果设置为auto降回去自动检测server的版本
timeout (int): 使用API调用的默认超时时间，默认单位为秒
tls (bool:class): 启用TLS。通过True使用默认选项启用它，或传递py:类，通过~docker.tls.tls配置`对象以使用自定义配置。
use_ssh_client (bool): 如果客户机的ssh连接设置为“True”，则通过ssh将连接设置为“True”。确保在主机上安装并配置了ssh客户机。
max_pool_size (int): 最大连接数
远程连接
docker守护进程配置
修改/lib/systemd/system/docker.service文件
[Service]
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2376 --tlsverify --tlscacert=/etc/docker/ssl/ca.pem --tlscert=/etc/docker/ssl/server-cert.pem --tlskey=/etc/docker/ssl/server-key.pem
 
参数详解：
-H tcp://0.0.0.0:2375: 允许远程连接和开启端口
/etc/docker/ssl/ca.pem: TLS证书路径
/etc/docker/ssl/server-cert.pem: TLS服务器证书签名
/etc/docker/ssl/server-key.pem: TLS服务器秘钥
加载重启服务
systemctl daemon-reload
systemctl restart docker
 
docker客户端配置
import docker
 
# 开启TLS验证
tls_config = docker.tls.TLSConfig(
    client_cert = ('/etc/docker/ssl/cert.pem', '/etc/docker/ssl/key.pem'),   # 客户端签名证书，客户端密钥
    ca_cert = "/etc/docker/ssl/ca.pem",       # 证书路径
    verify = "/etc/docker/ssl/ca.pem",        # 证书路径
    ssl_version = 2                           # SSL版本
)
 
client = docker.DockerClient(base_url = "tcp://IP:Port", tls = tls_config)
 
参数详解：
client_cert(tuple of str): TLS客户端证书签名，客户端密钥
ca_cert(str): TLS证书文件路径
verify(bool or str): 验证证书，可以是False，也可以是CA证书文件路径
ssl_version(int): SSL版本号
assert_hostname(bool): 验证服务器主机名
IP:Port，服务器的IP和docker守护进程的端口
TLS证书配置
Docker TLS docs
使用客户端
注释：以下的client全部都是使用DockerClient实例化的对象
管理docker配置的对象（configs）
管理docker容器的对象（containers）

二、新建容器（docker run）
client.containers.run(image, command=None, stdout=True, stderr=False, remove=False, **kwargs)
>>> <docker.models.containers.ContainerCollection object at 0x7fcfc081a250>
 
参数详解：
image(str): 需要运行的镜像
command(str or list): 运行镜像在容器中执行的命令
auto_remove(bool): 当容器的进程退出时，在守护程序端启用容器的自动删除
detach(bool): 后端运行容器并返回一个Container对象
hostname(str): 容器的主机名
name(str): 容器名字
ports(dict): 要在容器内绑定的端口，以及宿主机的映射端口，例如将容器的80端                   口映射成宿主机的8080端口{'80/tcp': 8080}
remove(bool): 删除容器，默认是False
dns(:py:class:`list`): 设置自定义DNS服务器，例：dns=['192.168....']
extra_hosts(dict): 要在容器内解析的其他主机名，作为主机名到IP地址的映射
                      例如：extra_hosts={'host1': '192.168.164.21'}
mac_address(str): 要分配给容器的mac地址
tty(bool): 分配一个伪终端
network(str): 指定docker网络ID或者名称
mem_limit(float or str): 内存限制
network_disabled(bool): 禁止联网
working_dir(str): 工作目录路径
cpu_count(int): 可用cpu的数量（仅限Windows）
cpu_shares(int): CPU份额（相对权重）
devices(list): 以字符串列表的形式将主机设备公开给容器，格式为
            <path_on_host>:<path_in_container>:<cgroup_permissions>
 /dev/sda:/dev/xvda:rwm将主机的/dev/sda暴露给容器的/dev/xvda
network_mode(str): 网络模式
bridge: 为网桥网络上的容器创建新的网络
none: 此容器没有网络（设置为none时只有本地回环）
container:<name|id>: 重用另一个容器的网络
*** 使用这种模式时不能给容器设置名称
host: 使用主机网络（和主机相同的网络设备）
read_only(bool): 将容器的根文件系统挂载为只读
remove(bool): 在容器运行完毕时将容器移除
volume_driver(str): 卷驱动程序/插件的名称
volumes(dict or list): 用于配置装载在容器内的卷的字典。键是主机路径或卷         名，值是包含键的字典
bind: 在容器内装入卷的路径
mode: rw以读/写方式装入卷，或ro以只读方式装入卷
例如：
{
    '/home/user1/':
        {'bind':'/mnt/vol2','mode':'rw'},
    '/var/www':
        {'bind':'/mnt/vol1','mode':'ro'}
}
 
volumes_from(list): 要从中获取卷的容器名称或ID的列表
init(bool): 在容器中运行初始化，该容器转发信号并获取进程
异常处理：
docker.errors.ContainerError   容器以非零退出代码退出且detach为False
docker.errors.ImageNotFound    指定的映像不存在
docker.errors.APIError          服务器返回错误
新建容器（docker create）
client.containers.create(image, command=None, **kwargs)
>>> <Container: bdcbbcda1d>
 
参数详解
image(str): 需要运行的镜像
common(str): 需要执行的命令
异常处理：
docker.errors.ImageNotFound    指定的映像不存在
docker.errors.APIError          服务器返回错误
获取所有容器（docker ps）
client.containers.list(all=False, before=None, filters=None, limit=-1, since=None, sparse=False, ignore_removed=False)
>>> [<Container: 8fe86b093b>]
 
参数详解
all(bool): 返回所有的存活的、退出的容器，docker ps -a  
before(str): 仅显示在Id或名称之前创建的容器，包括非运行的
limit(int): 表示到此为止前面创建的容器
filters(dict): 筛选器，具体的规则见如下
exited(int): 仅限具有指定退出代码的容器
status(str): 仅限具有指定状态的容器
restarting、running、paused、exited
id(str): 仅限具有指定标识符的容器
name(str): 仅限具有指定名称的容器
异常处理：
docker.errors.APIError          服务器返回错误
获取指定容器
container = client.containers.get(id_or_name)
>>> <Container: 8fe86b093b>
 
参数详解：
name(str): 容器的名称或ID
异常处理：
docker.errors.NotFound          指定的容器不存在
docker.errors.APIError          服务器返回错误
获取容器/客户端对象
client.containers.get(id or name).collection
>>> <docker.models.containers.ContainerCollection object at 0x7f7207494b50>
client.containers.get(id or name).client
>>> <docker.client.DockerClient object at 0x7f276d151610>
 
停止/启动容器（docker stop）
container.stop(timeout)
>>> None
container.start()    # 不支持附加选项
>>> None
 
参数详解 
timeout(int): 停止超时时间，默认是10S
异常处理：
docker.errors.APIError          服务器返回错误
更新容器（docker update）
container.update(**kwargs)
 
参数详解
blkio_weight（int）: Block IO（相对重量），介于10和1000之间
cpu_period（int）: 限制cpu CFS（完全公平调度程序）周期
mem_limit（int或str）: 内存限制
mem_reservation（int或str）: 内存软限制
memswap_limit（int或str）: 总内存（内存+交换），-1到禁用交换
异常处理：
docker.errors.APIError          服务器返回错误
重启容器（docker restart）
container.restart(timeout)
 
参数详解 
timeout(int): 重启超时时间，默认是10S
异常处理：
docker.errors.APIError          服务器返回错误
删除容器（docker rm）
container.remove(**kwargs)
>>> None
 
参数详解
v(bool): 删除与容器关联的卷
link(bool): 删除指定的链接，而不是基础链接
force（bool）: 强制移除正在运行的容器
异常处理：
docker.errors.APIError          服务器返回错误
修剪容器
client.container.prune(filters=None)
>>> {'ContainersDeleted': None, 'SpaceReclaimed': 0}
 
参数详解：
filters（dict）: 要在修剪列表上处理的筛选器
异常处理：
docker.errors.APIError          服务器返回错误
监控容器资源
container.stats(**kwargs)
>>> <generator object APIClient._stream_helper at 0x7f5e4e7a2650>
>>> {'read': '2020-12-17T06:49:09.934015394Z', 'preread': '2020-12-17T06:49:08.93188573Z', ...'tx_packets': 0, 'tx_errors': 0, 'tx_dropped': 0}}}
 
 
参数详解：
decode(bool): 如果设置为true，流将被解码为dict，仅当“stream”为True时才                   适用。默认为False
stream(bool): 如果设置为false，则只有当前的统计信息，而不是流。默认为                       True。
异常处理：
docker.errors.APIError          服务器返回错误
其他容器API
获取容器状态/容器信息
container.status
>>> 'running', 'exited', 'created', ......
container.attrs
>>> {'Id': '5f44feaa9b55c07542e82f485d51795bdced7e5900ceae86349f3746f100b723', ...}
 
连接指定容器
container.attach(**kwargs)
>>> b''
 
参数详解：
stdout(bool): 标准输出
stderr(bool): 标准错误输出
stream(bool): 以迭代器的形式逐步返回容器输出
logs(bool): 包括容器以前的输出
提交容器到镜像
container.commit(repository=None, tag=None, **kwargs)
>>> <Image: ''>
 
参数详解：
repository（str）: 将图像推送到的存储库
tag（str）: 要推送的标记
message（str）: 提交消息
author（str）: 作者的姓名
changes（str）: 提交时要应用的Dockerfile指令
conf（dict）: 容器的配置
异常处理：
docker.errors.APIError          服务器返回错误
阻塞容器
container.wait(**kwargs)
 
参数详解
timeout(int): 超时时间
condition(str): 容器停止阻塞的条件
not-running(default)、next-exit、removed
异常处理：
docker.errors.APIError          服务器返回错误
requests.exceptions.ReadTimeout 如果超时
容器执行cmd（docker exec）
container.exec_run(cmd, tty=False， stdout=True, stderr=True, stdin=False， user='', detach=False， workdir=None)
>>> (exit_code=0, output=b'search 192.168.164.21 8.8.8.8\nnameserver 8.8.8.8\nnameserver 114.114.114.114\nnameserver 192.168.1.103\n')
 
参数详解：
cmd(str or list): 要执行的命令
tty（bool）: 分配一个伪tty。默认为False
workdir（str）: 此exec会话的工作目录的路径
user（str）: 执行命令的用户
stdout（bool）: 附加到stdout。默认为True
stderr（bool）: 连接到stderr。默认为True
stdin（bool）: 连接到stdin。默认为False
异常处理：
docker.errors.APIError          服务器返回错误
更改容器名称（docker rename）
container.rename(name)
>>> None
 
参数详解
name(str): 最终更改的容器名称
异常处理：
docker.errors.APIError          服务器返回错误
向容器传递文件（docker cp）
container.put_archive(path, data)
>>> True
 
参数详解
path（str）: 容器中文件（文件夹）所在的路径
data（bytes）: 要提取的tar数据
注释：
1. 源主机需要复制的文件（文件夹）一定是以tar格式结尾的文件（文件夹）
2. 可以使用tarfile模块将源主机文件（文件夹）压缩成tar文件（文件夹）
3. 源主机传入的data参数必须是字节格式的，可以open().read()进行转换
4. 目标主机接收到的data值是已经解包以后的文件（文件夹）
代码注释：
import os, tarfile
def copy_to(src, dst):
    name, dst = dst.split(':')
    container = client.containers.get(name)
    os.chdir(os.path.dirname(src))
    srcname = os.path.basename(src)
    tar = tarfile.open(src + '.tar', mode='w')
    try:
        tar.add(srcname)
    finally:
        tar.close()
    data = open(src + '.tar', 'rb').read()
    container.put_archive(os.path.dirname(dst), data)
 
copy_to('/root/test.py', '0a8f97f869f3:/root/test.py')
 
异常处理：
APIError            如果发生错误
从容器获取文件
container.get_archive(path, chunk_size=DEFAULT_DATA_CHUNK_SIZE,
                encode_stream=False)
>>> (<generator object APIClient._stream_raw_result at 0x7f0860da75d0>, {'name': 'test.py', 'size': 7, 'mode': 420, 'mtime': '2020-12-16T09:16:20+08:00', 'linkTarget': ''})
 
参数详解：
path（str）: 容器中文件（文件夹）所在的路径
chunk_size: 每次迭代返回的字节数
encode_stream: 确定是否应该对数据进行编码
注释：
1. 返回的是元组，第一个值为文件（文件夹的tar元数据），第二个为文件（文件夹）详解
2. tar元数据为一个生成器，使用next或者循环去访问
代码详解：
>>> f = open('/root/test.py.tar', 'wb')
>>> bits, stat = container.get_archive('/root/test.py')
>>> print(stat)
{'name': 'test.py', 'size': 7, 'mode': 420, 'mtime': '2020-12-16T09:16:20+08:00', 'linkTarget': ''}
>>> for chunk in bits:
...    f.write(chunk)
>>> f.close()
>>> tar -xvf /root/test.py.tar
test.py
>>> cat test.py
111111
 
异常处理：
docker.errors.APIError          服务器返回错误
打包容器文件系统
container.export(chunk_size=DEFAULT_DATA_CHUNK_SIZE)
>>> <generator object APIClient._stream_raw_result at 0x7f90fd2bb5d0>
 
参数详解：
chunk_size: 每次迭代返回的字节数
注释：
1. 结果解析方式和get_archive相同
异常处理：
docker.errors.APIError          服务器返回错误
修改TTY大小
container.resize(height, width)
>>> None
 
参数详解
height(int): 会话高度
width(int): 会话宽度
异常处理：
docker.errors.APIError          服务器返回错误
容器文件更改
container.diff()
>>> [{'Path': '/tt.py', 'Kind': 1}]
 
异常处理：
docker.errors.APIError          服务器返回错误
暂停/恢复/显示容器内进程
container.pause()
>>> None
container.unpause()
>>> None
container.top()
>>> {'Processes': [['root', '49681', '49660', '0', '14:57', 'pts/0', '00:00:00', '/bin/bash']], 'Titles': ['UID', 'PID', 'PPID', 'C', 'STIME', 'TTY', 'TIME', 'CMD']}
 
异常处理：
docker.errors.APIError          服务器返回错误
获取容器ID/short_id/开放端口/标签
container.id
>>> 39352d85616ddbb18fa6a2d40cb979b0240a5f392bd656....
container.short_id
>>> 39352d8561
container.ports
>>> {'80/tcp': [{'HostIp': '0.0.0.0', 'HostPort': '8080'}]}
container.labels
>>> {'org.label-schema.build-date': '20201204', 'org.label-schema.license': 'GPLv2', 'org.label-schema.name': 'CentOS Base Image', 'org.label-schema.schema-version': '1.0', 'org.label-schema.vendor': 'CentOS'}
 
重新加载/强制杀死容器
container.reload()
>>> None
container.kill()
>>> None
 
注释：
再次从服务器加载此对象，并使用更新“attrs”
异常处理：
docker.errors.APIError          服务器返回错误
查看容器日志
container.logs()
>>> b''
 
参数详解：
stdout(bool): 标准输出Default ``True``
stderr(bool): 标准错误输出Default ``True``
stream(bool): Default ``False``
timestamps(bool): 显示时间戳Default ``False``
tail(str or int): 日志。行数的整数或字符串Default ``all``
since(datetime or int): 显示自给定日期时间
follow(bool): 跟踪日志输出Default ``False``
until(datetime or int): 显示在给定的整数
异常处理：
docker.errors.APIError          服务器返回错误
管理docker镜像的对象（images）
构建镜像（docker build）
client.images.build(**kwargs)
>>> (<Image: 'vv1:latest'>, <itertools._tee object at 0x7fe5f00f6e10>)
 
参数详解：
path(str): 包含Dockerfile文件的路径（目录）
fileobj: 用作Dockerfile的file对象（或类似于对象）
tag(str): 要添加到最终图像的标记
quiet(bool): 是否返回状态
nocache(bool): 设置为“True”时不要使用缓存
rm(bool): 移除中间容器，默认为False
timeout(int): 超时时间，单位为秒
custom_context(bool): 如果使用fileobj，则为可选
encoding（str）：流的编码。设置为“gzip”表示压缩
pull(bool): 在Dockerfiles中下载对FROM映像的更新
forcerm(bool): 始终移除中间容器，即使构建镜像不成功
dockerfile(str): 构建上下文中dockerfile的路径
buildargs(dict): 构建参数的字典
container_limits(dict): 配置参数
memory (int): 为构建设置内存限制
memswap (int): 总内存（内存+交换），-1表示禁用交换
   cpushares (int): CPU份额（相对权重）
   cpusetcpus (str): 允许执行的cpu，例如“0-3”，“0,1”
shmsize(int): /dev/shm的大小（以字节为单位）。尺寸必须是大于0。默认64MB
labels(dict): 为一个镜像设置一个字典格式的标签
cache_from(list): 用于构建缓存的映像列表
target(str): 要在多阶段中生成的名称
network_mode(str): 网络模式
squash(bool): 将生成的图像层压缩到单个层中
extra_hosts(dict): 要添加到构建容器中的/etc/hosts的额外主机，作为主机名          到IP地址的映射。
platform(str): 平台 in the format ``os[/arch[/variant]]``.
isolation(str): 构建期间使用的隔离技术。默认值None
use_config_proxy(bool): 如果为True，并且docker客户端配置文件，包含一个            代理配置，对应的环境，将在正在生成的容器中设置变量
Dockerfile文件详解：
1. dockerfile是用来构建docker镜像的文本文件，包含构建镜像所需的指令和说明
构建nginx镜像为例：
1. 在一个空目录下新建一个名为Dockerfile文件，添加一下内容
FROM nginx
RUN yum install -y wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz
 
FROM: 定制的镜像都是基于FROM的镜像，这里的nginx就是定制需要的基础镜像
RUN: 用于指定后面跟着的命令，有以下两种方式
# shell格式
RUN <命令行命令>
# exec格式
RUN ["可执行文件", "参数1", "参数2"]
 
2. 在Dockerfile中添加自己想要内容
指令详解：
1. COPY（复制指令，从上下文目录中复制文件或目录到容器的指定路径）
格式
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
 
[--chown=<user>:<group>]: 用户改变复制到容器内文件的拥有者和属组
<源路径>: 源文件或者源目录，这里可以是通配符表达式，例如
COPY hom* /mydir/
COPY hom?.txt /mydir/
 
<目标路径>: 容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建
2. ADD（复制指定）
ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐COPY）。功能类似，不同之处如下
在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>
在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定
3. CMD（类似于RUN指令，用于运行程序）
CMD 在docker run 时运行
RUN 在docker build时运行
为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖
如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效
CMD <shell 命令>
CMD ["<可执行文件或命令>","<param1>","<param2>",...]
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
 
4. ENTRYPOINT（类似于CMD指令，用于运行程序）
其不会被docker run的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给ENTRYPOINT指令指定的程序
运行docker run时使用了 --entrypoint选项就会覆盖ENTRYPOINT选项
在执行docker run的时候可以指定ENTRYPOINT运行所需的参数
如果Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效
异常处理：
docker.errors.APIError          服务器返回错误
docker.errors.BuildError        如果生成过程中出现错误
TypeError                       如果既没有指定路径也没有指定fileobj
获取所有镜像（docker images）
client.images.list(name=None, all=False, filters=None)
>>> [<Image: 'alpine:latest'>, <Image: 'ubuntu:latest'>, <Image: 'sysmag_def-x86_o77-4.4.216-ds_s.k.c.n.t:4.x'>, <Image: 'bfirsh/reticulate-splines:latest'>]
 
参数详解：
name(str): 仅显示属于存储库name的图像
all(bool): 显示中间图像层。默认情况下，这些是过滤掉
异常处理：
docker.errors.APIError          服务器返回错误
获取指定镜像
client.images.get(id or name)
>>> <Image: 'centos/httpd-24-centos7:latest'>
 
参数详解：
name(str): 仅显示属于存储库name的镜像
异常处理：
docker.errors.APIError          服务器返回错误
docker.errors.ImageNotFound     如果图像不存在
获取镜像/客户端对象
client.images.get(id or name).collection
>>> <docker.models.images.ImageCollection object at 0x7f021a583710>
client.images.get(id or name).client
>>> <docker.client.DockerClient object at 0x7f28f0443690>
 
查找镜像（docker search）
client.images.search()
 
参数详解：
term(str): 需要搜素的镜像名称，例如client.images.search('ubuntu')
异常处理：
docker.errors.APIError          服务器返回错误
拖取镜像（docker pull）
更新为国内镜像源（如果还是比较慢可以申请阿里镜像加速器）
[root@node1 ~]# cat /etc/docker/daemon.json
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
[root@node1 ~]# systemctl daemon-reload
[root@node1 ~]# systemctl restart docker
 
client.images.pull(repository, tag=None,all_tags=False)
>>> <Image: 'centos/httpd-24-centos7:latest'>
 
参数详解：
repository(str): 需要拖取的镜像名称
tag(str): 拖取镜像的标签
all_tags(bool): 拖取所有类型的标签
推送镜像（docker push）
client.images.push(repository, tag=None, stream=False, auth_config=None,decode=False)
>>> for i in client.images.push('yourname/app', stream=True, decode=True):
    print(i)
>>> {'status': 'Pushing repository yourname/app (1 tags)'}{'status': 'Pushing','progressDetail': {}, 'id': '511136ea3c5a'}{'status': 'Image already pushed, skipping', 'progressDetail':{}, 'id': '511136ea3c5a'}
 
参数详解：
repository（str）: 要推送到的存储库
tag（str）: 要推送的可选标记
stream（bool）: 将输出流化为阻塞生成器
auth_config（dict）: 重写凭据应包含username和password有效。
decode（bool）: 将服务器中的JSON数据解码为dict。
返回值：
(generator or str): 服务器的输出
异常处理：
docker.errors.APIError          服务器返回错误
删除镜像（docker rmi）
client.images.remove(id or name)
>>> None
 
参数详解：
name(str): 镜像的名称或者ID
force(bool): 强制删除镜像
noprune(bool): 不要删除未标记的父级
保存对象（docker save）
image = client.images.get('6bd81a343bae')
image.save(chunk_size=DEFAULT_DATA_CHUNK_SIZE, named=False)
>>> <generator object APIClient._stream_raw_result at 0x7f394cf005d0>
 
参数详解：
chunk_size(int): 每次迭代返回值的大小，默认为2M
named(bool): 是否保留此图像的存储库和标记信息
注释：
1. 其接口的返回值处理方法和容器的get_archive功能相同
异常处理：
docker.errors.APIError          服务器返回错误
载入镜像（docker load）
# 简单的情况（没有用，只是讲解思路）
image = client.images.get('6bd81a343bae').save()
client.images.load(image)
# 真实情况
f = open('/root/images/test.tar', 'rb')
client.images.load(f)
f.close()
>>> [<Image: 'centos/httpd-24-centos7:latest'>]
 
参数详解：
data(binary): 需要导入成镜像的文件流
异常处理：
docker.errors.APIError          服务器返回错误
归入仓库（docker tag）
client.images.tag(repository, tag=None, **kwargs)
>>> True
 
参数详解：
repository(str): 要为标记设置的存储库
tag(str): 标签名
force(bool): 强制执行
注释：
归纳镜像到仓库中去
修剪镜像（docker prune）
client.images.prune(filters=None)
>>> {'ImagesDeleted': None, 'SpaceReclaimed': 0}
 
参数详解：
filters(dict): 过滤器
dangling(bool): 设置为true时，仅修剪未使用和未标记的图像
注释：
删除未使用的镜像
异常处理：
docker.errors.APIError          服务器返回错误
删除缓存镜像
client.images.prune_builds()
>>> {'CachesDeleted': None, 'SpaceReclaimed': 0}
 
异常处理：
docker.errors.APIError          服务器返回错误
其他镜像API
获取镜像ID/short_id/labels/历史信息/仓库标签/详细信息/注册表数据
client.images.get(name).id
>>> dd85cdbb99877b7325af590ab188d79469eebdb23ec2d26d0d10e8....
client.images.get(name).short_id
>>> dd85cdbb9987
client.images.get(name).labels
>>> {}
client.images.get(name).history()
>>> [{'Comment': '', 'Created': 1607689205, 'CreatedBy': '/bin/sh -c #(nop)  CMD ["httpd-foreground"]', 'Id': 'sha256:dd85cdbb99877b73f0de2053f225af590ab188d79469eebdb23ec2d26d0d10e8', 'Size': 0, 'Tags': ['httpd:latest', 'test:test']}, {'Comment': '', 'Created': 1607689205, 'CreatedBy': '/bin/sh -c #(nop)  EXPOSE 80', 'Id': '<missing>', 'Size': 0, 'Tags': None}, ......]
client.images.get(name).tags
>>> ['httpd:latest']
client.images.get(name).attrs
>>> {'Id': 'sha256:6bd81a343bae01091a0d0e125b74b282d0475242301d4d95a10774b7478caa3c', 'RepoTags': ['centos/httpd-24-centos7:latest'],.....}
client.images.get_registry_data(name)
>>> <RegistryData: sha256:035ce6b3c5>
 
参数详解：
name(str): 镜像的名称或ID
异常处理：
docker.errors.APIError          服务器返回错误
重新加载镜像对象
client.images.get(name).reload()
>>> None
 
注释：
再次从服务器加载此对象，并使用更新“attrs”
异常处理：
docker.errors.APIError          服务器返回错误
管理docker网络的对象（networks）
新建网络（docker network create）
client.networks.create(name, *args, **kwargs)
>>> <Network: 106469d8be>
 
[root@node2 ~]# docker network list
NETWORK ID     NAME       DRIVER    SCOPE
f40a6f0f574d   bridge     bridge    local
 
创建高级网络
>>> ipam_pool = docker.types.IPAMPool(
            subnet='192.168.52.0/24', 
            gateway='192.168.52.254'
            )
>>> ipam_config = docker.types.IPAMConfig(
                       pool_configs=[ipam_pool]
                       )
>>> client.networks.create(
                        "network1",
                        driver="bridge",   
                        ipam=ipam_config
                        )
>>> <Network: b46e3db35c>
 
参数详解：
name(str): 网络名称
driver(str): 用于创建网络的驱动程序的名称
options(dict): 作为键值字典的驱动程序选项
ipam(IPAMConfig): 网络的可选自定义IP方案
check_duplicate(bool): 请求守护进程检查同名网络。默认值：None
internal(bool): 限制对网络的外部访问。默认值：False
labels(dict): 要在网络上设置的标签的映射。默认值：None
enable_ipv6(bool): 在网络上启用IPv6。默认值：False
attachable(bool): 如果启用，并且网络在全局范围内，工作节点上的非服务容器将                  能够连接到网络
scope(str): 指定网络范围(local, global， swarm)
ingress(bool): 如果设置，则创建一个入口网络，该网络提供
异常处理：
docker.errors.APIError          服务器返回错误
获取客户端对象
client.networks.client
>>> <docker.client.DockerClient object at 0x7f83a0707590>
 
获取全部网络
client.networks.list(*args, **kwargs)
>>> [<Network: f40a6f0f57>]
 
参数详解：
names(:py:class:`list`): 要筛选的名称列表
ids(:py:class:`list`): 要筛选的ID列表
filters(dict): 要在网络列表上处理的筛选器
    过滤器:
    driver=[<driver-name>]: 匹配网络驱动程序.
    label(str|list): 格式化key、key=value或类似的列表.
    type=["custom"|"builtin"]: 按类型筛选网络
greedy(bool): 分别获取每个网络的更多详细信息。你可能想用这个来连接容器.
异常处理：
docker.errors.APIError          服务器返回错误
获取指定网络
client.networks.get(network_id, *args, **kwargs)
>>> <Network: 106469d8be>
 
参数详解：
network_id(str): 网络的ID或者名称
verbose(bool): 以swarm模式检索整个集群的服务详细信息
scope(str): 按作用域筛选网络(swarm, global or local)
异常处理：
docker.errors.APIError          服务器返回错误
docker.errors.NotFound          网络不存在
获取网络信息
client.networks.get(id or name).attrs
>>> {'Name': 'bridge', 'Id': 'f40a6f0f574d194a1af1de07c7d62c314ae86106ca117cfaec621a1fa1a5ae29', 'Created': '2020-12-17T07:44:30.885322292+08:00', 'Scope': 'local', 'Driver': 'bridge', 'EnableIPv6': False, 'IPAM': {'Driver': 'default', 'Options': None, 'Config': [{'Subnet': '172.17.0.0/16', 'Gateway': '172.17.0.1'}]}, 'Internal': False, 'Attachable': False, 'Ingress': False, 'ConfigFrom': {'Network': ''}, 'ConfigOnly': False, 'Containers': {'5f44feaa9b55c07542e82f485d51795bdced7e5900ceae86349f3746f100b723': {'Name': 'frosty_villani', 'EndpointID': '28881a7b2af05d4d6f1d599b92416ff46cf8b30190797c738d384321e4714941', 'MacAddress': '02:42:ac:11:00:02', 'IPv4Address': '172.17.0.2/16', 'IPv6Address': ''}}, 'Options': {'com.docker.network.bridge.default_bridge': 'true', 'com.docker.network.bridge.enable_icc': 'true', 'com.docker.network.bridge.enable_ip_masquerade': 'true', 'com.docker.network.bridge.host_binding_ipv4': '0.0.0.0', 'com.docker.network.bridge.name': 'docker0', 'com.docker.network.driver.mtu': '1500'}, 'Labels': {}}
 
获取网络/客户端对象
client.networks.get(id or name).collection
>>> <docker.models.networks.NetworkCollection object at 0x7f0ce9cd1c10>
client.networks.get(id or name).client
>>> <docker.client.DockerClient object at 0x7f29f6e1a0d0>
 
容器连接网络
container = client.containers.get('5f44feaa9b55')
network = client.networks.get('d58f8bf0bbf8', verbose=False, scope='local')
network.connect(container)
>>> None
 
参数详解：
container(str): 容器的对象
aliases(:py:class:`list`): 此别名列表端点。名称在该列表中可以使用网络内                                      部的容器。默认为None
links(:py:class:`list`): 此链接的列表终结点。容器在此列表中声明的将链接                                        到此容器。默认为None
ipv4_address(str): 网络上使用IPv4协议的此容器的IP地址。默认为None
ipv6_address(str): 网络上使用IPv6协议的此容器的IP地址。默认为None
link_local_ips(:py:class:`list`): 链路本地（IPv4/IPv6）地址的列表
driver_opt(dict): 提供给网络驱动程序的选项字典。默认为None
异常处理：
docker.errors.APIError          服务器返回错误
获取网络上容器
client.networks.get(id or name).containers
>>> [<Container: 5f44feaa9b>]
 
删除网络上容器
container = client.containers.get('5f44feaa9b55')
network = client.networks.get('d58f8bf0bbf8', verbose=False, scope='local')
network.disconnect(container)
>>> None
 
参数详解：
container(str): 容器对象
force(bool): 强制容器断开与网络的连接，Default:False
异常处理：
docker.errors.APIError          服务器返回错误
获取网络ID/名称/short_id
client.networks.get(id or name).id
>>> f40a6f0f574d194a1af1de07c7d62c314ae86106ca117cf...
client.networks.get(id or name).name
>>> bridge
client.networks.get(id or name).short_id
>>> f40a6f0f57
 
重载/删除/修剪网络
client.networks.get(id or name).reload()
>>> None
client.networks.get(id or name).remove()
>>> None
client.networks.prune()
>>> {'NetworksDeleted': ['network1', 'network2']}
 
管理docker存储卷的对象（volumes）
获取存储卷
client.volumes.list()
>>> [<Volume: 7f1ae761cd>]
 
参数详解：
filters(dict): 要在修剪列表上处理的筛选器
异常处理：
docker.errors.APIError          服务器返回错误
创建存储卷
volume = client.volumes.create(
        name = 'foobar', driver='local'
        )
>>> <Volume: foobar>
 
参数详解；
name(str): 卷的名称。如果未指定，引擎将生成一个名称
driver(str): 用于创建卷的驱动程序的名称
driver_opts(dict): 作为键值字典的驱动程序选项
labels(dict): 要在卷上设置的标签
异常处理：
docker.errors.APIError          服务器返回错误
挂载存储卷
方法一：先创建存储卷卷，创建完以后再挂载
1. 利用创建存储卷方式创建一个名为test_volume的存储卷
volume = client.volumes.create(name = 'test_volume', driver='local')
 
2. 创建容器的时候利用mounts参数将存储卷挂载上去
from docker.types import Mount
 
volume1 = Mount('/mnt/test', 'test_volume')
container = containers.run('300e315adb2f', mounts=[volume1])
>>> <Container: 2c381acc19>
 
Mount参数详解：
第一个参数为挂载目标路径
第二个参数为存储卷名称
read_only(bool): 是否为容器内只读权限
type(str): 挂载类型bind/volume/tmpfs/npipe 默认值：vloume
labels(dict): 卷的用户定义名称和标签。仅对“volume”类型有效
tmpfs_size(int or string): tmpfs装载的大小（以字节为单位）
tmpfs_mode(int): tmpfs挂载的权限模式
方法二：创建容器时使用volumes参数将存储卷挂载
volume = {'/home/user1/': {'bind': '/mnt/vol2', 'mode': 'rw'}, '/var/www': {'bind': '/mnt/vol1', 'mode': 'ro'}}
container = containers.run('300e315adb2f', volumes = volume)
>>> <Container: 1b341cde14>
 
方法三：创建容器时继承上一个容器
container = containers.run('300e315adb2f', volumes_from=['2c381acc1977'],name='test')
>>> <Container: .......>
 
获取指定存储卷
client.volumes.get('volume_name')
 
参数详解：
volume_name(str): 卷的名称（id）
异常处理：
docker.errors.NotFound          如果卷不存在
docker.errors.APIError          服务器返回错误
清空未使用卷
client.volumes.prune()
>>> {'VolumesDeleted': ['foobar', '7f1ae761cde1fd3d6cad4354533ce89f66964074823e2268d391ee1b3e296285'], 'SpaceReclaimed': 0}
 
参数详解：
filters(dict): 要在修剪列表上处理的筛选器
获取存储卷ID/short_ID
client.volumes.get().id
>>> test_volume
client.volumes.get().short_id
>>> test_volum
 
获取存储卷名称/属性
client.volumes.get().name
>>> test_volume
client.volumes.get().attrs
>>> {'CreatedAt': '2020-12-31T11:50:43+08:00', 'Driver': 'local', 'Labels': {}, 'Mountpoint': '/var/lib/docker/volumes/test_volume/_data', 'Name': 'test_volume', 'Options': {}, 'Scope': 'local'}
 
重新加载存储卷
client.volumes.get().reload()
>>> None
 
删除存储卷
client.volumes.get().remove()
>>> None
 
参数详解：
force(bool): 强制删除卷驱动程序插件
异常处理：
docker.errors.APIError          服务器返回错误
管理docker集群的对象（swarm）
初始化swarm
client.swarm.init(advertise_addr=None, listen_addr='0.0.0.0:2377',force_new_cluster=False, default_addr_pool=None,subnet_size=None, data_path_addr=None, **kwargs)
>>> 8y59fs18dhdr7o6v1dvm1it6q    # 管理节点的ID
 
参数详解：这两个地址不太理解，advertise_addr做为swarm的守护进程地址，多个网卡IP的时候必须要指定，listen_addr为实际程序间通讯的地址，注意，如果两个都指定IP且不一致的话以listen_addr为准，如果listen_addr不是0.0.0.0的话，表示只有本机能加入集群，其他机器加入会报TLS错误
advertise_addr(str): 向其他节点发布的外部可到达地址
listen_addr(str): 用于管理程序间通信的侦听地址
force_new_cluster(bool): 强制初始化集群
default_addr_pool(list of str): 默认地址池指定全局作用域网络的默认子网池
subnet_size(int): 指定从默认子网池创建的网络的子网大小
data_path_addr(string): 用于数据路径通信的地址或接口
task_history_retention_limit(int): 最大历史存储任务数
snapshot_interval(int): 快照之间的日志条目数
keep_old_snapshots(int): 要保留在当前快照之外的快照数
log_entries_for_slow_followers(int): 要在同步日志后保持慢速关注者数量?
heartbeat_tick(int): 心跳间隔（以秒为单位）
election_tick(int): 在没有master的情况下触发新选举间隔（以秒为单位）
dispatcher_heartbeat_period(int): 代理向调度程序发送心跳信号的延迟
node_cert_expiry(int): 证书自动过期节点
name(string): 名称
labels(dict): 用户定义的键/值元数据
signing_ca_cert(str): 所有swarm节点TLS证书所需的签名CA证书，PEM格式
signing_ca_key(str): 所有swarm节点TLS证书所需的签名CA密钥，PEM格式
autolock_managers(boolean): 如果设置就生成一个密钥并使用它锁定存储在管理                                  器上的数据
异常处理：
docker.errors.APIError          服务器返回错误
管理docker集群节点的对象（nodes）
管理docker插件的对象（plugins）
```
