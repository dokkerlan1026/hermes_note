# OpenGame：用一句提示詞生成網頁遊戲

> 來源：[github.com/leigest519/OpenGame](https://github.com/leigest519/OpenGame)
> 整理日期：2026-05-10

---

## 這是什麼東西

OpenGame 是香港中文大學 MMLab 在 2026 年 4 月釋出的開源工具。GitHub 上 1.4k stars，不誇張。它的做法很直接：你輸入一段話描述你想做的遊戲，它會自己生成完整的 HTML5 專案——程式碼、結構、資源全有，最後跑出一個能在瀏覽器裡玩的东西。

比如你說「做一個貪吃蛇，WASD 控制，深色主題」，它就真的做出來一個能玩的貪吃蛇。

## 技術上怎麼做到的

### GameSkill：Agent 的雙手

這套東西分成兩塊：

**Template Skill**——根據你的需求挑引擎（Canvas、Phaser、three.js），生成專案骨架和目錄結構。目的是讓後續編輯有章法，不會亂寫一通。

**Debug Skill**——在沙盒裡跑遊戲，看控制台有沒有錯、互動會不會斷裂，然後系統性地修，直到整個遊戲能完整遊玩為止。

這一步很關鍵。多數 AI coding agent 的問題不在「寫不出像樣的程式碼」，而在「程式碼看起來對但跑不起來」。Debug Skill 就是補這個缺口。

### GameCoder-27B：自己訓練的模型

CUHK 團隊訓了一個 270 億參數的代碼 LLM，流程分三階段：

1. **持續預訓練**——用遊戲開發相關的程式碼資料繼續餵基礎模型
2. **監督微調（SFT）**——用整理好的遊戲開發軌跡數據，涵蓋引擎 API、專案搭建和除錯流程
3. **強化學習**——以實際遊戲可玩性作為獎勵信號，用 OpenGame-Bench 風格的驗證器來評分

先學語言，再學遊戲開發，最後在「能不能跑」這個現實反饋裡練出來。路徑不新奇，但做對了一件事：訓練場域從文本換成了可執行的東西。

### OpenGame-Bench：動態評估

傳統靜態代碼評測（lint、unit test）對遊戲這種高度互動的產品幾乎沒用。OpenGame-Bench 的做法是實際啟動生成的遊戲，用腳本驅動互動行為，然後驗證渲染是否正常、控制有沒有回應、遊戲循環能不能推進、勝負狀態正不正確。

「能跑」和「好玩」之間差了一大段距離。這個 benchmark 至少先解決了第一段。不過 VLM 當裁判本身也有侷限性——它可能看不出來某個動畫帧率不對，或者某個操作手感很怪。

## 展示作品

README 列了六個由單一提示詞生成的完整遊戲：

| 遊戲 | 類型 | 特點 |
|------|------|------|
| Marvel Avengers: Infinity Strike | 橫向卷軸動作平台 | 三位英雄、三個關卡、Thanos Boss、90s Capcom 像素風 |
| Harry Potter: Arithmancy Academy | 回合制卡牌對戰 | 答題施法、魔力共振連擊、哥德奇幻風格 |
| K.O.F: Celestial Showdown | 雙人搶答格鬥 | 物理題目搶答、90s SNK 復古手感 |
| Hajimi Defense: The Tuna Crisis | 塔防 | 貓咪炮塔、笑話風美術 |
| StarWars: Mandalorian Protocol | 頂視角動作 RPG 射擊 | Twin-Stick Shooter、鎧甲人突入帝國基地 |
| Squid Game: Red Light, Green Light | 生存反應遊戲 | 紅綠燈玩法、屍體堆積效果 |

每個作品都有線上 Demo 和原始碼下載。跑起來就三步：`unzip → npm install → npm run dev`。

## 怎麼用

```bash
git clone https://github.com/leigest519/OpenGame.git
cd OpenGame
npm install && npm run build && npm link

opengame -p "Build a Snake clone with WASD controls and a dark theme." --yolo
```

需要 Node.js 20+。主 Agent LLM 支援 OpenAI-compatible API，可以設 `OPENAI_API_KEY`、`OPENAI_BASE_URL`、`OPENAI_MODEL`。本地跑的話換成 GameCoder-27B。圖片、影片、音訊等資源生成工具各自獨立配置，需要帶入對應 Provider Key（tongyi / doubao / openai-compat）。

預設頭部模式下 Agent 只能編輯檔案，不能跑 Shell 指令。加 `--yolo` 才開放執行權限。這算是個安全設計——不給 Agent 隨意跑命令的權力，總比讓它亂搞好。

## 底層依賴

OpenGame 不是從零開始造的：

- Agent 運行時基於 [qwen-code](https://github.com/QwenLM/qwen-code)
- qwen-code 源自 [Google Gemini CLI](https://github.com/google-gemini/gemini-cli)
- 遊戲渲染用 [Phaser](https://github.com/phaserjs/phaser)，一個開源的 HTML5 遊戲框架

簡單說就是：Gemini CLI → Qwen Code → OpenGame，套娃式堆疊。

## 論文與文檔

- arXiv: [2604.18394](https://arxiv.org/abs/2604.18394)
- 專案頁面: [opengame-project-page.com](https://www.opengame-project-page.com/)
- HuggingFace Paper: [連結](https://huggingface.co/papers/2604.18394)

## 我的看法

**評估維度切中了痛點。** 靜態代碼測試對遊戲幾乎沒用。OpenGame-Bench 選擇動態執行 + VLM 評分，方向是對的——雖然 VLM 當裁判本身也有侷限性。它可能看不出來某個動畫帧率不對，或者操作手感很怪，但至少能抓出「完全不能跑」的低級問題。

**三階段訓練路徑合理。** 預訓練打底 → SFT 學技能 → RL 在真實反饋中優化。不新奇，但做對了一件事：訓練場域從文本換成了可執行的東西。

**限制也要說清楚：**

- GameCoder-27B 目前沒有公開模型權重，只能透過 API 使用。想完全本地部署的話選擇有限
- OpenGame-Bench 評估管道尚未正式釋出，benchmark 結果的透明度有待驗證
- 展示作品雖然豐富，但都是單一場景、相對簡單的遊戲。複雜系統（多人連線、持久化存檔、物理引擎深度整合）的能力還需要更多實測
- 資源生成工具需要使用者自帶 API Key——每次生成都可能產生額外費用

## 結語

OpenGame 是目前最完整的「AI 端到端遊戲開發」開源專案之一。它不是單純的代碼生成器，而是把模板管理、自動除錯、專門訓練的模型和動態評估串成了一個完整流程。想快速原型驗證遊戲想法的人可以試試看；研究者的話，GameCoder-27B 的訓練方法和 OpenGame-Bench 的評估框架值得追蹤。

---

*以上內容基於專案 README、GitHub 頁面及公開文檔整理。*
