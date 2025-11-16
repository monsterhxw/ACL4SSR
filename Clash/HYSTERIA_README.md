# Hysteria 代理协议支持文档

本文档介绍如何在 ACL4SSR 规则集中使用 Hysteria 和 Hysteria2 代理协议。

## 概述

Hysteria 是一个功能强大、快速且抗审查的代理协议，基于 QUIC 协议构建，提供卓越的性能和稳定性。

### 支持的版本

- **Hysteria (v1)**: 第一代协议，支持多种传输模式
- **Hysteria2 (v2)**: 第二代协议，简化配置，性能更优

## 快速开始

### 1. 配置示例文件

参考 `hysteria_example.yml` 文件获取完整的配置示例：

```yaml
proxies:
  - name: "Hysteria-Example"
    type: hysteria
    server: hysteria.example.com
    port: 443
    auth: your_password
    protocol: udp
    alpn:
      - h3
    up: 50
    down: 100
```

### 2. Subconverter 配置

使用 `ACL4SSR_Hysteria.ini` 配置模板，该模板包含：
- Hysteria 节点自动分组
- 针对 Hysteria 协议优化的规则集
- 自动筛选带有 Hysteria 关键字的节点

## 详细配置说明

### Hysteria (v1) 配置参数

#### 必需参数

| 参数 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `name` | string | 节点名称 | `"Hysteria-HK"` |
| `type` | string | 协议类型，固定为 `hysteria` | `hysteria` |
| `server` | string | 服务器地址 | `hysteria.example.com` |
| `port` | int | 服务器端口 | `443` |
| `auth` | string | 认证密码 | `your_password` |

#### 可选参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `protocol` | string | `udp` | 传输协议：`udp`, `wechat-video`, `faketcp` |
| `up` | int | - | 上传带宽 (Mbps) |
| `down` | int | - | 下载带宽 (Mbps) |
| `obfs` | string | - | 混淆方式：`salamander` |
| `obfs-password` | string | - | 混淆密码 |
| `sni` | string | - | TLS SNI |
| `skip-cert-verify` | bool | `false` | 跳过证书验证 |
| `alpn` | array | - | ALPN 协议，推荐 `["h3"]` |
| `recv-window-conn` | int | - | QUIC 流接收窗口大小 |
| `recv-window` | int | - | QUIC 连接接收窗口大小 |
| `disable-mtu-discovery` | bool | `false` | 禁用 MTU 发现 |
| `fast-open` | bool | `false` | 启用快速打开 |
| `ca` | string | - | 自定义 CA 证书路径 |
| `ca-str` | string | - | 自定义 CA 证书内容 |

### Hysteria2 (v2) 配置参数

#### 必需参数

| 参数 | 类型 | 说明 | 示例 |
|------|------|------|------|
| `name` | string | 节点名称 | `"Hysteria2-US"` |
| `type` | string | 协议类型，固定为 `hysteria2` | `hysteria2` |
| `server` | string | 服务器地址 | `hysteria2.example.com` |
| `port` | int | 服务器端口 | `443` |
| `password` | string | 认证密码 | `your_password` |

#### 可选参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `obfs` | string | - | 混淆类型：`salamander` |
| `obfs-password` | string | - | 混淆密码 |
| `up` | int | - | 上传带宽限制 (Mbps) |
| `down` | int | - | 下载带宽限制 (Mbps) |
| `sni` | string | - | TLS SNI |
| `skip-cert-verify` | bool | `false` | 跳过证书验证 |
| `alpn` | array | `["h3"]` | ALPN 协议 |
| `udp` | bool | `true` | 启用 UDP 中继 |
| `ca` | string | - | 自定义 CA 证书路径 |
| `ca-str` | string | - | 自定义 CA 证书内容 |

## 配置文件示例

### 基础配置

#### Hysteria (v1)

```yaml
proxies:
  - name: "Hysteria-Basic"
    type: hysteria
    server: server.example.com
    port: 443
    auth: password123
    protocol: udp
    alpn:
      - h3
    up: 50
    down: 100
```

#### Hysteria2 (v2) - 推荐

```yaml
proxies:
  - name: "Hysteria2-Basic"
    type: hysteria2
    server: server.example.com
    port: 443
    password: password123
```

### 高级配置

#### 带混淆的 Hysteria

```yaml
proxies:
  - name: "Hysteria-Obfs"
    type: hysteria
    server: server.example.com
    port: 443
    auth: password123
    protocol: udp
    obfs: salamander
    obfs-password: obfs_secret
    sni: server.example.com
    alpn:
      - h3
    up: 100
    down: 200
    recv-window-conn: 67108864
    recv-window: 134217728
```

#### 带自定义 CA 的 Hysteria2

```yaml
proxies:
  - name: "Hysteria2-CustomCA"
    type: hysteria2
    server: server.example.com
    port: 443
    password: password123
    sni: server.example.com
    ca: /path/to/ca.crt
    obfs: salamander
    obfs-password: obfs_secret
    up: 100
    down: 200
```

