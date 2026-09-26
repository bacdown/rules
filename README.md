# rules

个人维护的代理分流规则与客户端配置，面向 **Mihomo** 和 **Shadowrocket**。配置以按域名、IP 和服务分类进行分流为主，并针对 DNS、防泄漏、自动选节点做了较为完整的配置。

> ⚠️ 这些配置需要根据自己的订阅、网络环境和客户端版本进行调整。使用前请先阅读对应配置中的注释，并自行确认规则和 DNS 行为是否符合预期。

## 目录结构

```text
.
├── mihomo/
│   ├── config.yaml       # 单订阅 Mihomo 配置
│   └── mihomo2URL.yaml   # 双订阅 Mihomo 配置
└── Shadowrocket/
    ├── config.ini        # Shadowrocket 策略组配置
    └── ChinaMax_Domain.list # 中国大陆域名直连规则
```

## 配置说明

### Mihomo

- [`mihomo/config.yaml`](./mihomo/config.yaml)：适合单订阅使用，包含自动选择、故障转移和手动选择等策略组。
- [`mihomo/mihomo2URL.yaml`](./mihomo/mihomo2URL.yaml)：适合双订阅使用，可分别配置 `SUB1` 和 `SUB2`，按地区和策略类型进行管理。
- 使用 Mihomo 的远程规则集，包含域名规则、IP 规则以及 Fake-IP 过滤规则。
- 已预设 AI、Meta、YouTube、Google、GitHub、Telegram、Netflix、Microsoft、Apple、Steam 等服务的分流策略。
- 默认启用 TUN、Fake-IP、DoH 和域名嗅探；具体行为请以配置文件和客户端版本为准。

使用前至少需要修改订阅地址：

```yaml
proxy-providers:
  name:
    url: "订阅URL"
```

双订阅配置则需要分别修改 `SUB1` 和 `SUB2` 对应的 `url`，并同步检查配置中的订阅名称锚点。

### Shadowrocket

[`Shadowrocket/config.ini`](./Shadowrocket/config.ini) 包含：

- 按地区自动测速选择节点；
- AI、YouTube、Netflix、Telegram、PayPal、X、Meta、GitHub、Google、Apple 等服务分流；
- 中国大陆域名和 IP 直连；
- 通过 DoH、DNS 劫持和代理 DNS 设置降低 DNS 泄漏风险；
- 常用国内服务直连，以及广告和隐私规则的可选配置。

导入后，请在 `[Proxy]` 中填写或替换自己的节点，并根据实际节点名称调整 `[Proxy Group]` 中的筛选规则。

#### 节点组与策略配置

- **AUTO 节点组**：添加了分地区自动选择（HK、TW、JP、SG、KR、US），可根据实际节点情况调整测速间隔，避免网络波动节点失效。
- **FINAL 兜底规则**：默认使用 AUTO 策略。在软件界面选择的节点只影响 DNS 代理查询，建议启用自动回退并根据节点情况调整测速间隔（默认 600 秒）。

#### DNS 配置说明

配置中的 DNS 设置可根据实际需求调整：

**代理 DNS（用于代理域名）**
```
dns-server = https://dns.google/dns-query#proxy
```
- `#proxy` 可保持默认（软件界面手动选择节点），也可修改为具体节点组名称（如 `#AUTO`、`#JP自动选择`）或节点名称，这样对应节点组作为DNS的代理域名。
- `#proxy` 下软件界面选择的节点只对代理 DNS 查询生效

**直连 DNS（用于直连域名）**
```
direct-dns-server = https://doh.pub/dns-query,https://dns.alidns.com/dns-query,system
```
- 可根据需要修改 DNS 服务器地址
- `system` 为运营商 DNS，可删除以完全避免明文 DNS 泄漏
- 建议仅保留公共 DoH 服务（如阿里云 DNS、腾讯 DNS 等）

## 使用方式

### Mihomo

1. 下载并安装支持 Mihomo 内核的客户端，例如 OpenClash、Nikki 或其他兼容客户端。
2. 导入 `mihomo/config.yaml`，或根据订阅数量选择 `mihomo/mihomo2URL.yaml`。
3. 将配置中的占位订阅地址替换为自己的订阅 URL。
4. 检查端口、TUN、DNS 和外部控制器设置，确认没有与本机其他服务冲突。
5. 启用配置并观察日志，确认 DNS 和分流结果符合预期。

### Shadowrocket

1. 将 `Shadowrocket/config.ini` 导入 Shadowrocket。
2. 填入自己的代理节点或订阅。
3. 按需启用或关闭 AI、广告、隐私等规则集。
4. 开启连接后，通过 Shadowrocket 的请求记录检查规则命中情况。
5. 根据需要在软件界面调整节点选择，推荐启用自动回退机制以提高连接稳定性。
6. 可在配置中自定义代理 DNS 的查询节点和直连 DNS 服务器，根据实际网络环境优化 DNS 响应。

## 注意事项

- 配置中的 `订阅URL`、节点名称和策略组名称都是示例或占位内容，不能直接保证开箱即用。
- 配置依赖多个远程规则集和 GeoIP/GeoSite 数据；远程地址不可用时，相关规则可能无法更新。
- DNS、防泄漏和分流效果会受到客户端版本、网络环境、系统 DNS、IPv6 及上游规则集变化影响，不能视为绝对保证。
- 如果遇到 Google Play、Apple 推送、局域网设备或国内服务异常，请优先检查 DNS、Fake-IP 过滤列表和对应规则组。
- 请遵守当地法律法规以及所使用网络服务的条款，不要将配置用于未经授权的访问。

## 致谢

部分配置结构和规则集参考使用以下项目：

- [qichiyuhub/rule](https://github.com/qichiyuhub/rule)
- [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
- [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)

规则集及上游项目的许可证和使用条款以其各自仓库为准。
