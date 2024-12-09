# rosdep-oe 报告

源仓库：https://github.com/Grvzard/rosdep-oe.git

## 编译

> 说明：该编译行为发生在虚拟环境中。

```
git clone https://github.com/Grvzard/rosdep-oe.git
cd rosdep-oe
pip install .
```

能够成功编译，并且使用 `rodep update`

![6](./img/6.png)

而目前 `pip` 直接下载的 `bloom` 是 0.25 的版本，所以不能够直接使用 0.24 的版本的 `rosdep`，所以考虑也从源码编译 `bloom`，使用相应的版本

这个问题在 `bloom` 工具的报告中会解决。

![7](./img/7.png)

且安装 `rosdep-oe` 的时候会下载 `rospkg-oe` 这个移植版作为依赖，那么对应在调用依赖的时候需要注意。

## 结论

`rosdep-oe` 能够预期的在 `openeuler 24.03 x86` 上运行。