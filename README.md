# Jazzy-euler

该仓库旨在调研将 Jazzy 移植到 openEuler 上。

## 目前工具依赖关系

### 已移植

`bloom-generate` -> https://github.com/Sebastianhayashi/bloom-oe

- `rosdep` -> https://github.com/Grvzard/rosdep-oe.git
- `rosdistro` -> https://github.com/Sebastianhayashi/rosdistro-oe.git

## 目前进度

~~要运行 `bloom-generate` 的前提是能够让 `rosdep` 识别并且部署相关环境，所以需要先从 `rosdep` 开始进行适配。~~

已经成功让 `bloom-generate` 识别到 openEuler，下一步计划为尝试使用 bloom 工具在 openEuler 上打一个 Jazzy 的包。
