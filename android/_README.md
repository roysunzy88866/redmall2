# android/

车机端 Android APP(Kotlin + Jetpack Compose + Retrofit)。

**任何新文件路径必先核对** [docs/architecture.md](../docs/architecture.md) §车机端 `android/app/src/main/java/space/hearagain/ridemall/`(模块清单表)。
未在 architecture.md 声明的目录**不允许新增** —— 先去改 architecture.md,再 commit。

Gradle 工程结构在 C7 切片建立(`./gradlew init` 自动产生大部分目录)。

`pre-commit` 的 `path-must-be-declared` hook 会机械阻止未声明路径入库。
