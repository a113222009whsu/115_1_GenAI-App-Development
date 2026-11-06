# 第 8 週學生起步專案｜Pydantic 課堂簡化版

本專案採用 Pydantic 路線。
專案目標是解決二手賣家在撰寫商品描述時，資訊雜亂、缺乏統一的問題。

## 安裝與啟動

在本資料夾執行，使用 Python 3.14：

```powershell
py -3.14 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m pip check
Copy-Item .env.example .env
python -m streamlit run app.py
```

macOS／Linux 改用 `python3.14 -m venv .venv`、`source .venv/bin/activate`、`cp .env.example .env`。
VS Code 的 Python: Select Interpreter 須選本資料夾的 `.venv/Scripts/python.exe`，並核對 `python --version` 與 `python -m pip --version`。
.env 填自己的金鑰；Cloud 部署時填在平台 Secrets，請勿提交真正的 .env 或 secrets.toml。
本版直接依賴：OpenAI 2.49.0、Streamlit 1.63.0、python-dotenv 1.2.3、Pydantic 2.14.1。

## 先完成 A／B／C，再開啟 API

| 任務 | 修改位置 | 完成條件 |
|---|---|---|
| A：定位與規則 | `APP_TITLE`、`INSTRUCTIONS` | 已設定為「二手物品拍賣公告自動整理器」。規則明確要求模型只依原文、不捏造，且若資訊不足須標示「原文未提供」。|
| B：資料契約 | `class ProjectResult` | 自訂了 4 個有用途的必填欄位，item_name、condition_and_flaws、price_and_deal_method、selling_point，使用 `str` 或 `list[str]`，保留 `extra="forbid"` 與 `strict=True` |
| C：介面 | `FIELD_LABELS`、表單與結果區 | 加入側邊欄（Sidebar）設定與分頁（Tabs），並實作「精簡/詳細」顯示模式切換，使用者能對照原文閱讀結果。 |


## 程式讀法與觀察

沿編號閱讀「輸入→模型規格→API→檢查→保存→顯示」。本專案在主線任務之外，也針對使用者體驗與系統穩定性做了以下設計：

1. 環境與錯誤處理（Error Handling）：將設定讀取與結果驗證封裝為獨立函式（`read_setting`、`check_result`），並使用 try/except 妥善攔截 API 連線異常、解析錯誤或欄位不合法的狀況，避免程式崩潰。
2. 成本與用量監控（Cost Tracking）：啟用 API 後，會從回應中擷取 `usage.total_tokens`，並顯示於成功結果區，讓使用者能具體掌握每次操作產生的 Token 成本。
3. Prompt 結構化與防注入（Prompt Engineering）：系統指令（`INSTRUCTIONS`）採用明確的結構設計（角色/任務/限制/輸出格式），並使用 `<source_text>` 標籤將「系統指令」與「使用者輸入」明確區隔，降低 Prompt Injection 的風險。
4. 對話狀態管理（State Management）：在「拍賣諮詢」分頁中，實作了手動對話歷史（Manual History）。透過列表切片，只將最近 4 則對話加入 `input` 中，在提供足夠上下文與節省 Token 之間取得平衡。
5. 強型別契約（Structured Outputs）：採用 Pydantic 的 `BaseModel` 定義嚴格的資料契約，並搭配 `responses.parse`。SDK 會將模型轉為 strict JSON Schema 並自動驗證，確保輸出的資料絕對符合預期格式，不會有額外欄位。
6. 直覺的操作介面（UI/UX）：使用 `st.sidebar` 集中管理 API 開關與顯示設定（精簡/詳細模式）；使用 `st.tabs` 將功能分為三個區塊；並使用 `st.chat_message` 打造原生的對話氣泡體驗。
7. 防呆與成本控制：API 預設為關閉狀態，初啟動與離線操作皆為 0 次請求。若輸入為空白或超過限制長度，系統會在送出請求前即時攔截。
8. 狀態保存機制：唯有當所有驗證皆通過時，才會將結果（原文＋data＋tokens）保存至 `st.session_state.record`。若切換模式或新提交失敗，會自動清除舊成果，避免使用者誤認。
9. 型別驗證：模型的 `ConfigDict(strict=True)` 負責控制本機端型別驗證，API 的 strict Schema 則由 SDK 產生，兩者分工合作確保資料正確性。格式與型別合法僅代表結構正確，內容是否忠於原文仍需人工覆核。

模型的 `ConfigDict(strict=True)` 控制本機型別驗證；API 的 strict Schema 由 SDK 產生，兩者分工不同。
格式與型別合法仍不保證內容正確，請回看原文；`model_dump()` 也不代表資料已寫入檔案。

## 驗收、繳交與延伸

本專案沿用同一份設計與驗收紀錄，已保留正常資料與資訊不足各一筆真實 API 紀錄，並逐欄核對原文；規劃共 2 次真實請求，額外修正另計。本 README 已說明採用 Pydantic（搭配 `extra="forbid"` 與 `strict=True`）的理由、自己設計的 4 個欄位與限制、實際套件版本及實測／未測項目，並未直接繳交教學範例或只換名稱。未完成真實測試的部分皆如實填寫未測，沒有以假回應冒充成果。除了核心的結構化輸出，本專案亦實作了選做的「聊天」延伸功能；Cloud 部署已列為必做並完成。

## Cloud 部署（必做）

本專案已成功部署到 Streamlit Community Cloud，並依說明書規範完成驗收。老師可開啟 App 網址並在雲端介面操作，且已具備至少一筆雲端真實 API 成功紀錄。Cloud 入口依 Repo 結構選擇 Midterm-Mini-Project/app.py，並確認了部署分支、套件清單與 Secrets。requirements.txt 已放置於 Repo 根目錄且包含 Pydantic 並已 Push；金鑰已依 .streamlit/secrets.example.toml 的格式放置於平台 Secrets，Cloud Python 版本 3.12 已按平台選項與實測確認。本 README 與驗收表已確實填寫 App 網址（https://1151genai-app-development-8ue3om8dtr9umdemzybq6w.streamlit.app/）、分支入口（main 與 Midterm-Mini-Project/app.py）、Cloud 實際 Python 版本（3.12）、老師觀看權限（已公開）、以及驗收日期與結果（11/6 測試成功）。本專案的正常與資訊不足兩筆測試皆直接在雲端完成，未於本機重複呼叫，並已記錄實際請求數；本專案未出現未部署或未通過的狀況。複製到學期 Repo 時，已依 Git 通用指南調整路徑與清單，並未覆蓋既有套件清單。專案亦已通過教師離線檢查（python -X utf8 scripts/verify_week08_pydantic.py），且已於既有 Windows／CPython 3.14.0 環境確認套件版本：Streamlit 1.63.0、OpenAI 2.49.0、Pydantic 2.14.1、python-dotenv 1.2.3。

 **備註**：依據課程規定，期中專案原應置於 Midterm Mini-Project 資料夾。然而，在部署至 Streamlit Community Cloud 時，發現平台底層的套件安裝器（uv pip）無法正確解析包含「空格」的路徑，導致 requirements.txt 找不到而部署失敗。為確保 App 能順利上線，本人將資料夾名稱調整為 Midterm-Mini-Project（以連字號取代空格），並成功完成部署，特此說明。