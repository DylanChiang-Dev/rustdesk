# RustDesk 專案介紹

RustDesk 是一個使用 Rust 語言編寫的開源遠端桌面控制軟體。它主打開箱即用，無需繁雜配置，並且強調使用者擁有對自身資料的完全控制權，確保高度的安全性。

## 核心特色
- **開箱即用**：不需要特別配置即可進行遠端連線。
- **資料自主與安全**：可以選擇使用官方提供的中繼伺服器（Rendezvous/Relay Server），或是自行架設伺服器以實現完全的私人網路控制。
- **跨平台支援**：提供 Windows、macOS、Linux 甚至是行動裝置版本的支援。
- **現代化介面**：目前主要採用 Flutter 進行桌面端與行動端的圖形介面（GUI）開發（舊版的 Sciter 介面已廢棄）。

## 專案目錄結構解析
RustDesk 的架構包含了底層 Rust 實作的影像處理、網路通訊與系統控制，以及上層的圖形介面：

- **`libs/hbb_common`**：包含影像編碼、設定配置、TCP/UDP 封裝、Protobuf 定義、檔案系統操作（用於檔案傳輸）及其他核心工具函數。
- **`libs/scrap`**：負責螢幕畫面擷取（Screen capture）。
- **`libs/enigo`**：針對不同平台實作的硬體級鍵盤與滑鼠控制。
- **`libs/clipboard`**：跨平台（Windows, Linux, macOS）的剪貼簿同步實作，支援文字與檔案的複製貼上。
- **`src/server`**：提供音訊、剪貼簿、輸入控制與影像等服務，負責處理被控端的網路與事件。
- **`src/client.rs`**：負責啟動與遠端的點對點連接（Peer connection）。
- **`src/rendezvous_mediator.rs`**：負責與官方或自建的 Rendezvous 伺服器通訊，處理 NAT 穿透（TCP Hole punching）或退回使用 Relay 中繼網路。
- **`flutter/`**：包含目前主要的 Flutter 應用程式碼，涵蓋桌面端與行動端的 UI 實作。

總結來說，RustDesk 是一個強大且安全的遠端桌面替代方案，結合了 Rust 在底層的高效能與安全性，以及 Flutter 的跨平台靈活優勢。
