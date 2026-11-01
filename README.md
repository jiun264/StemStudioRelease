# StemStudio

**Windows 桌面音訊分軌工具**：分離人聲、吉他、貝斯、鼓與鋼琴，或移除指定樂器後將其餘音訊合成單一 WAV；也可貼上公開音訊網址，下載後加入處理佇列。

[下載 Windows x64 安裝程式](https://github.com/jiun264/StemStudioRelease/releases/download/v0.2.0/StemStudio-Setup.exe) · [查看 v0.2.0 發布資訊](https://github.com/jiun264/StemStudioRelease/releases/tag/v0.2.0)

> 這是作品集與安裝程式下載頁。**原始碼未公開，也不包含在此 repository 或 Release 中。**

## 專案簡介

StemStudio 是一個以 WPF 建構的 Windows 桌面應用程式，將音訊匯入、來源分離和結果輸出串成單一工作流程。使用者可拖放／選取音訊檔，或貼上支援的公開 HTTP(S) 網址；可分別輸出人聲、吉他、貝斯、鼓、鋼琴，或選擇一種樂器移除並輸出其餘音訊的混音檔。

## 功能

- 支援 WAV、MP3、FLAC、M4A、OGG、WMA、AAC 輸入格式。
- 以 Demucs `htdemucs_6s` 模型進行人聲、吉他、貝斯、鼓、鋼琴來源分離，輸出 WAV。
- 可選擇移除單一樂器，將其餘來源合成一個 WAV 檔。
- 透過 yt-dlp 取得支援來源的音訊，並由 FFmpeg 轉成 MP3。
- 支援多檔案佇列、狀態回報及取消工作。
- 隨安裝包提供私有 CPython runtime 與 CPU 版 PyTorch，使用者不必另外安裝 Python 或 CUDA。
- 模型權重不內附於安裝程式；首次使用時由應用程式提示並下載至使用者選定的快取位置。

## 技術架構

| 層 | 技術 | 職責 |
| --- | --- | --- |
| 桌面 UI | C#、WPF、.NET 8 | 檔案匯入、佇列、音軌選擇、設定與狀態呈現 |
| 應用核心 | .NET 類別庫 | 分離請求／結果、音軌型別及服務介面 |
| 程序整合 | .NET Worker | 驗證參數並啟動隨附 Python 子程序，轉送狀態與錯誤 |
| 音訊處理 | Python、Demucs、PyTorch CPU | 模型準備與來源分離 |
| 網址匯入 | yt-dlp、FFmpeg | 取得音訊並轉成 MP3 |

```text
WPF App → .NET Infrastructure → .NET Worker → Python worker
                                                ├─ yt-dlp → FFmpeg → MP3
                                                └─ Demucs + PyTorch → WAV stems / mix
```

## 工程設計

- **分層與程序隔離：** UI 不直接載入 Python／PyTorch；Core 介面隔離應用流程與基礎設施實作。
- **明確的程序契約：** 以分離的命令列參數傳遞路徑和工作選項，避免 shell 字串拼接及路徑空白造成的 quoting 問題。
- **可診斷的錯誤：** Worker 轉送 Python 輸出與錯誤，並以結束碼和預期輸出檔共同判定工作是否成功。
- **取消與暫存：** 取消工作時停止 Worker 子程序；中間檔使用暫存目錄，成功後再保留所選輸出。
- **使用者資料與安裝分離：** 模型快取、設定和輸出放在使用者資料位置，不寫入安裝目錄。

## 驗證範圍與限制

- .NET App 建置通過，零警告、零錯誤。
- 曾以安裝包建置目錄中的 Python runtime，直接呼叫 worker 測試實際網址下載及 MP3 轉換；這是 Python worker 單獨測試，不代表所有來源、網路環境或完整 UI 流程均已驗證。
- 分軌品質會受混音、音色、演奏方式與輸入品質影響；分離結果不保證與原始錄音完全一致。
- 此版本提供音訊分離與網址匯入，不包含吉他 TAB 自動轉譜功能。

## 下載與驗證

- **版本：** 0.2.0
- **平台：** Windows x64
- **安裝檔：** [`StemStudio-Setup.exe`](https://github.com/jiun264/StemStudioRelease/releases/download/v0.1.0/StemStudio-Setup.exe)
- **SHA-256：** `0F61C6F545EF63D484B0F468E9B64ACB7DA57BAE4DB540568D1A5D7F475B3BEA`

下載後可在 PowerShell 驗證檔案：

```powershell
Get-FileHash .\StemStudio-Setup.exe -Algorithm SHA256
```

## 負責任使用與第三方元件

請只下載你擁有或獲授權、且來源允許下載的內容。本程式不提供 DRM 或登入限制繞過功能。

安裝程式包含 Demucs、PyTorch、yt-dlp、FFmpeg 等第三方元件。各元件依其上游授權條款提供；公開散布前應檢視並遵守各元件的授權、署名、聲明及適用的原始碼提供要求。模型權重不隨安裝包散布。
