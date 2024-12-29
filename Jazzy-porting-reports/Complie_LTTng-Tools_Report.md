# LTTng-Tools 报告


## 背景

目前大部分包依赖未闭环的原因是 rmw-fastrtps-shared-cpp 这个软件包无法正常构建，提示：

```
Error:
236
2024-12-29 14:26:08 Problem: conflicting requests
237
2024-12-29 14:26:08 - nothing provides lttng-tools needed by ros-jazzy-tracetools-8.5.0-0.oe2403.x86_64 from 0
238
2024-12-29 14:26:08 (try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

所以现在需要解决 lttng-tools 的问题，但是由于 openEuler 上游没有直接提供 lttng-tools 这个软件包，所以需要手动构建这个依赖给 ros-jazzy-tracetools，便有了本报告。

本报告主要是记录如何编译 LTTng-Tools，以及如何解决上述问题。

## 编译和安装 LTTng-UST

由于 openEuler 上游缺少 lttng-ust，手动编译并安装较新版本的 lttng-ust。

### babeltrace2

liburcu 作为 babeltrace2 的前置依赖，需要先编译 babeltrace2。

依赖：

- elfutils-libelf-devel
- libdwarf-devel
- libdwarf
- libdwarf-tools
- elfutils
- elfutils-libelf
- elfutils-libelf-devel
- elfutils-devel

```
git clone https://git.efficios.com/babeltrace.git
git checkout stable-2.0
./bootstrap
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
```
 
### liburcu

由于 openEuler 上游缺少 liburcu，需要先手动编译 liburcu。

依赖：

- numactl-devel
- numactl-libs

```
git clone https://github.com/urcu/userspace-rcu.git
cd userspace-rcu
./bootstrap
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install

export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
export PATH=/usr/local/bin:$PATH
```

### LTTng-UST

由于 openEuler 软件源中的 lttng-ust 版本较低（2.13.7），需手动编译安装较新版本。

依赖：
- liburcu >= 0.12
- libnuma
- asciidoc
- xmlto
- babeltrace2
- elfutils-libelf-devel
- libdwarf-devel
- libdwarf
- libdwarf-tools
- elfutils
- elfutils-libelf
- elfutils-libelf-devel
- elfutils-devel

```
git clone https://github.com/lttng/lttng-ust.git
cd lttng-ust
git checkout master

./bootstrap
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
sudo ldconfig
cd ..

pkg-config --modversion lttng-ust

# 应输出 2.14.x 或更高版本号，表示安装成功。
```

## 编译和安装 LTTng-Tools

依赖

确保已安装以下依赖：
- libtool
- ibtool-ltdl-devel
- bison
- flex
- libxml2
- libxml2-devel
- asciidoc
- xmlto
- babeltrace2
- liburcu
- libnuma

## 克隆并编译 LTTng-Tools

```
git clone https://github.com/lttng/lttng-tools.git
cd lttng-tools
mkdir build && cd build
cmake ..

./bootstrap
./configure --prefix=/usr/local
make -j$(nproc)
sudo make install
sudo ldconfig
```

完成所有步骤后，验证安装是否成功。

```
lttng --version
```

应输出 LTTng-Tools 的版本信息，表示安装成功。

## 其他说明

由于上述软件包并不是 ros 软件包，所以需要将上面的过程写进 ros-jazzy-tracetools.spec，目前还在跟进。