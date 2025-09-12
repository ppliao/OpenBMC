# OpenBMC QEMU 實驗與學習筆記

這個 repo 用來整理我在 **QEMU 模擬環境** 下學習與驗證 **OpenBMC** 的紀錄。  
包含：BitBake 編譯、DBus 測試、Redfish API 操作，以及 Redfish ↔ DBus 的對照分析。  
同時也補充 HackMD 筆記，涵蓋 FRU、Entity-Manager、Host Power Control 與整體架構。

---

## 📂 檔案結構與內容

- **01_bitbake_qemu.md**  
  - 使用 **Yocto / BitBake** 編譯 `obmc-phosphor-image`  
  - 在 `qemuarm` 目標板上透過 `runqemu` 啟動  
  - 記錄必要套件、編譯流程與啟動輸出結果:contentReference[oaicite:0]{index=0}

- **02_dbus_qemu.md**  
  - 進入 QEMU 模擬的 BMC 後，利用 `busctl` 學習 **DBus 服務 / 物件 / 屬性**  
  - 包含：
    - `busctl list/tree/introspect` 查詢服務與介面  
    - `busctl monitor` 監聽事件  
    - `obmcutil poweron` 驅動狀態變化  
    - `busctl get-property` 讀取屬性:contentReference[oaicite:1]{index=1}

- **03_redfish_qemu.md**  
  - 使用 `curl` 測試 **Redfish API** 介面  
  - 觀察系統 JSON 回應內容（PowerState, Boot, BIOS, Console 等）  
  - 嘗試透過 `ComputerSystem.Reset` API 進行開機/關機操作:contentReference[oaicite:2]{index=2}

- **04_redfish_dbus_qemu.md**  
  - 驗證 **Redfish 與 DBus 對照結果**  
  - 範例：
    - Redfish `.PowerState` ↔ DBus `CurrentPowerState`  
    - Redfish `LogServices/Journal` ↔ DBus `xyz.openbmc_project.Logging.DeleteAll`  
  - 展示兩種介面查詢一致性的測試:contentReference[oaicite:3]{index=3}

- **openbmc-concepts-hackmd.md**  
  - 來源：[HackMD 筆記](https://hackmd.io/@-dj2hMxRT2-aleBNa4Xpvg/B1db0OIclg)  
  - 整理 OpenBMC 基本概念與實驗流程，包括：  
    - **Yocto / BitBake** 架構  
    - **Host Power Control**：Redfish/IPMI → DBus → State Manager → GPIO → Host Power Good  
    - **FRU / Inventory**：Fru-Device 掃描 EEPROM → Entity-Manager 整合 JSON → D-Bus → Redfish/IPMI 查詢  
    - **Sensors**：PMBus/I²C → DBus → Redfish/IPMI  
    - **Entity-Manager & Fru-Device 架構**：事件驅動與 DBus Signal 溝通，而不是輪詢  

---

HACKMD 筆記 https://hackmd.io/AnsTJ2VdTgCgPiQ8bMGpxg?view
