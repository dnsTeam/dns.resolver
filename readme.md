## **✨ dns.resolver ✨**

一个缓存优先的 DNS 解析程序，可以在 Windows、Linux 和 macOS 中运行，用作本地或局域网 DNS 服务器。

  > 如果你喜欢或觉得有用，给它一个 GitHub Star 。

## **1. 拓扑结构**

<p align="center"><img src="img/topo.png" /></p>

## **2. 主要功效**

- **加速客户端响应**
  * 缓存命中时立即返回响应结果，不必每次请求上游服务器；
    > 开发环境 dig 输出 ` Query time: 0 msec ` 。
  * 一般情况下，局域网带宽更高、响应时间更短。

- **提升安全性和隐私性**
  * 支持 ` DoH ( DNS-over-HTTPS ) ` 协议访问上游服务器。
  * 基于行业通用的 HTTPS 安全协议，传输过程中请求和响应都是对称加密的，可有效防止第三方监视和篡改，从此告别 DNS 劫持污染。

- **自定义域名解析**【待实现】
  * 基于用户配置，自定义特定域名的 DNS 响应结果。
  * 支持 A、AAAA、CName、MX 等常见记录类型。

- **拦截广告、跟踪、恶意软件、钓鱼或欺诈域名等**【待实现】
  * 基于规则进行判断，命中时返回 NxDomain 状态码。
  * 规则清单可通过互联网进行刷新。

## **3. 安装运行**

- 下载相应版本的应用压缩包，解压到合适的文件夹，例如： ` C:\dns.resolver ` 。
- 复制一份 ` appsettings.sample.json ` 文件，重命名为 ` appsettings.json ` 。
- 根据需要修改 ` appsettings.json ` 中的配置信息；详见 ` 4. 配置文件说明 ` 。
- 根据需要打开防火墙，允许应用通过防火墙，或开放配置文件中的本地侦听端口 。
- 根据需要为应用程序文件授予运行权限。

### **3.1. 直接运行**

在 windows、Linux 和 macOS 中直接双击打开运行应用程序，或在命令行中运行应用程序。

  > 应用显示 `Running... Press <enter> key to exit.` 提示信息。

试用的时候，可以选择此方式。

### **3.2. 任务计划**

以 windows 为例：
- 在 ` 开始 ` 菜单 ` Windows 管理工具 ` 中，打开 ` 任务计划程序 ` 。
- 点击 ` 创建任务 ` ，
  * 输入名称 ` dns.resolver ` ，
  * 选择 ` 不管用户是否登录都要运行 ` 。
- 触发器页签，点击 ` 新建 ` ；
  *  ` 开始任务 ` 选择 ` 启动时 ` ；
  * 根据需要设置 ` 延迟任务时间 ` ，实现启动一段时间后运行的效果。
- 操作页签，点击 ` 新建 ` ；
  *  ` 操作 ` 选择 ` 启动程序 ` ；
  *  ` 程序或脚本 ` 为应用全路径，例如： ` C:\dns.resolver\dns.exe ` ；
  *  ` 起始于 ` 为应用所在文件夹，例如： ` C:\dns.resolver ` 。
- 条件页签，
  * 根据需要，去除 ` 只有在计算机使用交流电源时才启动此任务 ` 选项。
- 设置页签，
  * 根据需要，去除 ` 如果任务运行时间超过以下时间，停止任务 ` 选项。
- 点击 ` 确定 ` ，输入用户名和密码，保存任务。

## **4. 配置文件说明**

配置文件 appsettings.json 位于应用程序相同的文件夹下。

### **4.1. 样例**

采用 JSON 格式，如下所示：

