# AdGuardHome 配置笔记
## 1. 创建配置目录并运行
```bash
mkdir -p /etc/AdGuardHome
/usr/bin/AdGuardHome -w /etc/AdGuardHome -s run
```
## 2. Web管理界面访问
```
http://192.168.3.1:3000
```
### 2.1 网络配置
| 项目 | 配置值 |
|------|--------|
| 路由器IP | 192.168.3.1 |
| DHCP起始地址 | 192.168.3.100 |
| DHCP结束地址 | 192.168.3.200 |
| 子网掩码 | 255.255.255.0 |
| DHCP租约时间 | 86400 秒 |
| IPv6网关 | 2001::1 |
### 2.2 上游DNS服务器配置
### DoH (DNS over HTTPS)
```
https://dns.alidns.com/dns-query
https://dns.pub/dns-query
https://sm2.doh.pub/dns-query
https://doh-pure.onedns.net/dns-query
https://public.adguardprivate.com/dns-query
tls://dns.alidns.com
tls://dot.pub
quic://dns.alidns.com:853
```
### 2.3 广告过滤规则
```
https://gcore.jsdelivr.net/gh/217heidai/adblockfilters@main/rules/adblockdns.txt
```
### 2.4 完成初始配置后 Ctrl + C 退出前台运行
## 3. 确认配置文件存在
```bash
ls -l /etc/AdGuardHome/AdGuardHome.yaml
```
## 4. 启动服务
```bash
/etc/init.d/AdGuardHome start
```

---

# 故障排查
## 1. 查看日志
```bash
logread | grep AdGuard
```
## 2. 检查运行状态
```bash
# 检查DNS端口(53)监听
netstat -antp | grep :53
# 检查进程
ps | grep AdGuardHome
```
## 3. 修复启动脚本
```bash
sed -i 's/procd_set_param command \$cmd\$cmd_args/procd_set_param command $cmd $cmd_args/g' /etc/init.d/AdGuardHome
```