## 代理组配置

### 选择组

```yaml
proxy-groups:
  - name: "Hysteria节点"
    type: select
    proxies:
      - Hysteria-HK
      - Hysteria-US
      - Hysteria2-JP
```

### 自动测速组

```yaml
proxy-groups:
  - name: "Hysteria自动"
    type: url-test
    proxies:
      - Hysteria-HK
      - Hysteria-US
      - Hysteria2-JP
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
```

### 负载均衡组

```yaml
proxy-groups:
  - name: "Hysteria负载均衡"
    type: load-balance
    proxies:
      - Hysteria-HK
      - Hysteria-US
      - Hysteria2-JP
    url: 'http://www.gstatic.com/generate_204'
    interval: 300
```

## Subconverter 使用

### 在 INI 配置中筛选 Hysteria 节点

```ini
; 选择所有 Hysteria 节点
custom_proxy_group=⚡ Hysteria节点`select`(Hysteria2?|HYSTERIA2?|HY-?2|Hy-?2|HY2|Hy2)

; 自动测速 Hysteria 节点
custom_proxy_group=⚡ Hysteria自动`url-test`(Hysteria|hysteria|HY)`http://www.gstatic.com/generate_204`300,,50

; 香港 Hysteria 节点
custom_proxy_group=🇭🇰 HK-Hysteria`url-test`(港|HK).*?(Hysteria|HY)`http://www.gstatic.com/generate_204`300,,50
```

### 使用专用配置模板

```bash
# 使用 ACL4SSR_Hysteria.ini 模板
https://api.example.com/sub?target=clash&url=YOUR_SUB_URL&config=https://raw.githubusercontent.com/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Hysteria.ini
```

## 性能优化建议

### 1. 带宽设置

根据实际网络环境设置 `up` 和 `down` 参数：

- **家庭宽带**: `up: 50`, `down: 100`
- **企业网络**: `up: 100`, `down: 200`
- **服务器**: `up: 500`, `down: 1000`

### 2. 混淆配置

使用混淆可以提高抗封锁能力：

```yaml
obfs: salamander
obfs-password: random_strong_password
```

### 3. QUIC 窗口大小

高延迟网络建议增加窗口大小：

```yaml
recv-window-conn: 67108864    # 64 MB
recv-window: 134217728         # 128 MB
```

### 4. ALPN 设置

推荐使用 HTTP/3：

```yaml
alpn:
  - h3
```

## 常见问题

### Q: Hysteria 和 Hysteria2 有什么区别？

**A**: Hysteria2 是改进版本，具有：
- 简化的配置参数
- 更好的性能
- 更稳定的连接
- 推荐新用户使用 Hysteria2

### Q: 如何选择 protocol 参数？

**A**: 对于 Hysteria v1：
- `udp`: 默认，性能最好
- `wechat-video`: 伪装成微信视频通话
- `faketcp`: 伪装成 TCP 连接

### Q: 带宽参数必须设置吗？

**A**: 不是必须的，但建议设置：
- 设置后可以获得更稳定的速度
- 避免占用过多带宽
- 提高整体连接质量

### Q: 如何验证 Hysteria 节点是否工作？

**A**: 可以通过以下方式验证：
1. 在 Clash 中查看节点延迟
2. 访问 https://ip.sb 检查 IP 地址
3. 运行速度测试

### Q: skip-cert-verify 应该设置为 true 吗？

**A**: **不建议**在生产环境中设置为 `true`，除非：
- 使用自签名证书进行测试
- 确认服务器身份
- 了解相关安全风险

## 支持的客户端

### 桌面客户端
- **Clash for Windows**: 完全支持
- **ClashX Pro**: 完全支持
- **Clash Verge**: 完全支持

### 移动客户端
- **Clash Meta for Android**: 完全支持
- **Stash (iOS)**: 支持 Hysteria2

### 内核支持
- **Clash.Meta**: 完全支持 Hysteria 和 Hysteria2
- **Clash Premium**: 部分支持

## 相关资源

- [Hysteria 官方网站](https://hysteria.network/)
- [Hysteria GitHub](https://github.com/apernet/hysteria)
- [Clash Meta 文档](https://wiki.metacubex.one/)
- [ACL4SSR 项目主页](https://github.com/ACL4SSR/ACL4SSR)

## 示例文件

本仓库提供以下 Hysteria 相关文件：

1. **hysteria_example.yml**: 完整的配置示例
2. **GeneralClashConfig.yml**: 包含 Hysteria 示例的通用配置
3. **config/ACL4SSR_Hysteria.ini**: Subconverter 专用配置模板

## 更新日志

### 2024-11
- 添加 Hysteria 和 Hysteria2 协议支持
- 创建专用配置模板
- 添加详细文档和示例

## 贡献

欢迎提交 Issue 和 Pull Request 来改进 Hysteria 支持。

## 许可证

本项目基于 CC-BY-SA-4.0 协议发布。
