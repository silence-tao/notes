# Centos相关笔记

## 1.Centos 7连接无线网络

1.查询可用的无线网卡，命令：`iw dev`

``` bash
[root@cluster-01 ~]# iw dev
phy#0
        Unnamed/non-netdev interface
                wdev 0x2
                addr 60:57:18:a5:9e:49
                type P2P-device
        Interface wlp2s0	# 无线网卡卡号
                ifindex 3
                wdev 0x1
                addr 60:57:18:a5:9e:48
                ssid SHJD-DT-53591 6171
                type managed
                channel 11 (2462 MHz), width: 20 MHz, center1: 2462 MHz
```

2.启动无线网卡

``` bash
$ ip link set wlp2s0 up
```

3.查看所有可用的无线网络信号

``` bash
$ iw wlp2s0 scan | grep SSID
```

示例：

``` bash
[root@cluster-01 ~]# iw wlp2s0 scan | grep SSID
        SSID: SHJD-DT-53591 6171
        SSID: JD_guest
        SSID: Mi_Wifi
        SSID: ljyap
        SSID: Jays iPhone
        SSID: Mac
        SSID: cavatina
        SSID: My_WIFI
```

4.连接无线网络

``` bash
$ wpa_supplicant -B -i wlp2s0 -c <(wpa_passphrase "[SSID]" "[密码]")
```

5.分配IP地址

``` bash
$ dhclient wlp2s0
```

6.查看无线网卡连接情况

``` bash
$ iw wlp2s0 link
```

示例：

``` bash
[root@cluster-01 ~]# iw wlp2s0 link
Connected to 22:52:16:59:9a:1d (on wlp2s0)
        SSID: SHJD-DT-53591 6171
        freq: 2462
        RX: 124673 bytes (1072 packets)
        TX: 80493 bytes (564 packets)
        signal: -43 dBm
        tx bitrate: 72.2 MBit/s MCS 7 short GI

        bss flags:      CTS-protection short-preamble short-slot-time
        dtim period:    1
        beacon int:     100
```

