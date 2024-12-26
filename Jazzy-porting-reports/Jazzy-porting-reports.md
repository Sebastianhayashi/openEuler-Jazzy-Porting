# ROS Jazzy 移植到 openEuler 工作汇总

## 目标

尝试将 ROS Jazzy 完整包集合移植到 openEuler 24.03 上，希望实现从 ROS 包信息获取的自动化构建 RPM 包的完整构建流程，并且确保这些批量生成的软件包压缩包以及对应的 spec 能够正确构建，在 eulermaker 上正确进行构建。

## 完成工作

### 分析 ROS Jazzy 包集合

通过一个名为 `fetch_ros_packages.py.py` 的脚本实现了如下功能：

- 从 ROS 官方的 sitemap.xml 文件中获取了所有 ROS 包页面链接
- 从 ROS 包页面链接提取源码仓库地址
  - 且如果有 `jazzy` 分支，则自动 `clone` 该分支，如果没有则 `clone` 主分支
- 自动化 `git clone` 这些仓库到本地指定的目录，且支持多线程操作

> 特殊说明：一般来说这些源码包都是 GitHub 链接，但是会有少数是 gitlab 等，脚本不具备判断的能力，所以会出错，但是是少数，可以手动克隆到本地。

未来工作：该脚本目前没有大问题，考虑进一步开发分析每个包的依赖链，明确核心包与二级、三级依赖包之间的关系的功能，能够通过使用一个字典来存储依赖关系：

```
dependencies = {
    "mola_msgs": ["ament_cmake", "rosidl_default_generators"],
    "ament_cmake": [],
    "rosidl_default_generators": ["rosidl_default_runtime"]
}
```

使用拓扑排序（Topological Sorting）解析依赖图，确保能够按正确的顺序构建包。

更详细的脚本说明，click here

### RPM 包构建流程

这个过程中涉及两个脚本：`process_ros_packages.sh`, `batch_process_packages.sh`

`process_ros_packages.sh`：这个脚本能够使用 `bloom-generate` 工具生成 `.spec` 文件。能够将源码根据生成的 `.spec` 对于  `.spec` 文件以及源码包进行重命名，并且打包成 `tar.gz` 格式压缩包。再将  `.spec` 文件以及源码压缩包放到 SPECS 和 SOURCES 目录中。

`batch_process_packages.sh`：当文件夹中存在十分大量（如几十、几百个软件包）时，这个脚本能够在指定目录（或当前目录）中，搜索是否包含 `CMakeLists.txt` 和 `package.xml`，判断当前这个文件夹是否是一个标准的软件包，再 使用 `parallel` 工具实现多线程并行处理多个 ROS 包。处理方法为：在每个 ROS 包目录的父目录下，调用 `~/process_ros_packages.sh` 脚本生成 RPM 包。
同时，这个脚本支持检测在构建过程中出现的错误，并且记录在相应的日志文件中：

- `missing_rosdeps.log`：这个日志文件会记录无法解析的依赖项。
- `skipped_packages.log`：这个日志会记录因 `bloom-generate` 失败而跳过的包。

未来工作：目前这两个脚本能够正常工作，不考虑对其进行进一步的改动。

## 已解决的主要问题

目前通过上面的内容能够成功实现了从 ROS 包索引中自动提取包名、依赖关系和仓库地址，并且自动切换分支，再克隆到本地，使用 `bloom-generate` 工具自动化生成 RPM 规范文件。

