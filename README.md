# Fujitsu-LIFEBOOK-U939-B-Hackintosh
Fujitsu LIFEBOOK U939/B (i5-8265U / UHD 620)黑蘋果macOS Sequoia 15 EFI &amp; 安裝教學  macOS Sequoia Hackintosh EFI &amp; Installation Guide
# 💻 Fujitsu LIFEBOOK U939/B 黑蘋果 (macOS Sequoia) 安裝教學

> **機型規格**：Intel Core i5-8265U / UHD 620 / Realtek ALC255 / Intel Wi-Fi & BT
> **安裝亮點**：OpCore-Simplify + UnPlugged 離線完整安裝 (免網路卡)

---

## 📖 黑蘋果離線安裝教學

這是一份結合 **OpCore-Simplify-GUI**（自動化 OpenCore EFI 生成工具）與 **UnPlugged**（離線版 macOS 安裝輔助腳本）的黑蘋果安裝教學。

傳統的黑蘋果安裝需要手動下載各式驅動、手動修改複雜的 config.plist，且在沒有 Mac 的情況下很難製作出「完整版」的離線安裝碟（通常只能下載約 3GB 的線上恢復版，安裝時極度依賴網卡驅動與網速）。

### 🌟 這套方案的優勢
1. **EFI 簡單做**：利用 OpCore-Simplify-GUI，在 Windows 或 Mac 下用圖形介面就能一鍵生成適合你硬體的 OpenCore EFI。
2. **免網卡離線安裝**：利用 **UnPlugged** 搭配 gibMacOS 下載的 12GB+ 完整版安裝包（`InstallAssistant.pkg`），直接在 Recovery 恢復模式下本機解包安裝，完美解決「安裝時網卡沒驅動、進不去系統下載」的痛點。

---

## 📀 準備工具與硬體需求

