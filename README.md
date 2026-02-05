# 在 docker 環境架設 rsync 伺服器, 透過 SSH

使用 alpine:3.21 建立 rsync 伺服器, 透過 SSH


# 帳號及密碼

設定帳號及密碼, 同時設定UID, GID

```
services:
  rsync:
    ...
    environment:
      # 權限設定 (對齊宿主機 UID/GID)
      - PUID=1000
      - PGID=1000
      # SSH 帳密與密鑰
      - SSH_USER=admin
      - SSH_PASSWORD=12345678
      ...
```

# 使用金鑰認證

在執行排程同步時, 無密碼執行

```
services:
  rsync:
    ...
    environment:
    ...
      # ssh-keygen -t ed25519 -C "su.charlie@gmail.com"
      #- SSH_PUBLIC_KEY=ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFN8iM9lbMnam5bKXCZqks+zEEzsDyb+V1Ti4rsbkB2q su.charlie@gmail.com
      
    ...
```


# 可以限制登入的IP

多組 IP 用逗號分開

```
services:
  rsync:
    ...
    environment:
    ...
      - ALLOW_IP=192.168.50.0/24,192.168.8.0/24
    ...
```

# SSH 服務需要保留金鑰

存放目錄在 /ssh_keys

需要建立 volumes 來存放

```
services:
  rsync:
    ...
    volumes:
      - ssh_host_keys:/ssh_keys
    ...
volumes:
  ssh_host_keys:
```

# 安全性問題

為了安全性, 限定使用者只能在限定目錄內活動

只能在 /data 目錄下活動

導入 rrsync 進行管控

同步時, /data 就等於是根目錄

!!! 使用 22 port 連線

```
rsync --list-only admin@192.168.0.1:/
```

!!! 使用非22 port連線

```
rsync --list-only -e "ssh -p 2223" admin@192.168.0.1:/
```

# rrsync [使用說明][0]

[0]: https://github.com/RsyncProject/rsync/blob/master/support/rrsync.1.md


2021年12月28日 改成 python 版本
<https://github.com/RsyncProject/rsync/commits/master/support/rrsyn>

2021年12月20日 以前是 perl 版本
<https://raw.githubusercontent.com/RsyncProject/rsync/ed19ea05fea83fe7c757a40060ecc54e0fd82f3a/support/rrsync>

為了減少容量及穩定性, 選用存活 16 年 perl 版本

!!! 如果改用 python 版本, 在 Dockerfile 加上安裝 python, 重新下載 rrsync



# 連線測試

測試連線指令

```
rsync --list-only -e "ssh -p 2223" admin@192.168.0.1:/
```

```
rsync -avz ./test/. admin@192.168.0.1:/test/
```