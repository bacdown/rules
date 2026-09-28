# rules

面向 Mihomo、Shadowrocket 和 sing-box 的分流配置与规则集。包含 DNS、节点选择、故障转移和常用服务分流预设，也提供订阅转换脚本。

> 配置需要按自己的订阅、网络环境和客户端版本调整。示例订阅地址、节点、策略组名称和密钥不是可直接使用的真实值。使用前请检查配置注释，并自行验证 DNS 和路由行为。

## Wiki

平台配置说明和使用指南见 [rules GitHub Wiki](https://github.com/bacdown/rules/wiki)。

## 目录结构

```text
.
├── mihomo/
│   ├── config.yaml          # 单订阅 Mihomo 配置
│   └── mihomo2URL.yaml      # 双订阅 Mihomo 配置
├── Shadowrocket/
│   ├── config.ini           # Shadowrocket 配置
│   └── ChinaMax_Domain.list # 中国大陆域名规则
└── sing-box/
    ├── config_phone.json    # 手机端 sing-box 模板
    ├── config_openwrt.json  # OpenWrt sing-box 模板
    ├── momo.json            # Momo 客户端模板
    ├── sub2singbox.py       # 订阅转换脚本
    └── rule/
        ├── WeChat.json      # WeChat 规则源文件
        └── WeChat.srs       # sing-box 二进制规则集
```

配置按客户端分别存放，路径保持稳定，因为模板会引用仓库中的规则文件。各平台的详细说明见 Wiki：

- [Mihomo 配置](https://github.com/bacdown/rules/wiki/Mihomo)
- [Shadowrocket 配置](https://github.com/bacdown/rules/wiki/Shadowrocket)
- [sing-box 配置与订阅转换](https://github.com/bacdown/rules/wiki/sing-box)
- [规则集说明](https://github.com/bacdown/rules/wiki/Rule-Sets)

## 快速开始

### Mihomo

选择 `mihomo/config.yaml`（单订阅）或 `mihomo/mihomo2URL.yaml`（双订阅），将配置中的占位订阅地址替换为自己的订阅链接，再导入 Mihomo 客户端。检查 TUN、DNS、端口和外部控制器设置后再启用配置。

### Shadowrocket

导入 `Shadowrocket/config.ini`，在 `[Proxy]` 中填写自己的节点或订阅，并按实际节点名称检查 `[Proxy Group]` 的筛选条件。
或者保持 `[Proxy]` 默认就行，默认选择的是客户端手动选择或者启用回退后系统自动选择的节点，只影响DNS查询节点。

### sing-box 订阅转换

需要 Python 3.8+ 和 PyYAML：

```sh
python3 -m pip install PyYAML
python3 sing-box/sub2singbox.py --help
```

转换一个本地 Clash YAML 文件：

```sh
python3 sing-box/sub2singbox.py ./mihomo.yaml \
  -c sing-box/config_phone.json \
  -o ./sing-box-phone.json
```

也可提供多个本地文件、远程订阅链接或 URI 文本文件；使用 `-c/--config` 指定 JSON 模板。省略 `-o/--output` 时，结果写入第一个本地输入文件所在目录；若所有输入都是 URL，则写入当前目录。手机和 OpenWrt 模板分别使用 `sing-box-phone.json` 和 `sing-box-openwrt.json` 默认文件名。

支持范围及手机、OpenWrt 模板差异见 [sing-box Wiki 页面](https://github.com/bacdown/rules/wiki/sing-box)。

## 使用前检查

- **订阅**：替换配置中的占位链接；多订阅 Mihomo 配置还需同步检查 `SUB1`、`SUB2` 等订阅锚点和名称。
- **节点组**：Shadowrocket 的筛选规则依赖节点名称；订阅节点命名不同可能需要调整正则或策略组。
- **DNS 与分流**：DoH、Fake-IP、DNS 劫持、IPv6 和分流效果取决于客户端版本、系统网络及规则集状态，不保证在所有环境下表现相同。
- **规则集**：配置使用远程规则集；远程地址不可用或上游格式变化时，规则可能无法更新。
- **网络暴露**：OpenWrt 模板的 Mixed 代理入口监听 `0.0.0.0:7890`，如将其暴露到局域网，请用防火墙限制为可信 LAN 设备，勿开放到 WAN。
- **验证**：导入/启动前使用目标客户端检查配置；遇到国内服务、Google Play、Apple 推送或局域网访问异常时，先检查对应 DNS、Fake-IP 过滤和路由规则。

这些配置不构成绝对的防泄漏或安全保证。请遵守当地法律法规以及相关服务条款，并自行评估规则和远程依赖。

## 致谢

部分配置结构和规则集参考以下项目：

- [qichiyuhub/rule](https://github.com/qichiyuhub/rule)
- [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat)
- [blackmatrix7/ios_rule_script](https://github.com/blackmatrix7/ios_rule_script)
- [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat)

上游项目的许可证和使用条款以各自仓库为准。
