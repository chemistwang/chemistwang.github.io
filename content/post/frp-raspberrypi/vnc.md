---
title: "VNC连接树莓派无法显示"
subtitle: ""
date: 2021-10-05 19:00:00
author: "汪洋龙"
tags:
  - raspberrypi
categories: [Tech]
image: "img/frp-raspberrypi-bg.jpg"
excerpt: ""
description: ""
---

# VNC 无法显示

问题描述：树莓派开启 `VNC` 之后，想通过 `VNC` 连接，但总是显示 `Cannot currently show the desktop`

原因：如果树莓派未检测到 `hdmi`，则默认不输出信号

解决方案：

1. 获取 ip 地址

```bash
ping raspberrypi.local
```

2. 连接到树莓派

```bash
ssh pi@192.168.1.17
```

3. 修改 `/boot/config.txt` 配置文件

```bash
sudo vi /boot/config.txt
```

```bash
# uncomment if hdmi display is not detected and composite is being output
# hdmi_force_hotplug=1
hdmi_force_hotplug=1 # 取消注释即可
```

4. 重启

```bash
sudo reboot
```
