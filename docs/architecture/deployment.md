# 架构 · 部署拓扑

> [← 返回总览](../architecture.md)
> **修订记录**:v1 (2026-05-28) — 从 architecture.md 拆出
> **硬上限 150 行**。

## 拓扑

Mac mini → launchd 拉起 Flask(`127.0.0.1:18767`)→ Cloudflared `panqian-tunnel` ingress → `https://ridemall.hearagain.space`。

- Android `debug` → `http://10.0.2.2:18767/`(emulator 指向宿主)
- Android `release` → `https://ridemall.hearagain.space/`
- 演示完 `launchctl unload` 下线,公网立即 404

## 端口

`18767` = ridemall 默认端口(panqian 占 18765,aishare 占 18766,顺延)。
**可被 env override**,代码走 `config.py` 读取,不硬编码(Rule 14)。

## 详见

- [需求共识.md](../../需求共识.md) F6 + §1 部署
- [ADR-0001](../adr/0001-engineering-discipline.md)
- C11 切片填充 `deploy/` 下的 launchd plist + cloudflared ingress 片段 + install/update 脚本
