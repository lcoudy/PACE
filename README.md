# PACE · Project-Aware Context Engine

PACE 面向同时负责多个项目的产品和运营团队。成员问一条业务规则时，系统要回答它适用于哪个项目、从哪一天生效、依据在哪；资料互相矛盾时，要能找到负责人核实，并在知识更新后验证答案。

## 项目状态

2026 年 11 月 27 日建立独立公开仓库。从 [lcoudy/ragent 的固定提交](https://github.com/lcoudy/ragent/commit/0c02af99212ddf8c1063eb1330bc7afb10e919a4) 开始，保留源码提交历史和 [Apache-2.0 许可证](LICENSE)。当前代码是 Ragent 基线。PACE 的业务规则、纠错发布和端到端测评处于设计阶段。

Ragent 提供 Java 问答服务、管理界面、知识检索和运行记录。首次做基线实验时应同时记录源码提交和使用的镜像版本，以便复现。

## 开发入口

- [首版设计、开源参考和验收场景](docs/pace-design.md)
- [继承的 Ragent 运行与架构说明](README.ragent.md)
- [现有 Agent 知识问答路径](rag/src/main/java/com/nageoffer/ai/ragent/rag/service/KnowledgeSearchFacade.java)
- [现有检索评测接口](rag/src/main/java/com/nageoffer/ai/ragent/rag/eval/EvalController.java)

单独克隆 PACE 的开发仓库：

```shell
git clone https://github.com/lcoudy/PACE.git
```

第一项验证是对两套模拟项目资料做原版问答，保存实际回答、证据及配置，再开始修改检索和反馈。
