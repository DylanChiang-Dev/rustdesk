# RustDesk 自建與部署對話記憶

本文件記錄了關於 RustDesk 專案介紹、PVE 自建伺服器部署以及用戶端設定的相關討論與設定細節。

## 1. 專案背景與架構
*   **RustDesk** 是一個開源、跨平台、基於 Rust 與 Flutter 開發的遠端桌面軟體。
*   主打開箱即用，強調資料自主與隱私安全。
*   核心架構分為底層的影像傳輸 (hbb_common, scrap) 與上層的 UI (Flutter)。
*   完整介紹已存放在根目錄的 `Agent.md` 中。

## 2. 伺服器自建 (Self-Hosting) 需求
*   官方推薦使用 **Docker / Docker Compose** 來部署自建伺服器。
*   需要兩大核心元件：
    *   **`hbbs` (ID/Rendezvous Server)**: 處理 ID 註冊與 P2P 連線配對「打洞」。
    *   **`hbbr` (Relay Server)**: 當 P2P 打洞失敗時，作為資料中繼轉發站。

## 3. PVE 虛擬機部署參數建議 (個人輕量使用)
*   **CPU**: 1 vCPU (Type 建議選 `host`)
*   **記憶體**: 1024 MiB (最低可至 512 MiB 但 1GB 較穩)
*   **硬碟**: 10 GiB (Write through 快取)
*   **網路**: VirtIO 模型
*   **OS**: 建立虛擬機 (VM) 並安裝 Debian 12 或 Ubuntu Server 22.04 尤佳（比 LXC 坑少）。
*   詳細步驟已存放在 `PVE_Deploy.md`。

## 4. 網路與防火牆 (關鍵)
*   **必須開放的 Ports**：
    *   TCP: 21115, 21116, 21117, 21118, (21119 Web端)
    *   UDP: 21116 (重要，用於分配 ID)

## 5. 實際部署紀錄 (IP: 1.34.162.105)
*   透過 SSH 連入 PVE 虛擬機。
*   建立了 `/opt/rustdesk/docker-compose.yml`。
*   成功啟動 `hbbs` 與 `hbbr` 容器。
*   取得伺服器連線公鑰 (Public Key): `X1Uv5UtxjqWxU3O1WhWpDbKm62efcdPdzkB1c28cyak=`

## 6. 客戶端 (Client) 設定須知
*   **網路設定填寫**：
    *   ID 伺服器: `1.34.162.105`
    *   Key: `X1Uv5...ak=`
    *   中繼伺服器、API 伺服器皆留空。
*   **進階選項**：
    *   使用 WebSocket: 關閉
    *   允許回退 TLS: 關閉
    *   禁用 UDP: **務必關閉** (關閉代表允許 UDP，有助於 P2P 直連)。
*   **帳戶/登入**：
    *   開源免費版 (OSS) 不需要也無法登入。此功能主要是留給 Pro 版或自建 API Server 用的。
    *   只要 IP 和 Key 填對，就能形成專屬的私有連線圈。
*   **密碼安全性**：
    *   連線密碼只存在於**被控端電腦本機**。
    *   即使流量經過 VPS中繼 (`hbbr`)，資料也是透過 Key 進行端到端加密，VPS 伺服器無法得知密碼與傳輸內容。
*   **開機自啟動設定**：
    *   Windows: 無需在中尋找打勾選項。只要點擊主畫面上的 **「安裝 RustDesk」**，將其註冊為 Windows 系統服務，就會自動在開機輸入密碼前背景常駐執行。
    *   Linux/macOS: 正常安裝即會註冊為系統或登入服務。
