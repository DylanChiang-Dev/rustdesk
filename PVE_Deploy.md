---
description: 如何在 PVE 建立並設定 RustDesk 伺服器
---

# PVE 上自建 RustDesk 伺服器完整指南

本指南說明如何在 Proxmox Virtual Environment (PVE) 上建立一台專屬於個人使用的 RustDesk 伺服器 (包含 hbbs 與 hbbr)，硬體需求非常低。

## 1. 虛擬機 (VM / LXC) 資源配置建議

由於您只有**個人使用**（連接數極少），RustDesk 伺服器對硬體的要求其實非常輕量，所以配置可以給得很低：

- **CPU**: 1 vCPU (核心數 1 即可，平時使用率通常不到 1%)
- **記憶體 (RAM)**: 512 MB ~ 1 GB (Docker 容器啟動後大約只佔用不到 100 MB)
- **硬碟 (Storage)**: 8GB ~ 10GB 即可 (主要是為了裝 OS 跟 Docker 工具，RustDesk 本身產生的 Log 跟 Key 資料夾佔用極小)
- **網路 (Network)**: 需要可以直接對外的網路環境，或者您的 Router / 防火牆可以將指定的 Port 轉發 (Port Forwarding) 到這台 PVE 虛擬機的內部 IP。

### 選擇 VM 還是 LXC？
- **LXC (Linux Container)**: 比較省資源（甚至 512MB RAM 就夠了），開機快。但如果在 LXC 跑 Docker，有時會遇到權限或檔案系統 (AppArmor / OverlayFS) 的坑，需要額外在 PVE 設定檔勾選 `nesting=1` 和 `keyctl=1`。
- **VM (Virtual Machine)**: 最沒有坑，完全獨立的系統環境。**強烈建議新手開 VM**，安裝 Ubuntu Server 22.04 或 Debian 12。

---

## 2. 網路與防火牆設定 (關鍵)

您的 PVE 可能是放在家裡的區網（例如取得 `192.168.x.x` IP）。因此，您需要在**家裡的路由器 (Router)** 上設定 **Port Forwarding (通訊埠轉發)**，把來自外部公網 IP 的特定 Port，指向這台 PVE 虛擬機的內部 IP。

**必須開放/轉發的 Ports：**
- **TCP**: `21115`, `21116`, `21117`, `21118` (這四個都要)
- **UDP**: `21116` (這個一定要 UDP)

*(註：21119 主要是給網頁版客戶端用的，如果不用網頁版遠端可以不開。)*

---

## 3. 系統基本安裝步驟 (以 Debian/Ubuntu 為例)

連入您剛建好的 PVE 虛擬機後，執行以下安裝流程：

### 步驟 A: 系統更新與安裝 Docker
```bash
# 更新套件清單並升級
apt update && apt upgrade -y

# 安裝必備工具
apt install -y curl wget git

# 使用官方指令碼自動安裝 Docker
curl -fsSL https://get.docker.com | bash

# 將當前使用者加入 docker 群組 (若非 root)
# usermod -aG docker $USER
```

### 步驟 B: 設定 RustDesk Docker Compose
```bash
# 建立專案資料夾
mkdir -p /opt/rustdesk
cd /opt/rustdesk

# 建立 docker-compose 檔案
cat << 'EOF' > docker-compose.yml
version: '3'

networks:
  rustdesk-net:
    external: false

services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: rustdesk/rustdesk-server:latest
    # 注意：將 <您的公網IP或域名> 替換成您家裡的對外 IP 或 DDNS 域名！
    command: hbbs -r <您的公網IP或域名>:21117 
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped

  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: rustdesk/rustdesk-server:latest
    command: hbbr
    volumes:
      - ./data:/root
    networks:
      - rustdesk-net
    restart: unless-stopped
EOF
```

> **非常重要**：上面檔案中的 `command: hbbs -r <您的公網IP或域名>:21117`，裡面的 `<您的公網IP或域名>` 請務必換成您家中的浮動/固定對外 IP，或設定好的 DDNS 網域名稱。

### 步驟 C: 啟動並獲取連線 Key
```bash
# 啟動容器 (背景執行)
docker compose up -d

# 查看產生的連線公鑰 (設定客戶端時會用到)
cat ./data/id_ed25519.pub
```

## 4. 用戶端連線設定

在需要被控制的電腦及您用來控制的手機/電腦上設定：
1. 開啟 RustDesk -> 設定 -> 網路 -> 解鎖網路設定
2. **ID 伺服器**: 填寫您剛剛設定的 `<您的公網IP或域名>`
3. **Key**: 貼上剛才從 `id_ed25519.pub` 獲得的那一串字元
4. 儲存後，狀態列顯示「就緒」即代表設定成功，全部流量都會走您家裡的這台 PVE VM 了！
