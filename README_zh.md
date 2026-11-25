# TCL HDMI Launcher

<p align="left">
  <a href="README.md">English</a> | <b>繁體中文</b>
</p>

[![Latest Release](https://img.shields.io/github/v/release/ian20040409/TCL-Android-TV-HDMI-R851T02?style=flat-square&color=blue)](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/ian20040409/TCL-Android-TV-HDMI-R851T02/total?style=flat-square)](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases)
[![License](https://img.shields.io/github/license/ian20040409/TCL-Android-TV-HDMI-R851T02?style=flat-square)](LICENSE)

寫這個 Launcher 是因為受夠了 TCL 電視原廠那套又慢又一堆廣告的首頁。如果你平常幾乎只用外接設備（像是 Apple TV、PS5、Switch 或電視盒），這支 App 可以讓電視單純當作顯示器，開機直接進 HDMI，省去中間一堆干擾。

**[下載最新版 APK](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases/latest)**

---

## 主要功能

- **開機自動進 HDMI（或指定 App）**：開機或按 Home 鍵後會倒數幾秒自動切到預設的 HDMI 埠，倒數中按任意方向鍵就會取消。也可以改成開機直接開特定 App（例如 YouTube、Netflix），或停在應用程式列表。
- **原生 `TvView` 播放器**：直接走 Android 的 TV Input Framework（`TvView`）硬體直通，不依賴問題一堆的 `com.tcl.tv`。支援 4K @ 60Hz、HDR10 與 Dolby Vision 直通，沒有多餘浮層；另外可自訂是否顯示「訊號搜尋 HUD」以及無訊號自動休眠秒數。
- **來源感知 CEC 判斷**：透過讀取 `HdmiCecController` 日誌來判斷休眠訊號的來源，只有「目前正在看的 HDMI 埠」發出關機時才會連動關閉電視。這樣 Switch 放底座自己休眠時，就不會害你在看 Apple TV 看到一半被關機；同時也會擋掉 TCL 韌體 `com.tcl.voicestandby` 造成的異常關機迴圈。
- **遙控器快捷切源**：在首頁或播放畫面中，按數字鍵 `1`、`2`、`3` 可直接跳到對應的 HDMI 埠，按 `INPUT` / `SOURCE` 鍵則依序循環切換 HDMI 1–3；若開啟「按鍵映射」，在其他 App 按下輸入源鍵也會直接切回 HDMI Launcher，不會跳出原廠訊號源選單。
- **TV 應用程式庫**：針對遙控器設計的卡片式列表，支援分類篩選（全部 / 最近 / TV 版 / 手機版 / 系統 / 已凍結）。若有搭配 Shizuku，長按即可直接凍結或解凍不想看到的系統 App。
- **Shizuku 一鍵設定**：設定裡內建 [Shizuku TV 版](https://github.com/thedjchi/Shizuku/releases) 下載器。授權後會自動幫你開好 `READ_LOGS` 權限跟「喚醒守護」無障礙服務，不用在電視設定選單裡翻半天。
- **純黑 (`#000000`) 介面與 Leanback 設定**：適合暗室觀看，內建原生電視設定選單、按鍵映射 (Button Mapper)、一鍵開啟 TCL 畫質/音效選單 (`com.tcl.settings`)，以及即時 CEC 事件檢視器方便除錯。

---

## 畫面截圖

| 主畫面（訊號源切換） | 倒數秒數設定 | TCL 畫質設定捷徑 | 應用程式列表 |
|:---:|:---:|:---:|:---:|
| ![Main](readme_pic/Screenshot_20260925_222244.png) | ![Countdown](readme_pic/Screenshot_20260925_222304.png) | ![Settings](readme_pic/Screenshot_20260925_222322.png) | ![Apps](readme_pic/Screenshot_20260925_222344.png) |

---

## 安裝與設定

### 1. 安裝 APK
到 [Releases](https://github.com/ian20040409/TCL-Android-TV-HDMI-R851T02/releases/latest) 下載 APK，透過 ADB 安裝到電視：
```bash
adb connect <電視IP>:5555
adb install -r app-release.apk
```

### 2. 設為預設桌面
```bash
adb shell cmd package set-home-activity com.lnu.tclhdmilauncher/.MainActivity
```

### 3. 停用原廠桌面（建議）
避免原廠桌面在背景偷跑或自己跳出來：
```bash
adb shell pm disable-user --user 0 com.google.android.tvlauncher
adb shell am force-stop com.google.android.tvlauncher
```
*(隨時可用 `adb shell pm enable com.google.android.tvlauncher` 恢復)*

### 4. 開啟喚醒守護與 CEC 自動切換

「喚醒守護」（無障礙服務）用來處理待機黑畫面與 Home 鍵攔截，`READ_LOGS` 則是讓 App 能讀取系統的 CEC 訊號切換事件。

- **透過 Shizuku 自動設定（推薦）**  
  如果電視上已經有跑 [Shizuku（建議用 thedjchi 的 TV 版）](https://github.com/thedjchi/Shizuku/releases)（或直接在 Launcher 設定裡下載安裝），進設定授權 Shizuku 後，它就會自動授予 `READ_LOGS` 並開啟無障礙服務。

- **手動透過 ADB 設定**  
  1. 授予 `READ_LOGS` 並重啟 App：
     ```bash
     adb shell pm grant com.lnu.tclhdmilauncher android.permission.READ_LOGS
     adb shell am force-stop com.lnu.tclhdmilauncher
     ```
  2. 開啟無障礙服務（也可直接點狀態列上的「喚醒守護」手動開）：
     ```bash
     adb shell settings put secure enabled_accessibility_services com.lnu.tclhdmilauncher/.WakeAccessibilityService
     ```

---

## 遙控器操作

| 畫面 | 按鍵 | 功能 |
|---|---|---|
| **主畫面 / 播放器** | `1` / `2` / `3` | 直接切到 HDMI 1 / 2 / 3 |
| **主畫面 / 播放器** | `INPUT` / `SOURCE` | 依序循環切換 HDMI 1 -> HDMI 2 -> HDMI 3 |
| **其他 App** | `INPUT` / `SOURCE` | 攔截原廠訊號源選單並返回 HDMI Launcher（需開啟按鍵映射） |
| **主畫面** | 方向鍵 (D-Pad) | 移動焦點；倒數期間按任意方向鍵可取消自動跳轉 |
| **主畫面** | `OK` / 確認鍵 | 進入選取的 HDMI 埠或 App（在 HDMI 卡片上長按可設為預設） |
| **主畫面** | `SETTINGS` / `MENU` | 開啟 Launcher 設定 |
| **HDMI 畫面** | `SETTINGS` | 呼叫 TCL 原廠畫質 / 音效設定 (`com.tcl.settings`) |
| **HDMI 畫面** | `BACK` / `MENU` | 退出 HDMI 畫面，回到首頁 |
| **應用程式庫** | `OK` / 確認鍵 | 開啟 App |
| **應用程式庫** | `MENU` / 長按 `OK` | 開啟選單（App 資訊、解除安裝、透過 Shizuku 凍結/解凍） |

---

## 實測環境與訊號源 ID

目前在 **TCL 65C715** 上實測正常（平台：RTD2851 / R851T02，韌體：`V8-R851T02-LF1V662`，Android 9）：

| 訊號來源 | 端口 | TvInput ID |
|---|---|---|
| HDMI 1 | 1 | `com.tcl.tvinput/.passthroughinput.TvPassThroughService/HW1413744128` |
| HDMI 2 | 2 | `com.tcl.tvinput/.passthroughinput.TvPassThroughService/HW1413744384` |
| HDMI 3 | 3 | `com.tcl.tvinput/.passthroughinput.TvPassThroughService/HW1413744640` |

---

## 權限與隱私說明

所有功能都只在電視本機端執行：
- **`READ_LOGS`**：僅在記憶體中過濾 `HdmiCecController` 的日誌來判斷 CEC 訊號源，不會儲存或上傳任何資料。
- **無障礙服務 (`WakeAccessibilityService`)**：僅用來監聽視窗切換與 Home 鍵事件以防止待機假死，不會記錄按鍵內容或個人隱私。
- **Shizuku**：只用於本機執行授權、開啟無障礙及凍結 App 等指令，沒有任何對外連線。

---

## 技術文件與排查筆記

想了解底層踩坑紀錄或查詢 ADB 指令可以參考 `docs/` 裡的筆記：

- **[問題排查筆記 (docs/troubleshooting_zh.md)](docs/troubleshooting_zh.md)** ([English](docs/troubleshooting.md)) — 包含 Apple TV 雙重休眠循環關機（卸載 `com.tcl.tv`）、Dolby Vision 橫幅卡死、`TvView` 黑屏 / Realtek HWC 殘影，以及 CEC 日誌監聽實作。
- **[ADB 指令速查表 (docs/adb-cheatsheet.md)](docs/adb-cheatsheet.md)** — 這台電視常用的 ADB 指令整理。
- **[韌體與相容性說明 (docs/firmware-guide.md)](docs/firmware-guide.md)** — Android 9 V662 降級與 Android 11 狀態紀錄。
