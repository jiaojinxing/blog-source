title: SylixOS for ARMv7-M 开发硬件说明
date: 2015-06-20 14:43:26
tag: 
- SylixOS 
- ARMv7-M
- 硬件 
categories: 
- SylixOS 
---

##STM32F429I-DISCO
 
STM32F429I-DISCO 是 ST 公司推出的一款针对高性能 STM32F4 系列设计的 Cortex-M4 开发板。

开发板基于 STM32F429ZIT6 设计，开发板还集成了 ST-LINK/V2 仿真下载器（但仅对外提供 SWD 接口），免除您另外采购仿真器或下载器的麻烦。更增添了一块 2.4 寸的 QVGA TFT 液晶屏，外扩 64 Mbits SDRAM，L3GD20 陀螺仪，用户指示灯，按键和一个micro-AB 型 USB 接口。

*STM32F429ZIT6微控制器

2048KB FLASH，256 KB RAM

*规则的引出了所有未被占用的IO口，方便做相关实验

![stm32f4.jpg](/img/SylixOS-for-ARMv7-M开发硬件说明/stm32f4.jpg "")

STM32F429I-DISCO 不带有网络接口、SD 卡接口、音频接口，所以我还另外买了这个三接口模块和一个 WIFI 模块。

￥158：http://item.taobao.com/item.htm?spm=a1z09.2.9.54.B0CAvW&id=43614635387&_u=kr55lbs1a8a

##ENC28J60 网络模块

ENC28J60 网络模块使用 SPI 接口与 STM32F429I-DISCO 连接。

![net.jpg](/img/SylixOS-for-ARMv7-M开发硬件说明/net.jpg "")

￥12.5：http://item.taobao.com/item.htm?spm=a1z09.2.9.43.B0CAvW&id=43235319615&_u=kr55lbs700c

##SD 卡读写模块

SD 卡读写模块使用 SPI 接口与 STM32F429I-DISCO 连接。

![sd.jpg](/img/SylixOS-for-ARMv7-M开发硬件说明/sd.jpg "")

￥2.1：http://item.taobao.com/item.htm?spm=a1z09.2.9.33.B0CAvW&id=43670337040&_u=kr55lbs0a05

##音频模块

网上较多的是 VS1053 模块，但我不喜欢，所以我买的是“野火”家的 PCM1770 DAC + 8002A 功放组合的模块。

音频模块使用音频接口与 STM32F429I-DISCO 连接。

![audio.jpg](/img/SylixOS-for-ARMv7-M开发硬件说明/audio.jpg "")

￥24.8：http://detail.tmall.com/item.htm?id=43080517488&spm=a1z09.2.9.11.B0CAvW&_u=kr55lbsd0d4

##WIFI 模块

用的是 ESP8266 串口 WIFI 模块，主要是 ESP8266 内建 LwIP，使用方便，价格便宜。

ESP8266 串口 WIFI 模块使用串口与 STM32F429I-DISCO 连接。

![wifi.jpg](/img/SylixOS-for-ARMv7-M开发硬件说明/wifi.jpg "")

￥15.3：http://item.taobao.com/item.htm?spm=a1z09.2.9.25.B0CAvW&id=43467108738&_u=kr55lbsba4f

整套下来花费约 ￥220，待全部到货，再上一份模块与 STM32F429I-DISCO 的连接教学吧。