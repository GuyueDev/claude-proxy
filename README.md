# Claude Proxy for Loon

让 Claude 的匹配请求使用你选择的代理。Mac 使用进程与域名规则，iOS / iPadOS / tvOS 使用域名规则。**无需预建默认策略组，导入后在插件设置中自行选择代理。**

模块不包含代理节点、订阅、账号凭据、脚本或 HTTPS 解密配置。

## URL 导入

将下面的地址复制到 Loon 的插件管理中添加：

```text
https://raw.githubusercontent.com/GuyueHub/claude-proxy/main/Claude-Fixed-Proxy.lpx
```

[查看模块原文](https://raw.githubusercontent.com/GuyueHub/claude-proxy/main/Claude-Fixed-Proxy.lpx)

> 此地址在文件推送到 GitHub 的 `main` 分支且仓库可公开访问后生效。不要使用 GitHub 的 `blob` 网页地址代替 Raw 地址。私有仓库不能通过这个公开 URL 直接导入，不要将 GitHub Token 写入导入链接。

1. 备份当前 Loon 配置。
2. 在插件管理中添加上面的 URL。
3. 打开该插件的设置，为 `PROXY` 选择你要使用的节点或策略组，再启用插件。不同版本的控件名称可能不同。
4. 如果希望出口固定，选择固定节点；如果使用策略组，确保组内不含 `DIRECT` 或会切换到其他出口的策略。
5. 保持 Loon 隧道开启并使用规则模式，重新启动 Claude，在请求记录中检查最终策略和节点。

**规则没有写死用户的节点名称。** `PROXY` 是插件规则的策略占位，由插件设置绑定。不要依赖尚未选择策略时的默认处理。

## 平台支持

| 平台 | 匹配方式 | 注意事项 |
| --- | --- | --- |
| Loon for Mac | Claude 安装目录、主进程、Helper 名称及域名 | 进程无法识别时依赖域名规则；不自动继承到所有子进程 |
| iOS / iPadOS | 域名 | 进程规则被忽略；未知第三方域名和 IP 直连可能遗漏 |
| tvOS | 域名 | 仅提供规则兼容，不意味着有 Claude 原生客户端 |

Mac 进程规则按 Loon for Mac 0.5.0（Build 1011）文档编写。移动端请使用支持当前插件语法的版本。已做静态结构检查，尚未完成跨设备导入与实际网络验收。

## 已有规则的冲突

Loon 的本地规则优先于插件规则。已有 Claude 进程或域名规则可能覆盖插件选择；将这些规则统一到相同出口，或在备份后删除重复的路由规则。仅调整插件顺序不能覆盖本地规则。

域名规则作用于所有访问对应域名的应用，并非仅 Claude App。外部浏览器登录、本地 MCP、任意 `node` / `python` 子进程及 Cowork 虚拟机流量不保证被进程规则覆盖。

## DNS、WebRTC 和断线行为

**仅导入本模块不会完成全部防泄漏配置。** 模块负责分流；下列设置需要单独调整主配置，而且会影响整机：

```ini
[General]
ip-mode = v4-only
hijack-dns = *:53
disable-stun = true
udp-fallback-mode = REJECT
```

把对应键合并到现有 `[General]`，不要用片段覆盖整份配置。DNS 示例见 [主配置合并片段](docs/General-Merge.txt)，详细说明见 [隐私设置与验收](docs/privacy-and-verification.md)。

- 加密 DNS 需要单独配置，并关闭普通 DNS 回落。DoH 加密不代表查询已走代理，必须验证实际出口。
- `disable-stun` 减少 STUN UDP 泄漏，不等于完全禁用 WebRTC。
- `udp-fallback-mode = REJECT` 防止不支持 UDP 时回落直连。
- Loon 停止、隧道外流量及绕过项不受模块保护；本模块不是系统级断网开关。
- 时区、语言、设备指纹和本地文件权限不属于网络分流的控制范围。

移动端如需在使用 Claude 期间覆盖未知目标，可以将 Loon 全局代理指定到同一节点，但会影响其他应用，且仍需检查 DNS 与绕过项。

## 更新与回滚

更新时在 Loon 中刷新该 URL，检查规则变动及策略绑定。需要固定版本时，可将 URL 中的 `main` 替换成你确认过的完整提交 SHA。

停用插件即可撤销插件路由；主配置和 DNS 设置需单独恢复备份。详见验收文档中的回滚说明。

## 仓库文件

| 文件 | 用途 |
| --- | --- |
| [Claude-Fixed-Proxy.lpx](Claude-Fixed-Proxy.lpx) | 可通过 URL 导入的插件 |
| [docs/General-Merge.txt](docs/General-Merge.txt) | 主配置合并片段 |
| [docs/privacy-and-verification.md](docs/privacy-and-verification.md) | 能力边界、验收与回滚 |

## 官方文档

- [Loon 插件与策略绑定](https://nsloon.app/docs/Plugin/)
- [Mac 进程规则](https://nsloon.app/en/docs/Rule/process_rule/)
- [通用配置](https://nsloon.app/docs/General/)
- [DNS](https://nsloon.app/docs/DNS/)
- [规则优先级](https://nsloon.app/docs/Rule/)
