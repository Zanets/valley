---
title: "Protect SSH Server using endless"
date: 2020-10-025T22:59:37+08:00
draft: false
---

# Introduction

最近在樹莓派上跑了一些Service，很快的SSH server就收到了來自世界的問候

```
Oct 24 09:51:33 ubuntu sshd[6415]: Failed password for root from 85.209.0.101 port 64104 ssh2
Oct 24 11:22:43 ubuntu sshd[7285]: Failed password for root from 85.209.0.100 port 37064 ssh2
Oct 24 11:58:17 ubuntu sshd[7528]: Failed password for root from 85.209.0.252 port 49888 ssh2
Oct 24 12:01:09 ubuntu sshd[7543]: Failed password for root from 141.98.10.213 port 35915 ssh2
Oct 24 12:01:26 ubuntu sshd[7547]: Failed password for invalid user 1234 from 141.98.10.209 port 45834 ssh2
```

馬上採取了幾個措施

1. 不使用預設 22 port
2. 禁止ssh root login
3. 禁止密碼登入，一律使用金鑰
4. 使用 fail2ban，ban掉登入失敗的IP

幾招下來後這幾天都平安無事，不過今天在github上看到一個有趣的專案

https://github.com/skeeto/endlessh

這是一個Tarpit服務，根據Wiki

> **Tarpit** 在計算機網絡是指一個服務或計算機系統，故意延遲響應收到的連接。用於防禦計算機蠕蟲與網絡濫用。如用戶程序掃描網段內的服務，但如果服務的延遲較大就不會去用它。
>
> Tarpit 原意是指能把動物陷入沒頂之災的沼澤地。

原理其實蠻簡單的，這是一個假的SSH server，根據RFC4253，TCP連線建立後SSH server首先發送

````
SSH-protoversion-softwareversion SP comments CR LF
````

而在發送這段訊息之前，可以發送其他訊息且沒有行數限制，只要每行不超過255字元且不以`SSH-`開頭。

endlessh 會無限傳送隨機字串，且每行間隔10秒，這能延長Protocol建立的時間，而不觸發Timeout。

因此所有嘗試連線到該Server的攻擊就會一直處在連線中，這能

1. 減慢攻擊者攻擊的速度
2. 佔據攻擊者的資源，這些資源可被用來攻擊其他服務器
3. 隱藏使用其他port的service

不過在實際測試時，SSH client 直接斷開了，沒有達到預期的效果，就先知道有Tarpit這種防禦手段就好。