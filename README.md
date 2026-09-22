# q-brain

基于 Obsidian 的个人知识大脑。全部信息来源统一规划、分批接入，第一阶段不开发本体。

## 项目文档

- [架构初稿](docs/research/obsidian-personal-brain-v0.1.md)
- [调研资料](docs/research/README.md)
- [工程技能配置](docs/agents/)

## 参考源码

`references/` 中的五个项目仅用于借鉴开发，不作为运行依赖。通过 Git 子模块固定调研版本。

完整克隆本项目与参考源码：

```sh
git clone --recurse-submodules https://github.com/benjamin-qhy/q-brain.git
```

已有克隆补齐参考源码：

```sh
git submodule update --init --recursive
```
