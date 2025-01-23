# Detect openEuler

> 如果你还没有读 README，推荐先去看看 README。

## 背景

本文旨在让 rospkg 工具能够正确的识别到 openEuler 及其对应的版本号。
本文中示例对应的系统及其版本：

```
(toolchain) ➜  rcl_logging_interface git:(jazzy) ✗ cd
(toolchain) ➜  ~ cat /etc/openEuler-release
openEuler release 24.03 (LTS)
(toolchain) ➜  ~ cat /etc/os-release
NAME="openEuler"
VERSION="24.03 (LTS)"
ID="openEuler"
VERSION_ID="24.03"
PRETTY_NAME="openEuler 24.03 (LTS)"
ANSI_COLOR="0;31"
```
本文的目标是实现以下效果，即 bloom 工具能够配合 rospkg 识别出 openEuler：

```
(toolchain) ➜  rcl_logging_interface git:(jazzy) ✗ bloom-generate rosrpm --os-name openEuler --os-version 24.03 --ros-distro jazzy
ROS Distro index file associate with commit '0d9a6f9eea5073fc27bdaf0f5e242b9d0c1b8d4a'
New ROS Distro index url: 'https://raw.githubusercontent.com/ros/rosdistro/0d9a6f9eea5073fc27bdaf0f5e242b9d0c1b8d4a/index-v4.yaml'
==> Generating RPMs for openEuler:24.03 for package(s) ['rcl_logging_interface']
No homepage set
GeneratorError: Error running generator: Could not determine the installer for 'openEuler'
```

## 源码改动

负责识别系统的工具是 rospkg，所以我们移植整个工具链的第一步就是移植 rospkg，首先我们要能够保证 bloom 能够正确识别到我们的系统。

rospkg 中负责处理系统检测的文件是 os_detect.py，我们也只需要对这个文件进行修改，该文件位于：

```
rospkg/src/rospkg/os_detect.py
```

具体的 patch 可以查看[这里](https://github.com/Sebastianhayashi/rospkg/commit/0ed4a19f12a065f8fea21cc540abf4c9b4c6f25b)

## 改动说明

第一处改动：

```
class OpenEuler(OsDetector):
    """
    Detect OpenEuler OS.
    """
    def __init__(self, release_file="/etc/openEuler-release"):
        self._release_info = read_os_release()
        self._release_file = release_file
    def is_os(self):
        return os.path.exists(self._release_file)
    def get_version(self):
        if self.is_os():
            return self._release_info["VERSION_ID"]
        raise OsNotDetected("called in incorrect OS")
    def get_codename(self):
        if self.is_os():
            return ""
        raise OsNotDetected('called in incorrect OS')
```

is_os() 检查了 /etc/openEuler-release 是否存在，这个文件是 openEuler 中特有的。该逻辑是当存在该文件的话就可以直接判定这个系统是 openEuler。
接着使用 get_version() 调用 read_os_release() 并返回 VERSION_ID，这样就能正确的输出 openEuler 以及对应的版本号了。

### 其他思路

对于 rospkg 而言，有一个通用类：

```
OsDetect.register_default("fedora", FdoDetect("fedora"))
```
这个通用类是使用 FdoDetect 读取 /etc/os-release，如果说读取出 ID="fedora"，则认为系统是 fedora。

而 openEuler 中采取了与 OpenSuse 以及 Gentoo 蕾丝的定义独立类检查发行版的特定文件，在移植的时候没有特别考量可以考虑使用 FdoDetect 读取。

第二处改动：

新增 OS_OPENEULER = 'openEuler' 常量，让 rospkg 把 OS 名定义为 openEuler

可以按照类似的写法去定义自己的发行版。

第三处改动：

```
OsDetect.register_default(OS_OPENEULER, OpenEuler())
```

这句语法是能够让 rospkg 在 detect_os() 的时候能会先尝试将 openEuler 匹配到 OpenEuler()，让开始进行处理。
当 OpenEuler().is_os()= True 的时候 openEuler 就会成为最先匹配成功的 OS，这样就能保证 rospkg 优先匹配自己的发型版，避免与其他的系统冲突。

## 测试

使用 `pip install -e .` 安装已经移植完的代码。
进入一个 ros 工具包使用如下指令（请根据实际情况修改）测试是否能够正确输出：
```
bloom-generate rosrpm --os-name openEuler --os-version 24.03 --ros-distro jazzy
```
预期行为：

```
(toolchain) ➜  rcl_logging_interface git:(jazzy) ✗ bloom-generate rosrpm --os-name openEuler --os-version 24.03 --ros-distro jazzy
ROS Distro index file associate with commit '0d9a6f9eea5073fc27bdaf0f5e242b9d0c1b8d4a'
New ROS Distro index url: 'https://raw.githubusercontent.com/ros/rosdistro/0d9a6f9eea5073fc27bdaf0f5e242b9d0c1b8d4a/index-v4.yaml'
==> Generating RPMs for openEuler:24.03 for package(s) ['rcl_logging_interface']
No homepage set
GeneratorError: Error running generator: Could not determine the installer for 'openEuler'
```

## 其他说明

本文中使用的 patch 出处为：https://github.com/ros-infrastructure/rospkg/commit/812e09840e3e264f87001f1557342b5265a3c403#diff-4ec3d8ff31d9d834ce01e10784e00d1ba38736582346ef8fa1610bc3a737d1cfL265-R721



