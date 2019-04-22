---
layout: single
title: 免費HTTPS憑證懶人版 LET’S ENCRYPT/CERBOT
categories: [note]
---

近期筆者的信箱收到`Let’s Encrypt certificate expiration notice for domain`的訊息，後來發現是`certbot`版本太舊，且`Let’s Encrypt` 本身已經移除了對TLS-SNI-01的domain驗證的支援，如果您的版本也是在0.28以下，只要『再』照著官方提供的步驟(或是下方的步驟)照常執行一次即可。若不知道版本可以透過`certbot --version`得知，筆者的版本為0.19，所以就照著以下的步驟逐一更新即可，費時不到5分鐘。

以下要介紹如何利用Cerbot這個自動化工具讓你的網站使用Let’s Encrypt提供的免費憑證。

使用方式也相當簡單直接參照[certbot](https://certbot.eff.org/)的網站，所需的工具如下：

1. 一個具有使用權限( shell access )的host
2. 一個Http Server
3. 知道你的作業系統
4. 約5分鐘的時間

然後選擇你的`Http Server`和 `System`，像筆者使用`Apache`及`Ubuntu`，選擇完後會直接跳到安裝說明。

接著直接照個安裝說明一連串的安裝(記得要進入你的Host):

```bash
sudo apt-get update

sudo apt-get install software-properties-common

sudo add-apt-repository universe

sudo add-apt-repository ppa:certbot/certbot

sudo apt-get update

sudo apt-get install certbot python-certbot-apache
```

安裝完後如果適用apache當作server的話，可以再加上

```bash
sudo cerbot --apache
```

輸入上述指令時只需要偶而按個Ｙ確定，或是選擇要加密的網站(輸入數字)，或是要輸入聯絡的email，照著指令上的描述一一輸入即可。

另外也可以使用`Automating renewal`，可以自動更新憑證，免除三個月就要定期更新的煩惱：

```bash
sudo certbot renew --dry-run
```

接著只要打開您的網站應該就完成了。

