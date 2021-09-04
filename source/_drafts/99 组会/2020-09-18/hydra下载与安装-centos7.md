### 下载与安装

下载

```bash
wget https://github.com/vanhauser-thc/thc-hydra/archive/master.zip
```

安装依赖

```bash
yum -y install gcc libssh-devel openssl-devel
```

解压

```bash
unzip master.zip
```

编译安装（su）

```bash
cd thc-hydra-master/
./configure
make && make install
```

