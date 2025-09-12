---
title: open bmc

---

# OPEN BMC 

## 1.  Basic concept
### YOCTO
> Yocto Project 是一個 **開源協作專案**，用來建立 **嵌入式 Linux 系統** 的工具和架構。  
它本身 **不是一個 Linux 發行版**，而是一組工具和框架，可以讓你為自己的硬體客製化 Linux 映像檔。

- **BitBake**：Yocto 的建構工具（類似 make/cmake，但功能更強），會依照 recipes（配方）來編譯。
- **Recipes (.bb 檔案)**：描述如何下載原始碼、編譯、安裝軟體的腳本。
-   **Layers**：模組化的結構，把 recipes 分類管理（例如：核心、網路堆疊、BSP）。
    
-   **Poky**：Yocto 的參考發行版，內含 BitBake 與基本的 recipes。
    
-   **Machine/Board Support Package (BSP)**：針對特定硬體的設定。

### IPMI
> IPMI = Intelligent Platform Management Interface， 是一套由 Intel 等公司提出的 開放標準（最早 1998 年），主要用來 **遠端管理伺服器**，即使伺服器 **沒開機、沒 OS** 也能操作。通常由 **BMC 韌體** 實作，透過 **LAN、串口、KCS (Keyboard Controller Style) 接口** 進行管理。

-   **遠端電源控制**：開機、關機、重開機。
    
-   **監控硬體感測器**：溫度、電壓、風扇轉速、電流。
    
-   **Event Log (SEL, System Event Log)**：記錄硬體錯誤、告警。
    
-   **FRU 資訊 (Field Replaceable Unit)**：像是主機板、電源的序號/型號。
    
-   **Console Redirection (Serial-over-LAN, SOL)**：可以遠端連到主機的 console。

### Redfish
> Redfish由分散式管理工作組(DMTF)開發，代表著伺服器管理的現代化方法，該技術具有降低複雜性提高擴展性

-   **Service Root** 所有資源都從這裡開始。
        
-   **Systems (`/redfish/v1/Systems`)** 主機系統 (CPU/Memory/Disk/NIC)。       
-   **Chassis (`/redfish/v1/Chassis`)** 機箱、電源、散熱。
            
-   **Managers (`/redfish/v1/Managers`)** BMC 自己。