```
{
    "dns": {
        "local": {
            "bind": [ "127.0.0.1", "A.B.C.D" ],
            "timeout": 1000
        },
        "upstream": {
            "server": [
                "https://1.12.12.12/dns-query",    // DNSPod（腾讯），DoH (IP)
                "https://120.53.53.53/dns-query",  // DNSPod（腾讯），DoH (IP)
                "https://223.5.5.5/dns-query",     // AliDNS（阿里），DoH (IP)
                "https://223.6.6.6/dns-query",     // AliDNS（阿里），DoH (IP)
                "https://101.226.4.6/dns-query",   // 360 安全 DNS，DoH (IP)，电信、移动、铁通首选
                "https://218.30.118.6/dns-query",  // 360 安全 DNS，DoH (IP)，电信、移动、铁通备用
                "https://123.125.81.6/dns-query",  // 360 安全 DNS，DoH (IP)，联通首选
                "https://140.207.198.6/dns-query", // 360 安全 DNS，DoH (IP)，联通备用
                "/name.lan/A1.B1.C1.D1",           // 解析 name.lan 的上游服务器 1
                "/name.lan/A2.B2.C2.D2"            // 解析 name.lan 的上游服务器 2
            ],
            "mode": "any",
            "timeout": 2000
        },
        "cache": {
            "timeout": 600000,
            "refreshOnCall": false
        }
    }
}
```

### **4.2. dns 配置节**

DNS 相关配置的根节点。

#### **4.2.1. dns/local 配置节**

DNS 本地配置信息。

