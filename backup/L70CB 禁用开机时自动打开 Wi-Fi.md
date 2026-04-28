# L70CB 禁用开机时自动打开 Wi-Fi

## 背景

CPE 放在阳台 5G 信号好的地方，用网线连接到了室内的路由器，不需要用到 CPE 的 Wi-Fi ，而且我这儿 Wi-Fi 多，担心干扰。如果仅仅从网页后台关闭 Wi-Fi 2.4G 和 5G 的开关，断电重启后还是会自动把 Wi-Fi 打开。

## 具体操作

### 电脑浏览器打开 CPE bash 终端 http://192.168.1.1:7681/

### 挂载文件系统为读写模式

```bash
mount -o remount,rw /
```

### 为文件添加可写权限

```ba
chmod +w /etc/network/interfaces
```

### 确认可写权限添加成功

```bash
ls -l /etc/network/interfaces
```

输出结果应为：

```bash
-rw-r--r--    1 root     root           615 Jan  2  2024 /etc/network/interfaces
```

### 编辑文件，注释掉 `wlan0` 的配置

```bash
vi /etc/network/interfaces
```

**先切换到英文输入法。**

浏览器访问的终端不支持方向键，使用 `k`、`j`、`h`、`l` 替代方向键。

`i`键进入编辑模式，`Esc`键退出编辑模式。

使用 `#` 号注释行，如下：

```bash
# iface wlan0 inet dhcp
#       wireless_mode managed
#       wireless_essid any
#       wpa-driver wext
#       wpa-conf /etc/wpa_supplicant.conf
```

其他行不要改动。

编辑完成后输入 `:wq` 保存并退出。

### 挂载文件系统为只读模式

```bash
mount -o remount,ro /

mount | grep -w /
```

输出结果应为：

```bash
ubi0:system on / type ubifs (ro,relatime,ubi=0,vol=20)
```

## 关机断电重启检查效果，Wi-Fi 不会再自动打开了

