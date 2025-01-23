# Porting rosdep and rosdistro to openEuler

## 移植思路

### rosdep 层

rosdep 的移植不仅仅关系到 rosdep 本身，还有 rosdistro 的移植。
首先，对于 rosdep 而言要能够正确的识别出 openEuler 或者其他的非官方发行版，这部分仅需修改 redhat.py 即可。

主要是改动该路径下的文件：src/rosdep2/platforms/redhat.py

- 新增常量：OS_OPENEULER = 'openEuler'
- 新增函数：register_openeuler(context)
  - 为 openEuler 注册多个 install key
- 新增 register_openeuler(context)
  - 使 rosdep 在平台注册阶段能够识别 openEuler
- 新增 register_rhel_clone(context, 'openEuler')
  - 者表示 openEuler 不走 “rhel clone”alias，是作为一个独立的 OS 进行处理的

更详细的内容请查看[这里](https://github.com/Sebastianhayashi/rosdep/commit/fd4978b827ff389c55ead0ce03817fb5613b03c4)

### rosdistro 层

于 rosdistro 而言，还需要自己从官方的仓库中 fork 一份到自己的账户中对对应的 yaml 文件进行修改。官方的 rosdistro 仓库中可以找到不同ROS发行版的配置文件和相关信息，这些文件列出了所有已发布的ROS软件包及其依赖关系。
需要做改动的部分是找到要移植的版本（这里以 jazzy 示例）对应文件夹下的 base.yaml 以及 python.yaml 进行补充。

接着，将 rosdistro 下的 `jazzy/distribution.yaml` 添加对应的发行版及其版本，如：

```
%YAML 1.1
# ROS distribution file
# see REP 143: http://ros.org/reps/rep-0143.html
---
release_platforms:
  debian:
  - bookworm
  rhel:
  - '9'
  ubuntu:
  - noble
  openeuler:
  - '24.03'
  ...
```
新增：
```
  openeuler:
  - '24.03'
```

请根据自身的发行版情况进行修改。

这里有额外自己做了一个自动化脚本，流程如下：
1. 读取该文件夹下的官方 base.yaml 文件
2. 对每个 key 按顺序的查找已经定义的包名
3. 找到包名后开始一个个的 dnf list <pkg>
4. 如果存在的话就记录，如果没有就记录到 fail_list.txt

这样的流程就能实现自动化的更新 yaml 中的内容。

脚本请查看[这里](../Scripts/%20auto_generate_openeuler_yaml.py)

> 这里的 rosdistro 指的是 https://github.com/ros/rosdistro，而不是工具，
>

接着需要更改的是：rosdep/sources.list.d/20-default.list
将这个文件下的链接都修改成自己仓库中的链接。

> 说明：如果你的系统是定制发行版，那么你需要补充 base.yaml 等，以及将 20-default.list 修改成你仓库中的 yaml 链接。


## 其他说明

原始 patch：https://github.com/ros-infrastructure/rosdep/commit/139dde1bfa4cb58f6cfc6ede07f297c25c3de236#diff-0beed7b5444dff2fafbee569654182570dc1046fe8ce9f6b815014234e7e5ec7L37-R83
