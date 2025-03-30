# 新讯 刷 ImmortalWrt 后对 LED 调整优化

## 背景

默认状态 红色 led 闪烁，蓝色 led 常亮，不明所以。

## 优化后效果

1. 开机启动状态下，红色 led 闪烁，蓝色 led 常亮。
2. 正常状态下，红蓝 led 均为常亮。
3. 长按三秒关机阶段，红色 led 快速闪烁，蓝色 led 熄灭。

## 具体操作

### 配置红色 led 监测 cpu 状态，绿色 led 关闭，蓝色 led 监测 无线网 状态。

```sh
uci set system.led_red=led
uci set system.led_red.name='red:power'
uci set system.led_red.sysfs='red:power'
uci set system.led_red.trigger='cpu'
uci set system.led_red.default='1'

uci set system.led_green=led
uci set system.led_green.name='green:wan'
uci set system.led_green.sysfs='green:wan'
uci set system.led_green.trigger='none'
uci set system.led_green.default='0'

uci set system.led_blue=led
uci set system.led_blue.name='blue:wlan'
uci set system.led_blue.sysfs='blue:wlan'
uci set system.led_blue.trigger='phy0radio'
uci set system.led_blue.default='1'

uci commit system
/etc/init.d/led restart

```

### 配置关机阶段，红色 led 快速闪烁，蓝色 led 熄灭。

`vi /etc/rc.button/power`  编辑电源键触发脚本，修改为如下内容并保存：


```sh
#!/bin/sh

[ "${ACTION}" = "released" ] || exit 0

if [ "$SEEN" -ge 3 ]
then
	echo 'none' > /sys/class/leds/blue:wlan/trigger && echo 'timer' > /sys/class/leds/red:power/trigger && echo 100 > /sys/class/leds/red:power/delay_on && echo 100 > /sys/class/leds/red:power/delay_off
	exec /sbin/poweroff
fi

return 0

```