![Screenshot 2025-09-04 153635](https://hackmd.io/_uploads/BJjsW6Icxe.png)

| 項目  | **Redfish** | **IPMI** |
| --- | --- | --- |
| **出現時間** | 2015 年，由 DMTF 制定 | 1998 年，由 Intel 主導 |
| **協定型式** | RESTful API over HTTP/HTTPS (JSON) | 二進位封包協定，通常走 UDP 623 / Serial |
| **資料格式** | JSON，結構化清楚，容易解析 | 固定長度的二進位欄位，擴充困難 |
| **安全性** | HTTPS、TLS、OAuth Token、Role-based Access | 明碼密碼傳輸（舊版），安全性較差 |
| **擴充能力** | 高，可依 Schema 增加新物件/欄位 | 低，Command Set 固定 |
| **操作方式** | `curl` / REST client / 網頁 GUI | `ipmitool` / IPMI client |
| **主要用途** | 現代資料中心管理，雲端整合，跨平台標準 | 傳統伺服器遠端管理，硬體相容性 |
| **OpenBMC 支援** | `bmcweb` 提供 `/redfish/v1` API | `ipmid` + `bmcweb` IPMI handler 提供 |
| **狀態** | **新標準，強力推廣中** | **舊標準，但仍有大量舊系統依賴** |


### Linux Kernel
> Linux Kernel 是 **作業系統核心**，主要負責：**硬體抽象**：提供 driver 讓軟體不用直接操作硬體。**資源管理**：CPU、記憶體、I/O、網路。**系統呼叫(syscall)**：提供 API 給 user space 程式呼叫。

-   **Process Management (排程器、系統呼叫、thread)**
    
-   **Memory Management (分頁、虛擬記憶體、cache、OOM)**
    
-   **File System (ext4, tmpfs, procfs, sysfs)**
    
-   **Device Drivers (I2C, SPI, GPIO, UART, PCIe, USB, Ethernet, etc.)**
    
-   **Networking Stack (TCP/IP, socket)**
    
-   **Architecture Support (ARM, x86, RISC-V)**
    
### Host Power Control
> 有兩個package負責控制與監控電源狀態。

- **Phosphor-state-manage** : 每一個操作(on/off/reset)需要有單獨的systemd，例如：obmc-chassis-poweroff@.target、obmc-chassis-poweron@.target等。

-   **x86-power-control** : 主要針對Intel平台的電源控制，支援標準的操作(on/off/reset)，也支援Redfish。他是利用libgpiod來實作，意思是需要在Kernel(Device Tree)中加入對應的GPIO設定，如果使用的名稱與標準的不一樣，可以透過配置檔將名稱傳入。預設使用的名稱為：POWER\_OUT、RESET\_OUT等。支援多個host，並且可以有個別的配置檔。

### FRU 
> FRU其實就是一個Flash Memory，用來保存這個板子的資訊，包含Product Name、Part Number、Serial Number等。

有兩個package與這個功能有關：

- ipmi-fru-parser(phosphor-ipmi-fru)是用來修改讀取FRU的EEPROM路徑。
    
-  Entity Manager透過配置檔將一個FRU定義為一個一個fru-device的Entity，因此叫做Entity Manager。

他會將所有可讀取的EEPROM列出來，並且提供讀取操作，支援8bit與16bit的device，所有FRU device有提供D-Bus介面可以存取。

### Sensors
- phosphor-hwmon透過sensor配置檔指定每個sensor讀取的方式。
    
- dbus-sensors有不同的daemon來提供不同類型的sensor，例如：hwmon、fan、cpu、ipmb、與psu等。

### Fan Control

phosphor-pid-control有兩種配置方法

-  以Json檔各別設定每個sensor與風扇轉速之間的控制關係。
    
-  透過Entity Manager需要設定enable-configure-dbus來指定使用Entity Manager，再自行加入風扇控制的設定檔。

### DBus
每個 DBus 物件由三個部份組成：
Service
-  `xyz.openbmc_project.State.Host`

Object Path (物件路徑)
-    `/xyz/openbmc_project/state/host0`

Interface (介面)
    
-   定義這個物件有哪些功能 (方法、屬性、訊號)
-   例如：`xyz.openbmc_project.State.Host` 提供 `CurrentHostState` 屬性

### DTS
> Device Tree Source (.dts / .dtsi)一個文字檔，用來描述硬體結構編譯後變成 DTB Device Tree Blob，在 BMC 開機時由 bootloader 傳給 Kernel 依照 DTS 的描述，載入對應的 **driver**，建立 `/sys` 節點，讓上層程式能存取硬體。

#### 節點語法速查
```clike=
label: node-name@unit-address {
compatible = "vendor,chip"; // 必填：對應 Linux 驅動 binding 名稱
reg = <addr [size]>; // I²C/SPI/Memory-mapped 位址與大小（依匯流排）
status = "okay"; // "okay" 啟用；"disabled" 停用


// 依需求：
interrupts = <IRQ ...>; // 中斷（需 parent 的 interrupt-controller）
clocks = <&clk X>; clock-names = "..."; // 時脈來源
resets = <&rst Y>; // 重置腳位/控制器
gpios = <&gpioX N flags>; // 或具名：xxx-gpios = <...>
};
```

#### 匯流排父節點（I²C/SPI/PCI 等）
```clike=
&i2c2 {
status = "okay";
clock-frequency = <400000>;


fanctrl@2e {
compatible = "smsc,emc2305"; // 例：EMC2305 風扇控制器
reg = <0x2e>; // I²C 位址
// 選配：present GPIO、重置 GPIO 等
fan-present-gpios = <&gpio2 13 GPIO_ACTIVE_LOW>;
};
};
```
![Screenshot 2025-09-12 104735](https://hackmd.io/_uploads/Bka15ZZogg.png)




## 2. Architecture

![3965317016](https://hackmd.io/_uploads/HyqSCqCcle.png)

```clike=
......

executable(
    'entity-manager',
    'entity_manager.cpp',
    'expression.cpp',
    'perform_scan.cpp',
    'perform_probe.cpp',
    'overlay.cpp',
    'topology.cpp',
    'utils.cpp',
......
)

......
    executable(
        'fru-device',
        'expression.cpp',
        'fru_device.cpp',
        'utils.cpp',
        'fru_utils.cpp',
        'fru_reader.cpp',
......
    )
```
可以看到，entity-manager 虽然是一个包，但却会构建两个可执行文件，一个是 entity-manager ，另一个是 fru-device 。这两个可执行文件分别是由 xyz.openbmc_project.EntityManager.service 和 xyz.openbmc_project.FruDevice.service 独立负责启动的，两者在启动时并没有直接的依赖关系。

从上面我们的架构分析可知，这两者，一个是“配置管理器”，另一个则是“探测器”。说句实在话，把 fru-device 放在这个包里其实并没有那么合适，倒是有种作为“探测器” demo 的感觉，它完全可以像 peci-pcie 一样自己起一个包。
### FRU Device
> fru-device 負責在沒有設備驅動的情況下發現 i2c 總線上所有可能存在的 FRU ，讀取解析 FRU 的內容並將其掛到 dbus 上。讀取的過程可能有一些約定的依賴性，比如設備必須能夠直接響應不帶有 command 的 i2c read 。同时，这个读取过程是低效且可能超时的，其默认的超时处理是将超时的总线直接加入黑名单以避免下一次扫描。但是这样的过程提供了非常不错的灵活性，设备所在的总线和地址不再需要写死了，可以在扫描的过程中动态生成。

#### After Scan
`rescanBusses()` 会创建一个 回调。這個回調會在掃描完成后被呼叫，可以看到，掃描完成后，它就 來將掃描結果掛到 dbus 上去了：`FindDevicesWithCallbackaddFruObjectToDbus()`
src/fru_device.cpp
```clike=
int main()
{
    ......
    // run the initial scan
    rescanBusses(busMap, dbusInterfaceMap, unknownBusObjectCount, powerIsOn,
                 objServer, systemBus);
......
}

void rescanBusses(
    BusMap& busmap,
    boost::container::flat_map<
        std::pair<size_t, size_t>,
        std::shared_ptr<sdbusplus::asio::dbus_interface>>& dbusInterfaceMap,
    size_t& unknownBusObjectCount, const bool& powerIsOn,
    sdbusplus::asio::object_server& objServer,
    std::shared_ptr<sdbusplus::asio::connection>& systemBus)
{
    ......
        ......

        auto scan = std::make_shared<FindDevicesWithCallback>(
            i2cBuses, busmap, powerIsOn, objServer, [&]() {
                ......
                for (auto& devicemap : busmap)
                {
                    for (auto& device : *devicemap.second)
                    {
                        addFruObjectToDbus(device.second, dbusInterfaceMap,
                                           devicemap.first, device.first,
                                           unknownBusObjectCount, powerIsOn,
                                           objServer, systemBus);
                    }
                }
            }); 
        scan->run();
    ......
}
```

#### Scan
> 接下来，我们主要聚焦于扫描的过程， 最终会调用 ，我们直接来看这个函数：`scan->run()findI2CDevices()`

src/fru_device.cpp
```clike=
static void findI2CDevices(const std::vector<fs::path>& i2cBuses,
                           BusMap& busmap, const bool& powerIsOn,
                           sdbusplus::asio::object_server& objServer)
{
    for (const auto& i2cBus : i2cBuses)
    {
        int bus = busStrToInt(i2cBus.string());

......

        auto& device = busmap[bus];
        device = std::make_shared<DeviceMap>();
......

        // fd is closed in this function in case the bus locks up
        getBusFRUs(file, 0x03, 0x77, bus, device, powerIsOn, objServer);

......
    }
}
```
在上方的这个函数中，它会遍历所有存在的 i2c 总线，（检测这些总线的特性，比如 和 ，上面的代码里省略掉了 ），最后调用 来扫描这一条总线。`I2C_FUNC_SMBUS_READ_BYTEI2C_FUNC_SMBUS_READ_I2C_BLOCKgetBusFRUs()`

传入的参数， 代表扫描的起始地址， 代表扫描的结束地址。
在這裡， device 以引用的方式傳遞，是掃描結果的輸出。device 以引用的方式被綁定到 busmap ，其整體結構是這樣的：0x03 0x77

在明白了重要參數的含義後，我們進一步看看 是如何獲取各個设备的 FRU 的：`getBusFRUs()`

src/fru_device.cpp
```clike=
int getBusFRUs(int file, int first, int last, int bus,
               std::shared_ptr<DeviceMap> devices, const bool& powerIsOn,
               sdbusplus::asio::object_server& objServer)
{

    std::future<int> future = std::async(std::launch::async, [&]() {
        ......
        // Scan for i2c eeproms loaded on this bus.
        std::set<size_t> skipList = findI2CEeproms(bus, devices);
        ......

        for (int ii = first; ii <= last; ii++)
        {
            ......
            if (skipList.find(ii) != skipList.end())
            {
                continue;
            }
            ......
            // Set slave address
            if (ioctl(file, I2C_SLAVE, ii) < 0)
            {
                std::cerr << "device at bus " << bus << " address " << ii
                          << " busy\n";
                continue;
            }
            // probe
            if (i2c_smbus_read_byte(file) < 0)
            {
                continue;
            }
            ......
            makeProbeInterface(bus, ii, objServer);
            ......
            auto readFunc = [is16BitBool, file, ii](off_t offset, size_t length,
                                                    uint8_t* outbuf) {
                return readData(is16BitBool, false, file, ii, offset, length,
                                outbuf);
            };
            FRUReader reader(std::move(readFunc));
            std::string errorMessage = "bus " + std::to_string(bus) +
                                       " address " + std::to_string(ii);
            std::pair<std::vector<uint8_t>, bool> pair =
                readFRUContents(reader, errorMessage);
            ......
            devices->emplace(ii, pair.first);
        }
        return 1;
    });
    ......
}
```
#### 解析与推送
> 接下来就是 FRU 的解析過程了，FRU 的解析發生在往 dbus 上推送對應 object 的时候。
接着上面的 中的 來看：FindDevicesWithCallbackaddFruObjectToDbus()

src/fru_device.cpp
```clike=
void addFruObjectToDbus(
    std::vector<uint8_t>& device,
    boost::container::flat_map<
        std::pair<size_t, size_t>,
        std::shared_ptr<sdbusplus::asio::dbus_interface>>& dbusInterfaceMap,
    uint32_t bus, uint32_t address, size_t& unknownBusObjectCount,
    const bool& powerIsOn, sdbusplus::asio::object_server& objServer,
    std::shared_ptr<sdbusplus::asio::connection>& systemBus)
{
    boost::container::flat_map<std::string, std::string> formattedFRU;

    std::optional<std::string> optionalProductName = getProductName(
        device, formattedFRU, bus, address, unknownBusObjectCount);
    ......

    std::string productName = "/xyz/openbmc_project/FruDevice/" +
                              optionalProductName.value();

    ......

    std::shared_ptr<sdbusplus::asio::dbus_interface> iface =
        objServer.add_interface(productName, "xyz.openbmc_project.FruDevice");
    dbusInterfaceMap[std::pair<size_t, size_t>(bus, address)] = iface;

    for (auto& property : formattedFRU)
    {
        ......

        if (property.first == "PRODUCT_ASSET_TAG")
        {
            std::string propertyName = property.first;
            iface->register_property(
                key, property.second + '\0',
                [bus, address, propertyName, &dbusInterfaceMap,
                 &unknownBusObjectCount, &powerIsOn, &objServer,
                 &systemBus](const std::string& req, std::string& resp) {
                ......
                });
        }
        else if (!iface->register_property(key, property.second + '\0'))
        {
            ......
        }
        ......
    }

    // baseboard will be 0, 0
    iface->register_property("BUS", bus);
    iface->register_property("ADDRESS", address);

    iface->initialize();
}
```


### Entity-manager
> 這個可執行文件负责根据 fru-device 等“探测器”挂在 dbus 上的内容，将本地的 json 与之进行匹配，选出那些存在的设备，再去完整的初始化这些设备（安装驱动），最后，将这些可用的设备所对应的 json 配置进行一个组合，合并形成 system.json 放置在 。最後，entity-manager 將這個 json 的條目推上 dbus 供其它服務使用。


#### 事件驅動架構
- 背景
    fru-device 負責 I²C 掃描並把 FRU 資料丟到 DBus。
    entity-manager 負責把 DBus 上的物件統整成系統 Inventory。

- 問題
    entity-manager 怎麼知道 fru-device 已經把物件建立在 DBus 上？
    不能靠輪詢 (polling)，效率太差。

- 解法
    採用 事件驅動架構 → 由 DBus 發送 signal (事件)，entity-manager 收到通知再處理。

:::success 
EntityManager 不直接依賴 FruDevice，而是靠 DBus 事件 來決定何時進行統整。
:::

#### 事件的監聽

- DBus 本質
    建立在 Linux IPC (Inter-Process Communication) 基礎上，本質就是 Unix socket 通訊。
    事件的傳遞其實就是「socket 上有數據」。

- 監聽方式
    訊號通知 (signal)
    阻塞/非阻塞 I/O
    select/poll/epoll 等事件驅動 I/O

- EntityManager 的做法
    在 DBus 上註冊 listener。
    當 FruDevice 發布 FRU 物件時，EntityManager 就會被 DBus 喚起 → 進入 callback。
    
:::success
EntityManager 並不是主動查 DBus，而是「被事件拉起來」。
:::

#### 事件的處理
- 基本原則
    收到事件後，必須有一個 loop 來依序處理。
    常見模式：事件隊列 + 單線程事件迴圈。

- 程式模型
假想版：
```clike=
run() {
    while (true) {
        if (API 收到一個事件) {
            // 處理事件
        }
    }
}
```
現實版：
    把事件放進 queue，由單一執行緒依序消化。
    
- EntityManager 的實作
    使用與 Android Looper / Windows message queue 類似的機制。
    DBus 事件進來 → 進 queue → 單執行緒依序處理。
:::success
事件驅動讓系統 即時、可靠、可擴充，避免輪詢浪費資源。
:::

```clike
[硬體層]
FRU EEPROM (I²C bus, 工廠燒錄 FRU 資料)
    │
    ▼
Device Tree (DTS → DTB)  //Where is EEPROM
  - 描述 bus/address/driver
  - 例如: eeprom@50 { compatible="atmel,24c64"; reg=<0x50>; }
    │
    ▼
Kernel Driver (ex: at24)
  - 掛載後在 sysfs 出現 /sys/bus/i2c/devices/.../eeprom
    │
    ▼
getBusFRUs()
  - 掃描位址 0x03–0x77
  - 從 sysfs 嘗試讀取 FRU 資料

[掃描管理層]
    │
    ▼
findI2CDevices()
  - 呼叫 getBusFRUs for 每個 bus
  - 整理成 busmap
    │
    ▼
busmap (資料結構)
  bus_id → { addr → DeviceInfo }

[掛載層]
    │
    ▼
rescanBusses()
  - 遍歷 busmap
  - addFruObjectToDbus()
    │
    ▼
D-Bus FruDevice 物件
  + 發送事件通知 (InterfacesAdded/Removed)

[管理層 - EntityManager]
    │
    ▼
讀取 JSON 配置
  - Probe 條件判斷 (BOARD_PRODUCT_NAME, BUS/ADDR…)
  - 配對/命名物件 (psu0, fan0, dimm0…)
  - 輸出 system.json
    │
    ▼
D-Bus Inventory 物件
  (/xyz/openbmc_project/inventory/... )

[對外層]
    │
    ▼
Redfish / IPMI
  - 查詢 Inventory
  - 回傳 JSON / CLI 結果
```