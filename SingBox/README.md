# Clash / Base64 订阅转换为 sing-box 1.14

## 环境要求

- macOS、Linux 或 Windows
- Python 3.8+
- PyYAML
- sing-box 1.14+

检查 Python 版本并安装 PyYAML：

```sh
python3 --version
python3 -m pip install PyYAML
```

Windows 可使用 `python` 代替 `python3`。

## 使用方法

### 快速开始

以下示例假设 `sub2singbox.py`、`config_phone.json` 和 `mihomo.yaml` 都在 macOS 的 `~/Downloads`（下载）目录中。现在需要通过 `-c` 明确选择手机或 OpenWrt 模板。省略 `-o` 时，输出文件保存在第一个本地输入文件所在目录；输入全是订阅 URL 时，保存在运行命令的当前目录。显式指定 `-o` 时按指定路径保存。

**方式一：直接使用完整路径**，不需要先切换目录：

```sh
python3 ~/Downloads/sub2singbox.py ~/Downloads/mihomo.yaml \
  -c ~/Downloads/config_phone.json
```

**方式二：先进入下载目录**，再使用相对路径：

```sh
cd ~/Downloads
python3 sub2singbox.py mihomo.yaml -c config_phone.json
```

转换成功后，在下载目录中运行以下命令启动生成的配置：

```sh
cd ~/Downloads
sing-box run -c sing-box-phone.json
```

### 转换本地 Clash YAML 文件

```sh
python3 sub2singbox.py clash.yaml -c config_phone.json
```

转换出的节点会加入模板中的手动选择、自动选择和对应地区策略组。模板里的 DNS、路由、入站及规则集会保留。

### 合并多个 YAML 文件

可以在同一条命令中传入两个或更多 Clash YAML 文件；脚本会合并所有文件中的节点，再统一写入一个 sing-box 配置：

```sh
python3 sub2singbox.py japan.yaml singapore.yaml usa.yaml \
  -c config_phone.json
```

输入文件可使用相对路径或完整路径。重复的节点名称会自动加后缀以保证 tag 唯一。多个本地文件混用时，默认输出保存在第一个本地文件所在目录；使用手机模板时文件名为 `sing-box-phone.json`。

也可以用 `-c` 指定其他模板：

```sh
python3 sub2singbox.py clash.yaml -c custom-config.json -o sing-box.json
```

### 按平台使用模板

`config_openwrt.json` 和 `config_phone.json` 是分别面向 OpenWrt 与手机端的模板。转换时用 `-c` 选择模板；以下命令假设文件都在 `~/Downloads`：

**OpenWrt：**

```sh
cd ~/Downloads
python3 sub2singbox.py mihomo.yaml \
  -c config_openwrt.json
```

省略 `-o` 时会在输入文件所在目录生成 `sing-box-openwrt.json`。OpenWrt 模板为 Linux TUN 启用了 `auto_route` 和 `auto_redirect`，并在 `0.0.0.0:7890` 提供 mixed 代理入口，供局域网设备使用。请在 OpenWrt 防火墙中将该端口限制为 LAN 访问，不要暴露到 WAN。将生成的配置放到路由器后，可用 `sing-box run -c sing-box-openwrt.json` 启动。

**手机端（Android/iOS）：**

```sh
cd ~/Downloads
python3 sub2singbox.py mihomo.yaml \
  -c config_phone.json
```

省略 `-o` 时会在输入文件所在目录生成 `sing-box-phone.json`。手机模板保留 `auto_route`，不配置 `auto_redirect`，避免手机端因平台或权限差异无法初始化该功能。mixed 代理入口只监听 `127.0.0.1:7890`，不会向局域网开放。将生成的配置导入手机端 sing-box 客户端，并按客户端提示授予 VPN/TUN 权限。

两份模板共用 DNS、策略组和分流规则；X 策略组承载 Twitter 分流，Tencent 规则集走直连。转换器会把订阅节点加入相应策略组。选择与运行设备匹配的模板，不要把 OpenWrt 模板导入手机端。

### 转换 Clash YAML 订阅链接

```sh
python3 sub2singbox.py \
  "https://example.com/your-clash-subscription" \
  -c config_phone.json
```

也可以一次合并多个订阅链接；链接和本地文件可以混合传入：

```sh
python3 sub2singbox.py \
  "https://example.com/subscription-jp" \
  "https://example.com/subscription-sg" \
  clash-us.yaml \
  -c config_phone.json
```

### 转换普通 Base64 订阅

```sh
python3 sub2singbox.py \
  "https://example.com/your-base64-subscription" \
  -c config_phone.json
```

### 转换明文 URI 文件

将节点 URI 保存到 `nodes.txt`，例如：

```text
vless://uuid@example.com:443?type=ws&security=tls&path=%2Fws#节点1
trojan://password@example.com:443?sni=example.com#节点2
```

然后运行：

```sh
python3 sub2singbox.py nodes.txt -c config_phone.json
```

### 常用参数

```text
-o, --output   指定输出文件；省略时保存在输入文件目录并按模板名生成平台文件名
-c, --config   必填；指定平台 sing-box JSON 模板（config_phone.json 或 config_openwrt.json）
-h, --help     显示帮助
```

查看完整命令帮助：

```sh
python3 sub2singbox.py --help
```

## 转换范围

脚本会将 Clash 配置中的 `proxies` 节点转换为 sing-box 出站，并保留 JSON 模板中的 DNS、路由、入站、规则集等配置。脚本不会把 Clash 配置中的以下部分转换成 sing-box 规则：

```yaml
proxy-groups:
rules:
rule-providers:
proxy-providers:
```

如果模板包含 `手动选择`、`自动选择` 以及地区手动/自动策略组，转换器会将节点加入相应组。当前模板提供日本、新加坡（狮城）、香港和美国的地区自动测速组；地区根据节点名称识别，自动组只会纳入匹配地区的节点。转换器不会一比一复刻 Clash 的代理组和分流规则。

## 常见跳过原因

如果某些节点被跳过，终端会显示类似信息：

```text
[跳过] 节点名称：暂不支持 Clash 节点类型：某协议
```

常见原因包括：

- 节点协议暂未实现
- Clash YAML 字段名称不符合标准格式
- 节点缺少 `server`、`port`、`uuid` 或 `password`
- 节点使用特殊插件或自定义字段