# OpenBMC Development Notes (QEMU)

這個 repo 用來整理我在 **QEMU 模擬環境** 下實作 **OpenBMC** 的紀錄。  
包含：BitBake 編譯、DBus 測試、Redfish API 操作，以及 Redfish ↔ DBus 的對照分析。  
同時也補充 HackMD 筆記，涵蓋 FRU、Entity-Manager、Host Power Control 與整體架構。

---

## 📂 檔案結構與內容

- **01_bitbake_qemu.md**  
  - 使用 **Yocto / BitBake** 編譯 `obmc-phosphor-image`  
  - 在 `qemuarm` 目標板上透過 `runqemu` 啟動  
  - 記錄必要套件、編譯流程與啟動輸出結果

- **02_dbus_qemu.md**  
  - 進入 QEMU 模擬的 BMC 後，利用 `busctl` 學習 **DBus 服務 / 物件 / 屬性** 並透過 `obmctuil` 做快速的查詢  
  - 包含：
    - `busctl list/tree/introspect` 查詢服務與物件與介面  
    - `busctl monitor` 監聽事件  
    - `obmcutil state` 讀取所有狀態  
    - `busctl get-property \ ` 讀取物件的屬性

- **03_redfish_qemu.md**  
  - 使用 `curl` 測試 **Redfish API** 介面  
  - 觀察系統 JSON 回應內容（PowerState, Boot, BIOS, Console 等）  
  - 嘗試透過 `ComputerSystem.Reset` API 進行開機/關機操作

- **04_redfish_dbus_qemu.md**  
  - 驗證 **Redfish 與 DBus 對照結果**  
  - 範例：
    - Redfish `.PowerState` ↔ DBus `CurrentPowerState`  
    - Redfish `LogServices/Journal` ↔ DBus `xyz.openbmc_project.Logging.DeleteAll`  
  - 展示兩種介面查詢一致性的測試

- **openbmc-concepts-hackmd.md**  
  - 來源：[HackMD 筆記](https://hackmd.io/@-dj2hMxRT2-aleBNa4Xpvg/B1db0OIclg)  
  - 整理 OpenBMC 基本概念與實驗流程，包括：  
    - **Yocto / BitBake** 架構  
    - **Host Power Control**：Redfish/IPMI → DBus → State Manager → GPIO → Host Power Good  
    - **FRU / Inventory**：Fru-Device 掃描 EEPROM → Entity-Manager 整合 JSON → D-Bus → Redfish/IPMI 查詢  
    - **Sensors**：PMBus/I²C → DBus → Redfish/IPMI  
    - **DTS Kernel & Entity-Manager & Fru-Device 架構理解**
