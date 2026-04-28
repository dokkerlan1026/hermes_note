# OpenGame 研究報告

## 📊 基本資訊

| 項目 | 內容 |
|------|------|
| **Repository** | https://github.com/leigest519/OpenGame |
| **Stars** | 1,418 ⭐ |
| **Forks** | 180 🍴 |
| **Default Branch** | main |
| **作者團隊** | CUHK MMLab (香港中文大學機器學習實驗室) |
| **主要聯絡人** | Xiangyu Yue* (*通訊作者) |
| **发布日期** | 2026-04-21 |

## 🎯 核心定位

**OpenGame**: Open Agentic Coding for Games

> 一個開源的 Agent 框架，用於從單一提示（prompt）端到端生成完整的網頁遊戲。

### 關鍵創新點

OpenGame 解決了當前 LLM 和代碼 Agents 在遊戲開發上的痛點：
- ❌ **跨檔案不一致性** (cross-file inconsistencies)
- ❌ **場景連接斷裂** (broken scene wiring)
- ❌ **邏輯不連貫** (logical incoherence)

## 🧠 核心技術架構

### 1. Game Skill（遊戲技能）

這是 OpenGame 的核心可重複能力，分為兩部分：

#### Template Skill（模板技能）
- 選擇合適的引擎/模板（canvas, Phaser, three.js 等）
- 建立穩定、規範化的專案結構
- 確保後續修改保持一致性

#### Debug Skill（除錯技能）
- 在沙盒中執行遊戲
- 偵測整合錯誤、Console 錯誤、斷裂的互動
- 系統性地解決問題直到遊戲可完整遊玩

**效果**：讓 Agent 從「寫出合理的程式碼」進化到「交付可運作的遊戲」。

### 2. GameCoder-27B（專精模型）

一個專門為遊戲引擎掌握的 Code LLM，通過三階段訓練：

1. **Continual Pre-training** (持續預訓練)
2. **Supervised Fine-Tuning (SFT)** - 監督式微調
   - 針對遊戲開發路徑進行 curated 數據
   - 涵蓋引擎 API、專案 scaffold、除錯工作流
3. **Execution-Grounded Reinforcement Learning** - 執行導向的強化學習
   - 獎勵信號來自真實遊戲可玩性驗證

### 3. OpenGame-Bench（評估基準）

一個針對「互動式網頁遊戲生成 Agents」的評估框架：

- 🔍 **Build Health** (建構健康度)
- 🔍 **Visual Usability** (視覺可用性)
- 🔍 **Intent Alignment** (意圖對齊度)

評估方式：
- 動態啟動生成的遊戲
- 通過腳本互動驅動遊戲
- 使用 headless browser + VLM 判斷可玩性標準（渲染、控制、遊戲迴圈進展、勝利/失敗狀態等）

## 🎮 展示案例（Playable Demos）

OpenGame 成功生成了以下 6 個完整遊戲：

### 1. Marvel Avengers: Infinity Strike（漫威復仇者聯盟：無限打擊）
- **類型**：橫向動作平台遊戲
- **特色**：可選擇鋼鐵人、雷神、浩克三大英雄
- **关卡**：3 個場景（毀滅城市、SHIELD 飛行航母、泰坦星）
- **Boss**：薩諾斯使用無限寶石力量
- **風格**：90 年代 Capcom 街機像素藝術

### 2. Harry Potter: Arithmancy Academy（哈利波特：算數魔法學院）
- **類型**：回合制卡牌戰鬥遊戲
- **特色**：需要回答數學/科學題目才能施放魔法卡牌
- **創新機制**：Magic Resonance 連擊系統（連續答對增加傷害）
- **風格**：哥特奇幻像素藝術 + 羊皮紙 UI

### 3. K.O.F: Celestial Showdown（拳皇：天界決鬥）
- **類型**：雙人本地問答格鬥遊戲
- **特色**：通過搶答物理題目來進行格鬥
- **機制**：答對造成傷害，答錯反噬自身
- **場景**：中式仙境「天庭」風格
- **風格**：90 年代 SNK 街機 16-bit 像素藝術

### 4. Hajimi Defense: The Tuna Crisis（哈吉米保衛戰：金槍魚危機）
- **類型**：塔防遊戲
- **特色**：可愛貓咪炮塔保衛金槍魚罐头
- **敵人**：黃瓜和吸塵器等家居害蟲
- **創新機制**：可點擊破壞障礙物釋放建築空間
- **風格**：手繪、柔和色調、超可愛（Kawaii）

### 5. StarWars: Mandalorian Protocol（星際大戰：曼達洛人協議）
- **類型**：頂視動作 RPG 射擊遊戲
- **特色**：扮演曼達洛人突入帝國基地拯救 Grogu
- **玩法**：Twin-Stick Shooter + 戰術深度系統
- **武器**：爆能槍（遠程）、Beskar 長矛（近戰）、噴氣背包閃避
- **風格**：金屬科幻像素藝術

### 6. Squid Game: Red Light, Green Light（魷魚遊戲：紅燈綠燈）
- **類型**：生存反應遊戲
- **特色**：重現經典「紅燈綠燈」場景
- **機制**：玩偶唱歌時跑，转头時必須立即停止
- **視覺細節**：死亡身體和血泊不會消失（堆積營造混亂氛圍）
- **風格**：粗糲、寫實 16-bit 像素藝術

## 🚀 使用方式

### 安裝步驟

```bash
# 前置條件：Node.js 20+
curl -qL https://www.npmjs.com/install.sh | sh

# 從源碼安裝（推薦）
git clone https://github.com/leigest519/OpenGame.git
cd OpenGame
npm install
npm run build
npm link
```

### Quick Start（快速開始）

