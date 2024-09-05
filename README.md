# MPRPC框架

用C++实现的基于Linux下的高性能RPC分布式框架。旨在解决分布式系统中服务之间的频繁调用问题。

## 功能

- 实现Application、Provider、Channel三个核心模块，即框架初始化、服务注册与发布、服务调用；
- 使用protobuf进行数据序列化和反序列化，以字节流进行数据传输，并设计包头+包体，一定程度解决粘包问题；
- 使用muduo库进行网络通信，基于Reactor网络模型，实现epoll+线程池的高性能网络通信；
- 使用ZooKeeper作为服务注册中心，服务调用方通过ZooKeeper发现服务提供方的主机和端口；
- 封装标准库queue构建lockQueue模版类，由多个线程写日志，单个线程读，实现异步日志记录；

## 环境要求

- Linux
- C++11
- Muduo
- Protobuf
- Zookeeper

## 目录树

```
.
├── bin            				可执行文件(编译后生成)
│   ├── consumer				服务调用方
│   ├── provider				服务提供方
│   └── test.conf				静态配置文件
├── build          				项目编译文件
├── lib            				项目库文件
├── src            				源文件
│   ├── include					src中对应h文件			
│   ├── logger.cc				日志	
│   ├── mprpcapplication.cc		框架初始化
│   ├── mprpcchannel.cc			rpc调用
│   ├── mprpcconfig.cc			配置文件加载(test.conf)
│   ├── mprpccontroller.cc	
│   ├── rpcprovider.cc			服务注册和发布
│   ├── zookeeperutil.cc		封装的zk客户端类	
│   ├── rpcheader.pb.cc
│   ├── rpcheader.proto			rpc调用序列信息格式
│   └── CMakeLists.txt
├── test           				测试代码(测试protobuf是否能正常使用)
├── example        				框架代码使用范例(业务)
│   ├── callee	
│   ├── caller	
│   ├── *.proto	
│   ├── *.pb.cc	
│   ├── *.pb.h
│   └── CMakeLists.txt
├── CMakeLists.txt 顶层的cmake文件
├── autobuild.sh   编译脚本
└── readme.md
```

## 项目启动

### 前置准备：

1.muduo网络库安装配置:

```bash
--	muduo库是基于boost开发，因此先安装boost库
--	安装具体可以参考:https://blog.csdn.net/QIANGWEIYUAN/article/details/88792874
--	这里假定我们已经下载了boost_1_69_0.tar.gz
tar -zxvf boost_1_69_0.tar.gz
cd boost_1_69_0/
--	运行工程编译构建程序
./bootstrap.sh
--	运行b2程序(如果无g++先装g++，建议版本4.6+)
./b2

--	进入root
sudo su
./b2 install
--	可以自己调一下boost的库写个简单的cpp代码编译运行来验证一下
--	muduo库安装
--	安装具体可以参考:https://blog.csdn.net/QIANGWEIYUAN/article/details/89023980?spm=1001.2014.3001.5502
--	这里假定我们已经下载了muduo-master.zip
unzip muduo-master.zip
cd muduo-master
vim CMakeLists.txt
--	注释掉option(MUDUO_BUILD_EXAMPLES "Build Muduo examples" ON)这一行
./build.sh
./build.sh install

--	避免每次g++都要指定muduo的头文件和库文件路径，直接做拷贝
cd build/release-install-cpp11/include/
mv muduo/ /usr/include/
cd ../lib/
mv * /usr/local/lib/
--	写代码测试一下
```

2.Protobuf安装配置:

```bash
git clone git@github.com:protocolbuffers/protobuf.git
--	或者直接下载对应解压包
unzip protobuf-master.zip
cd protobuf-master
--	安装对应所需前置工具(如果未安装)
sudo apt-get install autoconf automake libtool curl make g++ unzip
--	自动生成configure配置文件
./autogen.sh
--	配置环境
./configure
--	编译
make
--	安装
sudo make install
--	刷新动态库
sudo ldconfig
--	测试
protoc	...	--cpp_out=./
```

3.安装zk的原生开发api(C/CPP接口)

```bash
--	我这里使用的是zookeeper-3.4.10版本，需要下载对应压缩包
tar -zxvf zookeeper-3.4.10.tar.gz
--	去到src下的c文件夹
cd zookeeper-3.4.10/src/c
sudo ./configure
sudo make
sudo make install
```