* **USB 隨身碟**：16GB 或以上。
* **目標安裝硬碟**：請確保該硬碟上沒有重要資料，安裝時會格式化。
* **必要軟體下載**：
  * [Python](https://www.python.org/downloads/)（協助工具啟動，安裝時切記勾選加入 PATH）
  * [OpCore-Simplify-GUI](https://github.com/payne-huang/OpCore-Simplify-GUI)（圖形版 EFI 生成工具）
  * [UnPlugged](https://github.com/corpnewt/UnPlugged)（離線安裝腳本 `UnPlugged.command`）
  * [gibMacOS](https://github.com/corpnewt/gibMacOS)（下載完整 macOS 安裝包）
  * [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg/tree/master)（使用 Utilities/macrecovery 下載恢復鏡像）
  *
[GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)（生成序號用） 

---

## 🚀 第一階段：製作 OpenCore EFI 啟動檔案

1. **偵測硬體並下載工具**：在 Windows 下執行 `OpCore-Simplify.exe`，切換到工具功能表，執行內建的硬體檢測工具，匯出報告。
2. **設定硬體與系統版本**：選擇 macOS 版本（如 Sonoma 14 / Sequoia 15），勾選「僅使用 Intel 核顯」。
3. **自動生成 ACPI 補丁與驅動 (Kexts)**：工具會自動勾選 ACPI 補丁（如 FakeEC, PLUG 等）與必備驅動，並自動匹配最佳 SMBIOS 機型。
4. **構建並匯出 EFI**：點擊「建構 EFI」，工具會下載最新版 OpenCorePkg 並在本機生成 `EFI` 資料夾。

## ⚠️ 重要提示：請務必自行生成 SMBIOS 機型序號！
為了保護個人 Apple ID 與帳號安全，本教學所提供的EFI文件的 `config.plist` **已抹除個人專屬的三碼資訊**。

**在正式開機與登入 Apple ID 前，請務必按照以下步驟填入你自己的序號：**

1. 使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 或 OpenCore Configurator 工具。
2. 選擇適當的機型（`MacBookPro15,2`）。
3. 生成全新的三碼，並填入 `config.plist` 的 `PlatformInfo -> Generic` 欄位：
   * `MLB` (Board Serial)
   * `SystemSerialNumber` (Serial Number)
   * `SystemUUID` (SmUUID)
4. **切勿直接使用他人分享的序號**，否則可能導致 iCloud、iMessage、FaceTime 被封鎖或帳號遭停用！

---

## 📦 第二階段：下載完整版 macOS 安裝檔與 Recovery

### ⚠️ macOS Sonoma / Sequoia 重要提示
由於 Sonoma/Sequoia 的 Recovery 環境預設不支援直接掛載 FAT32 或 ExFAT 磁區：
* **引導恢復鏡像**：請下載 **macOS Monterey 12** 的 `BaseSystem.dmg`（讀取 ExFAT 極順暢）。
* **完整安裝包**：下載全新的 macOS Sonoma 或 Sequoia。

1. **下載完整安裝包**：
   * 執行 `gibMacOS.bat`，輸入 `r` 切換成完整安裝版。
   * 下載目標 macOS 版本的 Full Install，完成後於 `gibMacOS/macOS Downloads/` 取得 `InstallAssistant.pkg`。
2. **下載引導恢復鏡像**：
   * 開啟終端機/命令提示字元，切換至 `OpenCorePkg/Utilities/macrecovery`，輸入：
     `py macrecovery.py -b Mac-FFE5EF870D7BA81A -m 00000000000000000 download`
   * 下載完成會產生 `com.apple.recovery.boot` 資料夾（內含 `BaseSystem.dmg` 與 `BaseSystem.chunklist`）。

---

## 💾 第三階段：分割並製作 USB 啟動碟

使用磁碟管理工具（如 DiskGenius）將 USB 分割為兩個區段：

| 磁區名稱 | 格式 | 大小 | 存放內容 |
| :--- | :--- | :--- | :--- |
| **OPENCORE** | **FAT32** | 約 1GB | `EFI` 資料夾、`com.apple.recovery.boot` 資料夾 |
| **UnPlugged** | **ExFAT** | 剩餘空間 | `InstallAssistant.pkg`、`UnPlugged.command` 腳本 |

> ⚠️ **配置單位提示**：ExFAT 格式化時，配置單位大小請設定為 **1024 KB 或更小**，否則 Recovery 可能無法掛載該磁區。

---

## ⚙️ 第四階段：調整 BIOS 設定

重開機進入 BIOS，調整以下設定：

* **必須關閉 (Disable)**：
  * Fast Boot (快速啟動)
  * Secure Boot (安全啟動)
  * SerialPort (序列埠)
  * VT-d (若無法關閉，可在 OpenCore 中開啟 `DisableIoMapper`)
  * CSM 傳統相容模式 (必須關閉以啟用純 UEFI)
* **必須開啟 (Enable)**：
  * VT-x (虛擬化技術)
  * Above 4G Decoding (大於 4G 解碼)
  * Hyper-Threading (超線程)
  * SATA Mode: **AHCI** (絕不能是 RAID)

儲存重開機，按下 Boot Menu 快捷鍵，選擇隨身碟 UEFI 模式啟動，進入 Recovery 恢復環境。

---

## 🛠️ 第五階段：使用 UnPlugged 進行離線安裝

1. **抹除目標硬碟**：進入 **Disk Utility (磁碟工具)** ➔ 點擊左上角「顯示所有裝置」➔ 選擇實體硬碟點擊「抹除 (Erase)」，格式選擇 APFS，完成後關閉磁碟工具。
2. **開啟終端機切換路徑**：點選上方選單列 `Utilities` ➔ `Terminal`，輸入：
   ```bash
   cd /Volumes/UnPlugged
3. **執行 UnPlugged 腳本**：
   ```bash
   ./UnPlugged.command
   # 若權限不足可改用：bash UnPlugged.command
4. **​解包並開始安裝**：選擇 **Fully expand InstallAssistant.pkg** ，接著選擇目標硬碟（如 Macintosh HD），輸入 y 開始執行。背景會解開安裝包並喚醒官方安裝程式。(安裝期間請勿關閉終端機)

---

## 🔄 結語與後續優化

**多次重啟**：安裝過程會重啟 3-5 次，每次請手動從 Boot Menu 選隨身碟，並在 OpenCore 選單選 macOS Installer 繼續。
轉移 EFI：進入桌面後，將隨身碟的 EFI 資料夾複製到本機硬碟的 ESP (EFI) 分割區。

**驅動微調**：若有舊款硬體，可下載 OpenCore Legacy Patcher (OCLP) 進行 Root Patch。

---

## 📋驅動順序與排錯

在 `config.plist` 裡的 `Kernel -> Add` 必須維持標準順序：

1. `Lilu.kext` (必須永遠排在第一個)
2. `VirtualSMC.kext`
3. `WhateverGreen.kext`
4. `AppleALC.kext`
5. `IntelMausi.kext`
6. `itlwm.kext`
7. `SMCBatteryManager.kext` / `SMCProcessor.kext` / `ECEnabler.kext`
8. `VoodooPS2Controller.kext`
9. `BrightnessKeys.kext`
10. `VoodooI2C.kext` ➔ `VoodooI2CHID.kext`
11. `IntelBluetoothFirmware.kext` ➔ `IntelBTPatcher.kext` ➔ `BlueToolFixup.kext`

> ⚠️ **重複 VoodooInput 提示**：ProperTree 清掃發出 Duplicate CFBundleIdentifiers 警告時，**請直接點 Yes** 停用重複模組。

---

## 🔍 config.plist 終極檢查清單

### 1. ACPI
* [x] **Add**：`SSDT-PLUG.aml` / `SSDT-EC-USBX.aml` / `SSDT-PNLF.aml`
* [x] **Quirks**：`FadtEnableReset` ➔ `False` | `ResetHwSig` ➔ `False` | `SyncTableIds` ➔ `False`

### 2. Booter -> Quirks
* [x] `AvoidRuntimeDefrag` ➔ `True`
* [x] `DevirtualiseMmio` ➔ `False`
* [x] `EnableSafeModeSlide` ➔ `True`
* [x] `RebuildAppleMemoryMap` ➔ `True` (Whiskey Lake 必開)
* [x] `SetupVirtualMap` ➔ `True`
* [x] `SyncRuntimePermissions` ➔ `True`

### 3. DeviceProperties -> Add
* **內顯 `PciRoot(0x0)/Pci(0x2,0x0)`**：
  * `AAPL,ig-platform-id` (`Data`) ➔ `00009B3E`
  * `device-id` (`Data`) ➔ `9B3E0000`
  * `framebuffer-patch-enable` (`Data`) ➔ `01000000`
  * `framebuffer-stolenmem` (`Data`) ➔ `00003001`
  * `framebuffer-fbmem` (`Data`) ➔ `00000002`
* **音效 `PciRoot(0x0)/Pci(0x1F,0x3)`**：
  * `layout-id` (`Data`) ➔ `03000000`

### 4. Kernel -> Quirks
* [x] `AppleCpuPmCfgLock` ➔ `False`
* [x] `AppleXcpmCfgLock` ➔ `True` (防止 CFG-Lock 卡死)
* [x] `DisableIOMapper` ➔ `True` (防止 VT-d 崩潰)
* [x] `XhciPortLimit` ➔ `False`

### 5. Misc
* **Boot**：`HideAuxiliary` ➔ `False` | `PickerMode` ➔ `Builtin` (排除圖形黑屏) | `ShowPicker` ➔ `True` | `Timeout` ➔ `5`
* **Security**：`AllowSetDefault` ➔ `True` | `DmgLoading` ➔ `Signed` | `SecureBootModel` ➔ `Disabled` | `Vault` ➔ `Optional`

### 6. NVRAM
* **7C436110-AB2A-4BBB-A880-FE41995C9F82**：
  * `boot-args` ➔ `-v keepsyms=1 debug=0x100 -igfxblr` *( -igfxblr 為 Whiskey Lake 抗黑屏參數 )*

### 7. UEFI
* **Drivers**：`OpenRuntime.efi` ➔ `True` | `OpenHfsPlus.efi` ➔ `True`
* **Output**：`ConsoleControl` ➔ `True` | `ProvideConsoleGop` ➔ `True` (解決選單無畫面) | `Resolution` ➔ `Max`
* **Quirks**：`RequestBootVarRouting` ➔ `True`

---

## 🚀 檢查完畢存檔四步驟
1. 按 `Cmd + Shift + R` (OC Clean Snapshot) 重新掃描 Kexts 與 Drivers。
2. 彈出重複驅動視窗點 **Yes**。
3. 按 `Cmd + S` 儲存 `config.plist`。
4. 重啟電腦，準備開機

---

# Fujitsu LifeBook U939/B RU.EFI 修改 BIOS DVMT 預分配顯存完整實作指南


安裝完成後，您因該會發現核顯顯存只有7mb，跟著這部分教學操作，可以一舉解決 Intel UHD 620 核顯在黑蘋果（macOS）中卡 7MB顯存、無法開啟硬體加速的難題。

---

## 🛠️ 第一部分：實作前準備與軟體下載

### 需要準備的工具與硬體：
1. **Windows 10/11 環境**
2. **空隨身碟**：1GB～32GB 即可，格式化為 **FAT32**。
3. **工具軟體包**：
   - [UEFITool](https://github.com/LongSoft/UEFITool/releases)
   - [IFRExtractor-RS](https://github.com/LongSoft/IFRExtractor-RS)
   - [Intel FPT](https://github.com/CE1CECL/IntelCSTools/releases/tag/v1.0.0)
   - [modGRUBShell.efi](https://github.com/zearp/OptiHack/blob/master/EFI/OC/Tools/modGRUBShell.efi)

---

## 📖 第二部分：核心步驟

### 步驟 1：導出 BIOS 文字檔並尋找 DVMT 的位址
1. **生成bios 韌體：**
   * 解壓縮 `Intel CSME System Tools v12`，​找裡面的  **FLASH PROGRAMMER TOOL** -> **WIN64** 資料夾，在 Windows 搜尋欄輸入 `cmd`， **右鍵** -> **以管理員身份運行** 
   * 輸入指令 **「cd WIN64資料夾路徑」** 定位到資料夾路徑後輸入 **「FPTW64.exe -bios -d u939_bios.bin」** 當出現 `FPT Operation Successful`代表已經成功生成
1. **開啟 UEFITool：**
   * 執行 `UEFITool.exe`，按下 **File** -> **Open image file**，選取你 U939/B 的 BIOS `.rom` 或 `.bin` 檔案。
2. **搜尋關鍵字：**
   * 按下快速鍵 `Ctrl + F`，切換到 **Text** 標籤頁。
   * 輸入 `DVMT Pre-Allocated`，勾選 **Header and body**，然後點擊 **OK**。
3. **定位並匯出 PE32 模組：**
   * 在下方搜尋結果視窗中，點兩下搜尋結果，主畫面會自動展開並高亮顯示目標模組（通常位於 Setup 或 Platform 結構下）。
   * 在高亮項目的 **PE32 image section** 上點擊**右鍵**，選擇 **Extract as is**，另存為 `Setup_PE32.sct`（或 `.bin`）。
4. **使用 Universal IFR Extractor 解碼：**
   * 開啟 `Universal IFR Extractor.exe`。
   * 在 Open File 選取剛才匯出的 `Setup_PE32.sct` 檔案。
   * 點擊 **Extract**，軟體會生成一份 `Setup_PE32 IFR.txt` 的文字檔。
5. **尋找關鍵 Offset：**
   * 用記事本開啟 `Setup_PE32 IFR.txt`，按下 `Ctrl + F` 搜尋 **DVMT Pre-Allocated**。
   * 你會看到類似下方的代碼（**請務必記下你電腦導出文字檔裡的實際數值**）：
     ```text
     0x1A2B3 One Of: DVMT Pre-Allocated, VarStoreInfo (VarOffset/VarName): 0x152, VarStore: 0x1, ...  
     0x1A2B8 One Of Option: 32M, Value (8 bit): 0x2  
     0x1A2BF One Of Option: 64M, Value (8 bit): 0x3  
     0x1A2C6 One Of Option: 16M, Value (8 bit): 0x1 (default)  
     ```
   📌 **筆記你查到的結果：**
   * **VarStore 名稱**：Setup（如果有 GUID，也順便記下）。
   * **VarOffset（暫存器位址）**：例如 **0x152**（此處僅為舉例，請以你的 `.txt` 為準！）。
   * **目標改寫數值**：把預設的 16M（`0x01`）改為 **32M（`0x02`）** 或 **64M（`0x03`）**。

### 步驟 2：製作 RU.EFI 開機隨身碟
1. 插入 FAT32 格式的隨身碟。
2. 在隨身碟根目錄下建立資料夾層級：`/EFI/BOOT/`。
3. 將下載好的 `RU.efi` 複製到 `/EFI/BOOT/` 資料夾內，並重新命名為 **`BOOTX64.EFI`**。
   * 完整路徑為：`隨身碟:\EFI\BOOT\BOOTX64.EFI`

### 步驟 3：進入 RU.EFI 修改 NVRAM 暫存器
1. **關閉安全啟動（Secure Boot）：**
   * 開機按 `F2`（或 `Fn + F2`）進入 U939/B 的 BIOS 介面。
   * 切換到 **Security** 標籤，將 **Secure Boot** 設為 **Disabled**。
   * 按 `F10` 儲存並重啟。
2. **引導隨身碟開機：**
   * 開機連續按 `F12` 進入選單，選擇你的 USB 隨身碟開機。
   * 畫面會進入 RU.EFI 的**藍色文字介面**。
3. **切換到 Setup 模組：**
   * 按下 **`Alt + =`**（或按 `F2` 打開選單）。
   * 使用方向鍵向下滾動，找到 **Setup**（如果是 Dell 機型通常叫 IntelSetup，富士通等日系機型叫 Setup）。按下 `Enter` 進入。
4. **定位至 VarOffset 位址：**
   * 假設你在步驟一查到的 VarOffset 是 **0x152**：
   * 畫面左側是列位址（例如 `00000150`），上方是行欄位（`00` 到 `0F`）。
   * 移動游標至 **150** 行的 **02** 欄位置（即 `0x152` 交會處）。
5. **改寫數值並儲存：**
   * 游標停在該格上，原本顯示的數值應該是預設的 `01`（對應 16MB）。
   * 直接鍵盤輸入 **`02`**（改為 32MB）或 **`03`**（改為 64MB）。
   * 確認數字更改後，按下 **`Ctrl + W`** 儲存變更（畫面上方會跳出成功寫入提示）。
   * 按下 **`Alt + Q`** 離開 RU.EFI 並重啟電腦。

### 步驟 4：調整黑蘋果 OpenCore config.plist
BIOS 設定修改完成後，你的主機板已經原生為 macOS 釋放了足夠的顯存。請開啟 OpenCore 的 `config.plist`：
1. 進入 `DeviceProperties` -> `PciRoot(0x0)/Pci(0x2,0x0)`。
2. **刪除所有舊的顯存欺騙補丁**（如果之前加過）：
   * 刪除 `framebuffer-stolenmem`
   * 刪除 `framebuffer-fbmem`
3. **僅保留基本的硬體加速參數**：
   * `AAPL,ig-platform-id` -> `<00009B3E>`（Data 型態）
   * `framebuffer-patch-enable` -> `<01000000>`（Data 型態）
4. 儲存 `config.plist` 並開機進 macOS，你的 Intel UHD 620 顯卡此時應該能成功載入 **1536 MB 顯存** 並開啟全硬體渲染！

---

## 🚨 第三部分：如果失敗/出錯的救援與解決方案

在 NVRAM 修改暫存器時，最怕改錯欄位或修改後導致電腦無法開機。以下是針對不同失敗狀況的應對措施：

### 狀況 A：改完數值後，重啟黑屏 / 警報聲 / 無法開機進 BIOS
不用慌張，這代表暫存器寫入的值超過 BIOS 接受範圍或改錯位址，導致 BIOS 檢驗失敗。請按以下步驟恢復：
1. **硬體級 RTC / CMOS 重置（最有效）：**
   * 長按電源鍵 10 秒強制關機，拔掉電源變壓器。
   * 拆開 U939/B 後蓋，**拔掉主電池連接線**。
   * 在主機板上找到 **CR2032 水銀鈕扣電池（CMOS 電池）**，將其拔下/斷開連接線。
   * 按住電源鍵 30 秒以放盡主機板殘餘電量。
   * 靜置 5 分鐘後，接回 RTC 電池與主電池，插上電源開機。
   * BIOS 會自動清空所有自訂暫存器並恢復出廠預設（Restore Factory Defaults）。
2. **利用富士通 Emergency Recovery（若有）：**
   * U939/B 具備 BIOS 保護機制，出廠重置後即可正常按 `F2` 重新進入 BIOS。

### 狀況 B：改完後依舊卡 7MB，或是進 macOS 直接「黑屏/無訊號」
如果修改了 DVMT，但進 macOS 依然卡 7MB 或直接黑屏，請按照順序排查：
1. **檢查 BIOS 是否自動重置：**
   * 再次進入 RU.EFI 查看 `0x152` 欄位，確認數值是否依然維持在 `02` 或 `03`。如果變回 `01`，說明你的 BIOS 在重啟時觸發了 NVRAM 保護並還原了數值。
   * **解法：** 改試 `02` (32MB) 而不是 `03` (64MB)；或者需要額外在 RU.EFI 裡將 **CFG Lock** 的 VarOffset 也改成 `00` (Disable)。
2. **解決顯卡黑屏（BusID 連接埠對不上）：**
   * 如果開機有聲音、鍵盤燈會亮，但螢幕黑屏，說明顯卡驅動成功載入，但 macOS 把信號發給了外接 HDMI 而是內建螢幕。
   * **解法：** 在 `config.plist` 的 `boot-args` 裡暫時加入 **`igfxonln=1`** 或 **`-no_compat_check`**；並加入 `framebuffer-con0-type` 指定內建 eDP 螢幕。

### 狀況 C：UEFITool / IFR Extractor 找不到 DVMT Pre-Allocated
1. **更換解包工具：**
   * 某些日系 BIOS 的語系文字被壓縮在 AMI/Insyde 的特殊 Setup 模組裡，可以改用 Universal BIOS Backup Toolkit 備份你目前的原生 BIOS，再用 AMISetupWriter 或 UEFITool NE (New Engine) 來開啟尋找。
2. **搜尋替代關鍵字：**
   * 嘗試搜尋 `DVMT`、`Stolen Memory` 或 `Graphics Memory`。

按上述流程操作，就有極大機會成功解放 Intel UHD 620 硬體加速