OpenGame 目前透過命令行以 **headless mode**（無頭模式）運行：

```bash
# 建立新遊戲資料夾
cd agent-test
mkdir -p games/my-game && cd games/my-game

# 從單一提示生成遊戲
opengame -p "Build a Snake clone with WASD controls and a dark theme." --yolo
```

完成後打開生成的 `index.html`（或執行顯示的 dev-server 命令）即可遊玩。

### 環境變數配置

#### 主要 Agent LLM（OpenAI API 兼容）

```bash
export OPENAI_API_KEY="***"
export OPENAI_BASE_URL="https://api.openai.com/v1"     # 可選
export OPENAI_MODEL="gpt-4o"                            # 可選，本地運行時可改用 GameCoder-27B
```

#### Asset / GDD Provider Keys（圖片、影片、音訊、推理）

OpenGame 的資產生成工具可以配置不同的提供者：

```bash
export OPENGAME_IMAGE_PROVIDER=tongyi         # tongyi | doubao | openai-compat
export OPENGAME_IMAGE_API_KEY=***
# ... 同樣適用於 OPENGAME_REASONING_*, OPENGAME_VIDEO_*, OPENGAME_AUDIO_*
```

可混合使用不同提供者（例如：Tongyi 圖片、Doubao 影片、OpenAI 推理）。

## 📁 Repository 結構概覽

專案包含以下主要部分：

### Core Components
- **agent-test/** - Agent 測試框架和模板系統
- **docs/** - 完整文檔（用戶配置、功能說明等）
- **assets/** - 遊戲素材、海報、下載檔案
- **.qwen/** - 設定檔案（後向兼容，未來會遷移到 `.opengame`）

### Infrastructure
- **.github/workflows/** - CI/CD 自動化流程
- **.aoneci/** - AOneCI 配置
- **.gcp/** - GCP Docker 部署配置

### Configuration Files
- `.env.example` - 環境變數範本
- `settings.json` - 用戶和專案設定
- `package.json` - Node.js 依賴

## 🏗️ 技術堆疊

### 前端遊戲引擎
- **Phaser** - HTML5 遊戲框架（渲染和機制的核心）
- Canvas API
- Three.js（3D 場景支持）

### Agent Runtime
基於 **[qwen-code](https://github.com/QwenLM/qwen-code)**：
- 繼承 Google Gemini CLI 的架構設計
- 擴展了 Game Skill、GameCoder-27B 集成、OpenGame-Bench 工具

### 開發環境
- Node.js >= 20.0.0
- npm/yarn 包管理
- VS Code + Qwen 擴展（`.vscode/extensions.json`）

## 🔑 關鍵功能特點

1. **端到端生成** - 從單一提示到完整可遊玩遊戲
2. **智能模板選擇** - 自動匹配最合適的遊戲引擎和架構
3. **系統性除錯** - 不僅修復語法錯誤，還解決整合問題
4. **跨檔案一致性維護** - 確保多檔案專案的邏輯連貫
5. **可玩性驗證** - 通過 OpenGame-Bench 進行動態測試
6. **混合 AI Provider 支持** - 靈活配置不同的資產生成服務

## 📚 引用與貢獻來源

OpenGame 建立在以下開源項目之上：

1. **[qwen-code](https://github.com/QwenLM/qwen-code)** - Agent runtime 和 CLI scaffold
2. **[Google Gemini CLI](https://github.com/google-gemini/gemini-cli)** - 原始 CLI 架構
3. **[Phaser](https://github.com/phaserjs/phaser)** - HTML5 遊戲框架

## 📄 研究資源

- **項目頁面**：https://www.opengame-project-page.com/
- **arXiv 論文**：https://arxiv.org/abs/2604.18394
- **Hugging Face Paper**：https://huggingface.co/papers/2604.18394

## 🎓 技術創新意義

OpenGame 代表了 AI Agent 在遊戲開發領域的重要突破：

### 從「程式碼生成」到「完整應用交付」
- 傳統 LLM Agents：擅長解決孤立編程任務
- OpenGame：能夠處理複雜、多檔案、狀態耦合的完整應用程式

### 可玩性驗證的革命
- 傳統方法：靜態代碼檢查
- OpenGame-Bench：動態執行、互動測試、VLM 判斷

### Game Skill 的可重複性
- 建立持續學習的知識庫（Template Library）
- 維護活躍的除錯協議（Debug Protocol）
- 從經驗中成長的能力系統

## 📊 性能表現

根據研究報告：
- **測試規模**：150 個多樣化遊戲提示
- **結果**：OpenGame 建立了新的 state-of-the-art (SOTA)
- **評估維度**：Build Health, Visual Usability, Intent Alignment

## 🌟 未來展望

根據團隊聲明，OpenGame 希望推動代碼 Agents：
> "超越離散的軟體工程問題，向構建複雜、互動的真實世界應用程式前進"

### 潛在應用方向
- **教育工具**：快速生成教學遊戲、互動學習內容
- **原型開發**：遊戲設計師的快速概念驗證工具
- **創意激發**：幫助創作者探索新的遊戲機制和玩法
- **低代碼平台**：降低遊戲開發門檻，讓更多人參與創作

## 📝 總結

OpenGame 是一個革命性的開源框架，展示了 AI Agent 在複雜應用程式生成方面的巨大潛力。通過 Game Skill、GameCoder-27B 和 OpenGame-Bench 的協同作用，它成功解決了 LLM Agents 在遊戲開發上的核心痛點，為未來的 AI 輔助創意和軟體工程开辟了新的道路。

---

**報告生成日期**：2026-04-28  
**研究來源**：GitHub Repository + README.md + arXiv Paper  
**整理者**：Hermes AI Agent  
