# bloom-generate 可以检测到 openEuler

# 背景

在 [bloom-generate](https://github.com/Sebastianhayashi/Jazzy-euler/blob/main/Bloom_tool_research.md) 中能够看到，目前 `bloom-generate` 工具无法识别 openEuler，所以需要对其进行适配。

# rosdep

根据 rosdep-oe 中的内容已经确定了 rosdep-oe 可以使用，但是在实际安装 bloom 的时候会因为依赖关系直接下载最新的 rosdep 导致移植版（rosdep-oe）被覆盖过去，于是设计了以下思路运行了 bloom-generate。

## 编译

bloom、rosdep 以及 rosdistro 从源码编译，因为需要修改其依赖关系，修改好的仓库如下：

- [bloom](https://github.com/Sebastianhayashi/bloom-oe)
- [rosdep](https://github.com/Grvzard/rosdep-oe)
- [rosdistro](https://github.com/Sebastianhayashi/rosdistro-oe.git)

编译过程：

```jsx
// build rosdep-oe from source
git clone https://github.com/Grvzard/rosdep-oe
cd rosdep-oe
pip install .

// build rosdistro from source
git clone [https://github.com/Sebastianhayashi/rosdistro-oe.git](https://github.com/Sebastianhayashi/rosdistro-oe.git)
cd rosdistro-oe
pip install .

// build bloom from source
git clone [https://github.com/Sebastianhayashi/bloom-oe](https://github.com/Sebastianhayashi/bloom-oe)
cd bloom-pe
pip -r install requirements.txt
pip install --no-deps .
```

# 验证

使用指令 `bloom-generate rosrpm --os-name openeuler --os-version 23.03 --ros-distro humble --help` 验证是否能够检测到 openEuler 系统及其版本，如图没有报错，正确的输出了相关帮助信息。

![6](./img/6.png)