# UCLLIU TSF Bridge

這是肥米輸入法的 TSF commit bridge。C# 版 repo 會保留 Python 版 `UclTsfBridge` C++ 原始碼，讓 TSF Bridge 可以和主程式一起追蹤、重建與打包。

目標不是把肥米邏輯搬進 TSF DLL，而是提供一個 Windows TSF Text Service，讓 C# 主程式透過 named pipe 嘗試走 TSF commit。失敗時仍回到 Unicode `SendInput` / 剪貼簿 / Big5 流程。

## Build

需要先安裝：

- Microsoft Visual Studio 2022 Build Tools 或 Visual Studio Community
- 「使用 C++ 的桌面開發」工作負載
- MSBuild
- MSVC C++ x64/x86 build tools
- Windows 10/11 SDK

在專案根目錄直接執行：

```bat
build_tsf.bat
```

這會編譯 `Release|x64` 與 `Release|Win32`，成功後複製下列檔案到 `tsf_bridge`、`bin\Debug\tsf_bridge` 與 `bin\Release\tsf_bridge`：

```text
x64\UclTsfBridge.dll
x86\UclTsfBridge.dll
register_tsf_bridge.bat
unregister_tsf_bridge.bat
unlock_tsf_bridge.ps1
```

使用 Visual Studio Developer PowerShell：

```powershell
msbuild .\UclTsfBridge\UclTsfBridge.vcxproj /p:Configuration=Release /p:Platform=x64
```

`build_tsf.bat` 會依序尋找 Visual Studio 2026/2022 與 `vswhere`，並用可用 toolset 覆寫 `PlatformToolset`。

Release 使用 `/MT` 靜態連結 C/C++ runtime，降低使用者端需要額外安裝 VC Runtime 的機率。

## Register

```powershell
.\register_tsf_bridge.bat
```

註冊後到 Windows 輸入法清單切換 `UCLLIU TSF Bridge`。

## Pipe API

Pipe 名稱：

```text
\\.\pipe\uclliu_tsf_bridge
```

Request:

```json
{"cmd":"status"}
{"cmd":"commit_text","text":"肥"}
```

Response:

```json
{"ok":true,"active":true,"has_context":true}
{"ok":true}
{"ok":false,"error":"COMMIT_FAILED","hr":"0x8007139F"}
```

## 注意

- 目前只提供 commit bridge，不處理候選窗、不攔字根、不顯示 composition。
- C# 端必須保留 fallback，因為未切換到 TSF Bridge 或 context 消失時一定會失敗。
- 現行註冊腳本使用系統 regsvr32，請以系統管理員權限執行；肥米右下角選單會協助導引。
- 若 DLL 正被瀏覽器、Explorer、Codex、Notepad++ 等程序載入，`build_tsf.bat` 可能無法覆蓋 `bin\Debug\tsf_bridge`，請切離 TSF Bridge 或關閉相關程序後重跑。
