# OpenGame 研究報告

> 來源：[github.com/leigest519/OpenGame](https://github.com/leigest519/OpenGame)
> 日期：2026-05-10

---

## 一、專案概述

OpenGame 是香港中文大學 MMLab（Yilei Jiang, Xiangyu Yue 等）在 2026 年 4 月 21 日釋出的開源框架，定位為「從一句提示詞到可遊玩網頁遊戲」的端到端 AI Agent 開發工具。目前 GitHub 上已有 **1.4k stars**、**180 forks**。

核心賣點很直接：你輸入一段自然語言描述（比如「做一個貪吃蛇遊戲，WASD 控制，深色主題」），它會自動生成完整的 HTML5 專案結構、程式碼、資源，最後跑出一個能在瀏覽器裡玩的游戏。

## 二、技術架構

### 2.1 GameSkill —— Agent 的雙手

GameSkill 分成兩塊：

- **Template Skill**：根據需求挑選合適的引擎（Canvas、Phaser、three.js 等），生成專案骨架和目錄結構，讓後續編輯保持連貫。
- **Debug Skill**：在沙盒中執行遊戲，偵測控制台錯誤、互動斷裂等問題，系統性地修復直到遊戲可完整遊玩。

這套設計把 Agent 從「寫得出像樣的程式碼」推到「能交出一個真正能跑的東西」。

### 2.2 GameCoder-27B —— 專門訓練的代碼模型

這是 CUHK 團隊自己訓練的一個 270 億參數的代碼 LLM，訓練流程分三階段：

1. **持續預訓練（Continual Pre-training）**：用遊戲開發相關的程式碼資料繼續訓練基礎模型。
2. **監督微調（SFT）**：使用精心整理的遊戲開發軌跡數據，涵蓋引擎 API、專案搭建和除錯流程。
3. **執行導向的強化學習（RL with execution grounding）**：以實際遊戲可玩性作為獎勵信號，用 OpenGame-Bench 風格的驗證器來評分。

簡單說就是：先學語言 → 再學遊戲開發 → 最後在「能不能跑」這個現實反饋裡練出來。

### 2.3 OpenGame-Bench —— 動態評估基準

跟傳統靜態代碼評測不同，OpenGame-Bench 會實際啟動生成的遊戲，用腳本驅動互動行為，然後驗證：

- 渲染是否正常
- 控制是否回應
- 遊戲循環能否推進
- 勝負狀態是否正確

這比檢查語法錯誤難得多——因為「能跑」和「好玩」之間差了一大段距離。

## 三、展示作品

專案 README 列了六個由單一提示詞生成的完整遊戲：

| 遊戲名稱 | 類型 | 關鍵特色 |
|---------|------|---------|
| Marvel Avengers: Infinity Strike | 橫向卷軸動作平台 | 三位英雄、三個關卡、Thanos Boss、90s Capcom 像素風格 |
| Harry Potter: Arithmancy Academy | 回合制卡牌對戰 | 答題施法、魔力共振連擊系統、哥德奇幻風格 |
| K.O.F: Celestial Showdown | 雙人搶答格鬥 | 物理題目搶答、90s SNK 復古格鬥手感 |
| Hajimi Defense: The Tuna Crisis | 塔防 | 貓咪炮塔、笑話風美術、可愛手繪風格 |
| StarWars: Mandalorian Protocol | 頂視角動作 RPG 射擊 | Twin-Stick Shooter、鎧甲人突入帝國基地 |
| Squid Game: Red Light, Green Light | 生存反應遊戲 | 紅綠燈玩法、屍體堆積效果、寫實像素風格 |

每個作品都有線上 Demo 和原始碼下載。跑起來很簡單：`unzip → npm install → npm run dev`。

## 四、使用方式

### 基本流程

```bash
# 安裝（需要 Node.js 20+）
git clone https://github.com/leigest519/OpenGame.git
cd OpenGame
npm install && npm run build && npm link

# 用一句提示詞生成遊戲
opengame -p "Build a Snake clone with WASD controls and a dark theme." --yolo
```

### 認證與 API Key

- 主 Agent LLM：支援 OpenAI-compatible API，可設定 `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`OPENAI_MODEL`。本地跑的話可以換成 GameCoder-27B。
- 資源生成工具（圖片、影片、音訊）：各自獨立配置，帶入對應 Provider Key（tongyi / doubao / openai-compat）。

### 除錯模式

預設頭部模式下 Agent 只能編輯檔案，不能跑 Shell 指令。加 `--yolo`（或 `--approval-mode yolo`）才開放指令執行權限。

## 五、專案底層與依賴

OpenGame 不是從零開始：

- **Agent 運行時**：基於 [qwen-code](https://github.com/QwenLM/qwen-code)，在 CLI 架構上疊加 Game Skill 和 GameCoder-27B 整合。
- **CLI 藍圖**：qwen-code 本身又源自 [Google Gemini CLI](https://github.com/google-gemini/gemini-cli)。
- **遊戲渲染**：使用 [Phaser](https://github.com/phaserjs/phaser)——一個開源的 HTML5 遊戲框架。

## 六、論文與文檔

- arXiv: [2604.18394](https://arxiv.org/abs/2604.18394)
- 專案頁面: [opengame-project-page.com](https://www.opengame-project-page.com/)
- HuggingFace Paper: [連結](https://huggingface.co/papers/2604.18394)

## 七、觀察與評價

### 值得注意的點

1. **評估維度切中了痛點**。靜態代碼測試（lint、unit test）對遊戲這種高度互動的產品幾乎沒用。OpenGame-Bench 選擇動態執行 + VLM 評分，方向是對的——雖然 VLM 當裁判本身也有侷限性。

2. **三階段訓練路徑合理**。預訓練打底 → SFT 學技能 → RL 在真實反饋中優化，這跟現在主流的大模型訓練思路一致，只是把訓練場域從「文本」換成了「可執行遊戲」。

3. **GameSkill 的 Debug Skill 是實戰關鍵**。多數 AI coding agent 卡在「程式碼看起來對但跑不起來」這個環節。系統性地抓錯、修錯、直到完整可玩——這才是把玩具變成產品的分水嶺。

### 限制與待觀察

- GameCoder-27B 目前沒有公開模型權重，只能透過 API 使用。如果想完全本地部署，選擇有限。
- OpenGame-Bench 評估管道尚未正式釋出， benchmark 結果的透明度有待驗證。
- 展示作品雖然豐富，但都是單一場景、相對簡單的遊戲。複雜系統（多人連線、持久化存檔、物理引擎深度整合）的能力還需要更多實測。
- 資源生成工具（圖片、音訊）需要使用者自帶 API Key——這意味著每次生成都可能產生額外費用。

## 八、總結

OpenGame 是目前最完整的「AI 端到端遊戲開發」開源專案之一。它不是單純的代碼生成器，而是把模板管理、自動除錯、專門訓練的模型和動態評估串成了一個完整流程。對於想快速原型驗證遊戲想法的人來說，價值很直接；對研究者而言，GameCoder-27B 的訓練方法和 OpenGame-Bench 的評估框架也提供了值得追蹤的方向。

---

*本報告由人工整理撰寫，內容基於專案 README、GitHub 頁面及公開文檔。*