- **bind：本地绑定信息。**

  * 格式： ` “ 协议 + IP 地址 + 端口号 ” ` 。
    + 多个本地绑定用 ` 英文逗号 ` 分隔。
  * 建议首选 ` 默认 udp 协议、默认 53 端口 ` 方式，性能和兼容性都很好。
  * 示例：
    + ` 127.0.0.1 ` ：默认 ` udp ` 协议、默认 ` 53 ` 端口；
    + ` 127.0.0.1:53 ` ：默认 ` udp ` 协议、指定 ` 53 ` 端口；
    + ` udp://127.0.0.1 ` ：指定 ` udp ` 协议、默认 ` 53 ` 端口；
    + ` udp://127.0.0.1:53 ` ：指定 ` udp ` 协议、指定 ` 53 ` 端口；
    + ` tcp://127.0.0.1 ` ：指定 ` tcp ` 协议、默认 ` 53 ` 端口；
    + ` tcp://127.0.0.1:53 ` ：指定 ` tcp ` 协议、指定 ` 53 ` 端口。
  * 用作本地 DNS 服务器，可设为 ` [ "127.0.0.1" ] ` ( IPv4 ) 或 ` [ "::1" ] ` ( IPv6 ) 。
  * 用作局域网 DNS 服务器，可设为 ` [ "A.B.C.D" ] ` ( IPv4 ) 或 ` [ "A:B:C:D:E:F:G:H" ] ` ( IPv6 ) 。
    + ` A.B.C.D ` 为本地网络连接的局域网 IPv4 地址 。
    + ` A:B:C:D:E:F:G:H ` 为本地网络连接的局域网 IPv6 地址 。
      > IPv6 局域网地址 ( Unique Local Address，ULA ) 为 ` FC00::/7 ` 网段，详见 [ ` RFC4193 ` ](https://www.rfc-editor.org/rfc/rfc4193.html)。

- **timeout**

  * 客户端访问本应用，单次交互接收数据和发送数据的超时时间，单位：毫秒。

#### **4.2.2. dns/upstream 配置节**

DNS 上游服务器配置信息。

- **server：上游服务器信息**

  * 格式： ` “限定域名 + 协议 + IP 地址 + 端口号” ` 。
    + 多个上游服务器用 ` 英文逗号 ` 分隔。
  * 建议：
    + 局域网首选 ` 默认 udp 协议、默认 53 端口 ` 方式，
    + 互联网首选 ` DoH ( DNS-over-HTTPS ) ` 方式。
    + 默认上游服务一般指定 2 - 5 个即可。
  * 示例：
    + ` A.B.C.D ` ：默认 ` udp ` 协议、默认 ` 53 ` 端口；
    + ` A.B.C.D:53 ` ：默认 ` udp ` 协议、指定 ` 53 ` 端口；
    + ` udp://A.B.C.D ` ：指定 ` udp ` 协议、默认 ` 53 ` 端口；
    + ` udp://A.B.C.D:53 ` ：指定 ` udp ` 协议、指定 ` 53 ` 端口；
    + ` tcp://A.B.C.D ` ：指定 ` tcp ` 协议、默认 ` 53 ` 端口；
    + ` tcp://A.B.C.D:53 ` ：指定 ` tcp ` 协议、指定 ` 53 ` 端口；
    + ` https://A.B.C.D/dns-query ` ：加密 DoH ( DNS-over-HTTPS )；
    + ` /name.lan/A.B.C.D" ` ：为特定域名 ` name.lan ` 指定上游服务器。
  * 采用 IP 地址可避免上游服务器在多个运营商网络切换带来性能波动。

- **mode**

  * 缓存未命中时，同时访问多个上游服务器的等待模式，有 any 和 all 两个选项。
    + any : 任一上游服务器返回结果后继续；其他上游服务器在后台处理。 `【建议】`
    + all : 等待所有的上游服务器，全部返回结果后再继续。
  * 多个上游服务器的响应结果，去重后存储到缓存中。

- **timeout**

  * 本应用访问上游服务器，单次交互接收数据和发送数据的超时时间，单位：毫秒。

#### **4.2.3. dns/cache 配置节**

缓存上游服务器解析结果的配置信息。

- **timeout**

  * 缓存数据保持有效的超时时间，单位：毫秒。

- **refreshOnCall**

  * 有效期内，客户端请求时，立即返回命中的缓存结果。
  * 当 ` refreshOnCall = true ` 时，启动后台刷新任务，更新缓存内容，滑动缓存有效期。

## **5. 验证 DNS 响应**

- 在 windows 命令行中可输入 ` nslookup github.com server ` 。
- 在 linux 命令行中可输入 ` dig github.com @server ` 。

## **6. 已知的公共 DNS 服务器**

- **DNSPod（腾讯）**

|      协议      |            地址                  |
| :------------: | :------------------------------: |
| DNS, IPv4      | `119.29.29.29`                   |
| DNS, IPv4      | `119.28.28.28`                   |
| DNS, IPv6      | `2402:4e00::`                    |
| DNS-over-HTTPS | `https://1.12.12.12/dns-query`   |
| DNS-over-HTTPS | `https://120.53.53.53/dns-query` |
<!-- | DNS-over-HTTPS | `https://doh.pub/dns-query`      |
| DNS-over-HTTPS | `https://dns.pub/dns-query`      |
| DNS-over-TLS   | `tls://dot.pub`                   | -->

- **Alidns（阿里巴巴）**

|      协议      |              地址                  |
| :------------: | :--------------------------------: |
| DNS, IPv4      | `223.5.5.5`                        |
| DNS, IPv4      | `223.6.6.6`                        |
| DNS, IPv6      | `2400:3200::1`                     |
| DNS, IPv6      | `2400:3200:baba::1`                |
| DNS-over-HTTPS | `https://223.5.5.5/dns-query`      |
| DNS-over-HTTPS | `https://223.6.6.6/dns-query`      |
<!-- | DNS-over-HTTPS | `https://dns.alidns.com/dns-query` |
| DNS-over-TLS   | `tls://dns.alidns.com`                 | -->

- **360 安全 DNS**

|      协议      |                     地址                       |
| :------------: | :--------------------------------------------: |
| DNS, IPv4（电信/铁通/移动）      | `101.226.4.6`                     |
| DNS, IPv4（电信/铁通/移动）      | `218.30.118.6`                    |
| DNS, IPv4（联通）                | `123.125.81.6`                    |
| DNS, IPv4（联通）                | `140.207.198.6`                   |
| DNS-over-HTTPS（电信/铁通/移动） | `https://101.226.4.6/dns-query`  |
| DNS-over-HTTPS（电信/铁通/移动） | `https://218.30.118.6/dns-query` |
| DNS-over-HTTPS（联通）           | `https://123.125.81.6/dns-query`  |
| DNS-over-HTTPS（联通）           | `https://140.207.198.6/dns-query` |
<!-- | DNS-over-HTTPS              | `https://doh.360.cn/dns-query`    |
| DNS-over-TLS                | `tls://dot.360.cn`                 | -->

## **7. 赞助**

您的支持，我的动力！ 更是本项目获得长期运维的保障！ 万分感谢！

<table border="0" cellspacing="0" cellPadding="0" style="border:0">
<tr>
  <td> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<img src="img/alipay.jpg" height="200" width="200" /> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;</td>
  <td> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<img src="img/wechat.jpg" height="200" width="200" /> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;</td>
</tr>
<tr>
  <td align="center">支付宝扫码</td>
  <td align="center">微信扫码</td>
</tr>
</table>