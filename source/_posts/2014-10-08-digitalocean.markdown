---
layout: post
title: "自宅サーバからDigitalOceanのVPSへ引っ越した"
date: 2014-10-08 14:06:37 +0900
comments: true
categories: VPS
---
## DigitalOceanとは
1時間1円で借りられる格安VPSサーバです．  
今回借りた最安プラン（一ヶ月5ドル）のスペックは

* CPU: 1 Core
* Memory: 512MB
* Storage: 20GB SSD
* 転送量: 1TB

とまずまず．  
<!-- more -->
さくらVPSとAWSの中間のような機能を持っており，お手軽にIaaSを体感することができます．  
SSDのおかげで体感速度はなかなかのものです．

## ベンチマーク
* リージョン: シンガポール
* OS: CentOS 7

### cpuinfo
``` bash
$ cat /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
cpu family      : 6
model             : 62
model name        : Intel(R) Xeon(R) CPU E5-2630L v2 @ 2.40GHz
stepping          : 4
microcode         : 0x1
cpu MHz             : 2399.998
cache size          : 4096 KB
physical id         : 0
siblings : 1
core id    : 0
cpu cores  : 1
apicid       : 0
initial apicid : 0
fpu            : yes
fpu_exception  : yes
cpuid level    : 13
wp             : yes
flags            : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon rep_good nopl eagerfpu pni pclmulqdq vmx ssse3 cx16 pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm xsaveopt fsgsbase tsc_adjust smep erms
bogomips         : 4799.99
clflush size     : 64
cache_alignment  : 64
address sizes    : 40 bits physical, 48 bits virtual
power management:
```

### UnixBench
``` bash
$ ./Run
(snip)
------------------------------------------------------------------------
Benchmark Run: Wed Oct 08 2014 15:41:14 - 16:08:57
1 CPU in system; running 1 parallel copy of tests

Dhrystone 2 using register variables       29081440.4 lps   (10.0 s, 7 samples)
Double-Precision Whetstone                     3607.8 MWIPS (8.2 s, 7 samples)
Execl Throughput                               3347.0 lps   (30.0 s, 2 samples)
File Copy 1024 bufsize 2000 maxblocks        871848.0 KBps  (30.0 s, 2 samples)
File Copy 256 bufsize 500 maxblocks          225866.5 KBps  (30.0 s, 2 samples)
File Copy 4096 bufsize 8000 maxblocks       1427132.0 KBps  (30.0 s, 2 samples)
Pipe Throughput                             1665126.8 lps   (10.0 s, 7 samples)
Pipe-based Context Switching                 226668.9 lps   (10.0 s, 7 samples)
Process Creation                               9210.5 lps   (30.0 s, 2 samples)
Shell Scripts (1 concurrent)                   4643.1 lpm   (60.0 s, 2 samples)
Shell Scripts (8 concurrent)                    619.2 lpm   (60.1 s, 2 samples)
System Call Overhead                        2135413.1 lps   (10.0 s, 7 samples)

System Benchmarks Index Values               BASELINE       RESULT    INDEX
Dhrystone 2 using register variables         116700.0   29081440.4   2492.0
Double-Precision Whetstone                       55.0       3607.8    656.0
Execl Throughput                                 43.0       3347.0    778.4
File Copy 1024 bufsize 2000 maxblocks          3960.0     871848.0   2201.6
File Copy 256 bufsize 500 maxblocks            1655.0     225866.5   1364.8
File Copy 4096 bufsize 8000 maxblocks          5800.0    1427132.0   2460.6
Pipe Throughput                               12440.0    1665126.8   1338.5
Pipe-based Context Switching                   4000.0     226668.9    566.7
Process Creation                                126.0       9210.5    731.0
Shell Scripts (1 concurrent)                     42.4       4643.1   1095.1
Shell Scripts (8 concurrent)                      6.0        619.2   1031.9
System Call Overhead                          15000.0    2135413.1   1423.6
                                                                   ========
System Benchmarks Index Score                                        1193.9
```

### hdparm
``` bash
$ sudo hdparm -Tt /dev/vda1
/dev/vda1:
 Timing cached reads:   17478 MB in  2.00 seconds = 8750.56 MB/sec
 Timing buffered disk reads: 2612 MB in  3.00 seconds = 869.81 MB/sec
```

### Ping
``` bash
$ ping -c 10 www.shironoo.org
PING www.shironoo.org (128.199.136.12): 56 data bytes
64 bytes from 128.199.136.12: icmp_seq=0 ttl=49 time=78.354 ms
64 bytes from 128.199.136.12: icmp_seq=1 ttl=49 time=77.436 ms
(snip)
64 bytes from 128.199.136.12: icmp_seq=9 ttl=49 time=82.251 ms

--- www.shironoo.org ping statistics ---
10 packets transmitted, 10 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 77.436/78.479/82.251/1.337 ms
```

## 注意事項
IPv6でメール配送が上手くいかなくてつまづきました．  
調べたところ，IPv6のみOP25Bをやっているようです．  
https://www.digitalocean.com/community/questions/outgoing-smtp-over-ipv6-on-london-location

とりあえずPostfixのメール配送にIPv4を優先して用いるようにして対処しました．
``` bash
$ vi /etc/postfix/main.cf
smtp_address_preference = ipv4
```

## まとめ
検証環境のためにちょっとサーバが欲しいんだけど・・・というときにぴったりのVPSです．  
APIも充実しているので，スクリプトとの親和性も高そう．  
日本から最寄りのリージョンはシンガポールなので，そこまでアクセス速度が問題になることもないかと．

次のリンクから登録していただけると僕が喜びます．  
登録者も$10のボーナスが貰えるそうなので是非．  
https://www.digitalocean.com/?refcode=3e87155bd345
