# 最後紀元：第 5 賽季繁體中文與粉圓字型修補

這是 [aianlinb/LETraditionalChinese](https://github.com/aianlinb/LETraditionalChinese) 的非官方延伸版本，補上 Steam 第 5 賽季的繁體文字與字型修補。原作者的翻譯、工具與粉圓字型資產仍是本專案的基礎。

**已測試環境：Windows x64、Steam 遊戲版本 1.5.2（第 5 賽季）、Steam build 25764382、Unity 6000.4.8f1。** 其他遊戲版本尚未驗證；官方更新若更換資產結構、檔名或語系內容，需要重新檢查。

## 使用方式

1. 關閉遊戲，從 [Releases](https://github.com/PadeAdmin/LETraditionalChinese/releases) 下載本 fork 的發布包並完整解壓縮。
2. 保留 `LETraditionalChinese.exe` 旁的 `LETraditionalChinese` 資料夾；裡面有字典、字型與材質資料。不要把資料夾換成舊版同名 ZIP：新增的 bundle 修補需要資料夾形式。
3. 雙擊 EXE，自動尋找 Steam 遊戲位置；找不到時用 PowerShell 指定**遊戲安裝根目錄**：

```powershell
.\LETraditionalChinese.exe "E:\SteamLibrary\steamapps\common\Last Epoch"
```

4. 遊戲內選擇「簡體中文」語言，會顯示修補後的繁體文字。

程式會直接替換遊戲資產。要還原，在 Steam 對遊戲執行「驗證遊戲檔案的完整性」。遊戲更新後先確認本版是否適用，再重新套用。

發布包不需要另外安裝 .NET。從原始碼編譯則需要 .NET 10 SDK。

### 唯讀檢查

```powershell
.\LETraditionalChinese.exe --check "E:\SteamLibrary\steamapps\common\Last Epoch"
```

檢查主資產與 `PermaLoad.bundle` 中預期的粉圓字型槽位；這不是完整的文字覆蓋或畫面驗證。

## 會修改哪些遊戲檔案？

以下路徑都相對於遊戲安裝根目錄。**使用者不需要手動編輯這些檔案**，由工具處理。

| 檔案 | 用途 |
| --- | --- |
| `Last Epoch_Data/StreamingAssets/aa/StandaloneWindows64/localization-string-tables-chinese(simplified)(zh)_assets_all.bundle` | 以字典替換簡體語系文字 |
| `Last Epoch_Data/StreamingAssets/aa/catalog.bin` | 處理語系 bundle 的 CRC 資訊 |
| `Last Epoch_Data/resources.assets` | 注入字型、字型圖集、材質與 fallback 設定 |
| `Last Epoch_Data/sharedassets0.assets` | 更新字型引用與字型 fallback |
| `Last Epoch_Data/StreamingAssets/LEAssetBundles/assets_a3ec63478769f648.bundle` | 替換介面使用的 NotoSansSC／NotoSansTC 動態字型資產 |
| `Last Epoch_Data/StreamingAssets/LEAssetBundles/PermaLoad.bundle` | 修補物品詳細說明、任務等常駐介面字型 |

`GameAssembly.dll` 與 IL2CPP metadata 僅供解析型別時讀取，不會修改；也不修改遊戲 EXE 或存檔。一般字型與 bundle 修補不建立還原備份；沿用的 catalog 程式在部分格式分支仍可能建立 `catalog.bin.bak`。還原方式以 Steam 驗證為準。

## 開發者：要修改哪些原始碼？

| 原始碼／資料 | 調整位置 |
| --- | --- |
| `Program.cs` | Steam 偵測、遊戲執行中檢查、語系與三階段字型修補、`--check` |
| `LETraditionalChinese/dictionary.json` | 翻譯對照表；修改用語時編輯此檔 |
| `LETraditionalChinese/fonts/`、`LETraditionalChinese/manifest.json` | 沿用原作者的字型與圖集資料；更換字型不能只換 TTF，需同步對應資料 |
| `tools/LEFontPatch/LEFontManager.cs` | `resources.assets`／`sharedassets0.assets`，字型引用與 fallback 正規化 |
| `tools/LEFontPatch/BundleFontPatch.cs` | 指定介面 bundle 的動態字型、圖集與材質；版本變動時先查 bundle 檔名與資產 |
| `tools/LEFontPatch/PermaFontPatch.cs` | `PermaLoad.bundle` 中的字型、fallback、材質 padding／GradientScale |
| `tools/LEFontPatch/Program.cs` | 字型修補工具入口 |
| `tools/LELocalePatch/Program.cs`、`Catalog.cs` | 語系 bundle 的文字替換與 catalog CRC 處理 |
| `LETraditionalChinese.csproj`、`tools/*/*.csproj` | 建置與來源專案引用 |
| `deps/AssetsTools.NET` | 固定版 submodule；其 Cpp2IL 專案改用新版 LibCpp2IL 來源與 context API |
| `deps/Cpp2IL` | 固定 upstream commit 的 submodule，用於目前 IL2CPP metadata 解析 |

`tools/LEFontPatch` 與 `tools/LELocalePatch` 收錄各自原作者程式的修改版與授權。舊版預編譯 `lib/` DLL 已不參與建置。

### 取得與編譯

```powershell
git clone --recurse-submodules --branch season5-support https://github.com/PadeAdmin/LETraditionalChinese.git
cd LETraditionalChinese
dotnet publish .\LETraditionalChinese.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:TargetFrameworks=net10.0 -p:DebugType=embedded -o .\artifacts\publish
Copy-Item .\licenses .\artifacts\publish\licenses -Recurse -Force
Copy-Item .\README.md .\artifacts\publish\README.md
Compress-Archive -Path .\artifacts\publish\* -DestinationPath .\artifacts\LETraditionalChinese-Season5.zip -Force
```

已有 checkout 時先執行 `git submodule update --init --recursive`。請使用儲存庫固定的 submodule commit；不要直接更新到各依賴的最新分支。`TargetFrameworks=net10.0` 用於限制依賴的多目標建置。

### 翻譯與字型說明

- 原作者字典來自 Azure AI Translator，並經原作者程式及部分人工調整。本版保留此基礎，新增／整理第 5 賽季文字，使用 OpenCC `s2tw` 做繁體轉換；新增內容並非全面人工校對。
- 字典包含本次語系的對照與已轉換文字的自身映射。官方新增且未收進字典的文字，仍可能顯示簡體。
- 以原作者粉圓體（jf-openhuninn 2.1）資產為主，統一中文 fallback，修正原本仍使用襯線字的介面。
- 動態來源字型補入本次文字需要的 5 個 Noto Sans TC 字形：U+5DCB、U+7341、U+8855、U+885A、U+9B9E。修改後字型內部名稱為 `LE Chinese Rounded`；資產名稱保留原有形式以維持工具相容。

### 已完成驗證

- 建置及 Windows x64 自包含發布。
- 本次 23 個語系表共 66,443 筆資料的輸出對照，確認條目 ID 保留。
- 修補 bundle 重新開啟，比對預期替換資產及未修改資產內容。
- `--check` 驗證主字型及常駐 bundle 字型槽位。
- 使用者實機確認地圖、物品詳細說明及任務介面顯示正常。

## 授權與來源

主程式與修補工具依原專案 MIT 授權；粉圓體與 Noto Sans TC 字形依 SIL Open Font License。各專案及字型授權見 `licenses/`，submodule 亦保留原始授權。發布包必須包含相關授權文件。

- [LETraditionalChinese](https://github.com/aianlinb/LETraditionalChinese)
- [LEFontPatch](https://github.com/aianlinb/LEFontPatch)
- [LELocalePatch](https://github.com/aianlinb/LELocalePatch)
- [AssetsTools.NET](https://github.com/nesrak1/AssetsTools.NET)
- [Cpp2IL](https://github.com/SamboyCoding/Cpp2IL)
- [粉圓體](https://github.com/justfont/open-huninn-font)
- [Noto Sans TC](https://github.com/google/fonts/tree/main/ofl/notosanstc)

本儲存庫與發布包不包含遊戲原始資產。問題請在本 fork 回報，避免將尚未合併的延伸版本問題直接報給原作者。
