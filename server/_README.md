# server/

Flask 后端 + 后台。

**任何新文件路径必先核对** [docs/architecture.md](../docs/architecture.md) §后端 `server/`(模块清单表)。
未在 architecture.md 声明的目录**不允许新增** —— 先去改 architecture.md,再 commit。

`pre-commit` 的 `path-must-be-declared` hook 会机械阻止未声明路径入库。
