# UU 加速器插件

注：本插件基于[网易 UU 加速器 OpenWrt 版](https://router.uu.163.com/app/html/online/baike_share.html?baike_id=5f963c9304c215e129ca40e8)适配，初始化时会从UU官方的 API 拉取 uuplugin 二进制，插件 ipkg 包内不分发任何网易公司二进制。UU 加速器的商标、版权归网易公司所有。

## 1. 插件定位

UU 加速器插件（`uu`）将网易 UU 主机加速器的路由器接入能力集成到 iKuai 应用市场。安装后用户可在**网易 UU 主机加速器 App** 中扫描路由器 SN 完成绑定，路由器即作为 UU 加速器的出口节点加速 iKuai 路由局域网下的游戏主机，如 PS4、PS5、*Nintendo* Switch、Xbox 等。

插件本身只负责托管和监控网易官方提供的 `uuplugin` 二进制，自身不含任何加速逻辑。

## 2. 能力范围

### `uud`（守护进程）

Go 编写的守护进程，是插件的核心。`start.sh` 拉起它后，`uud` 负责所有后续工作：

**功能：**

1. **插件管理** — 从 `router.uu.163.com` 查询 `uuplugin` 最新版本，按需下载，MD5 校验后解压到 `/tmp/uu/`
2. **缓存** — `uuplugin` 压缩包持久化到 `app/cache/uu.tar.gz`，重启后直接使用本地缓存（首次启动约 5 秒，有缓存 < 2 秒）
3. **Supervisor** — 每 5 秒检查 `uuplugin` 存活，崩溃自动重启
4. **云控响应**：
   - 检测到 `/tmp/uu/uu.update` → 热更新（下载新版重启）
   - 检测到 `/tmp/uu/uu.uninstall` → 云端解绑，重置 `.sn`
5. **每小时** 向 `router.uu.163.com` 检查更新，发现新版本入库

### `uuplugin`

网易官方二进制，负责实际的加速协议、云端通信和 SN 绑定。由 `uud` 管理，不直接调用。

运行目录：`/tmp/uu/`（重启后丢失，`uud` 会自动重建）

## 3. 工作原理

```
用户扫码
    │
    ▼
UU App → 云端分配真实 SN → 云端下发到 uuplugin
    │
    ▼
uuplugin 写 /tmp/uu/activate_status   ← 激活标记（权威）
         写 /usr/sbin/uu/.sn          ← 部分版本会覆盖占位符
    │
    ▼
uud 检测到 activate_status → 持久化到 app/cache/activate_status
```

**SN 占位符**：插件启动时写入 LAN MAC（如 `aa:bb:cc:dd:ee:ff`）作为 `.sn` 初始值。`uuplugin` 读到 `.sn` 为空会崩溃，因此占位符必须非空。

## 4. 插件配置

安装后，可在“应用市场-已安装-网易UU加速器-日志”看到绑定状态

![log](https://img.meituan.net/csc/aa28652ffe6b6605aa3c3e75a37fee2897172.png)

此时为[等待绑定]状态，使用手机下载「UU 主机加速器」，一般应用市场都有。

App 下载：https://adl.netease.com/d/g/uu/c/uu_router

### 绑定设备

下载好「UU 主机加速器」后，打开并登录，选择「合作款路由器」

![uuapp](https://img.meituan.net/content/47826c4f0e4fb8ecd3419d9bd8ff3c90308149.png)

然后按照指引添加 iKuai 路由，因为使用的是 OpenWrt 插件，可在后续重命名设备修改

![bind](https://img.meituan.net/content/326ed238879f376783fe39f7998adf38651702.png)

此时Web 将会显示已绑定

![done](https://img.meituan.net/csc/0d2104c50d317475c3e256d0791a7cb057698.png)

### 加速游戏主机

打开手机App的主界面，选择刚刚绑定的路由，即可加速！

![acc](https://img.meituan.net/content/cf862925fdd4fd2c8cdf061c39b155c2260643.png)

