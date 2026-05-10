## GitHub 熱門趨勢：AI Agent 框架與工具（2026/05）

## 這個階段在看什麼
這段時間 GitHub 上 **AI Agent 的基礎設施層**，包括 Agent Harness（執行環境）、Headless Browser、以及自主代理架構。這是整個 AI Agent 生態系統的「底盤」，為上層的 Skill 和應用提供運行載體。

---

## 統計數據

- **總星數**: 98,063 ★
- **專案數量**: 11 個

---

## 關鍵項目詳情

### 1. [garrytan/gbrain](https://github.com/garrytan/gbrain) -- ★ 14,239

語言：TypeScript  | 14239★

**描述**: Garry's Opinionated OpenClaw/Hermes Agent Brain -- 為 Hermes/Claw 代理設計的「大腦」模組，提供推理、決策和工具調用的統一接口。

這個專案說明 Agent 架構的標準化趨勢--將 Agent 的核心能力（感知->思考->行動）抽象為可插拔的 Brain 模組。TypeScript 實現意味著它天然適合 Web 生態整合。

---

### 2. [HKUDS/OpenHarness](https://github.com/HKUDS/OpenHarness) -- ★ 12,267

語言：Python  | 12267★

**描述**: OpenAgent Harness with a Built-in Personal Agent--Ohmo! -- 開源 Agent 執行環境，內建個人化代理 Ohmo。

港大（HKUDS）的學術研究專案，將 Agent 框架與個人助手結合。看得出 Agent 技術正從「通用工具」走向「個人化服務」。

---

### 3. [browser-use/browser-harness](https://github.com/browser-use/browser-harness) -- ★ 11,874

語言：Python  | 11874★

**描述**: Browser Harness | Self-healing harness that enables LLMs to complete any task -- 自癒式瀏覽器自動化框架。

「自癒能力」是本專案的核心賣點--當網頁結構變化時，Agent 能自動適應而非崩潰。這解決了長期以來的爬蟲/自動化痛點。

---

### 4. [h4ckf0r0day/obscura](https://github.com/h4ckf0r0day/obscura) -- ★ 11,320

語言：Rust  | 11320★

**描述**: The headless browser for AI agents and web scraping -- 為 AI Agent 量身打造的無頭瀏覽器。

用 Rust 實現意味著極致的效能和安全性。與通用無頭瀏覽器（如 Puppeteer）不同，Obscura 針對 Agent 使用場景做了深度優化--反檢測、快速加載、低記憶體佔用。

---

### 5. [browser-use/video-use](https://github.com/browser-use/video-use) -- ★ 7,063

語言：Python  | 7063★

**描述**: Edit videos with coding agents -- 用編程代理編輯影片。

Agent 應用場景的延伸--從網頁操作擴展到多媒體處理。大概是想 Agent 的能力邊界正在快速擴張。

---

### 6. [HKUDS/Vibe-Trading](https://github.com/HKUDS/Vibe-Trading) -- ★ 6,623

語言：Python  | 6623★

**描述**: Vibe-Trading: Your Personal Trading Agent -- 個人化交易代理。

金融領域的 Agent 應用範例，結合市場數據分析和自動化交易決策。

---

### 7. [ultraworkers/claw-code-parity](https://github.com/ultraworkers/claw-code-parity) -- ★ 6,691

語言：Rust  | 6691★

**描述**: claw-code Rust port parity work -- Claw Code 的 Rust 移植版本。

Claw Code（Claude Code 的開源替代品）正在向多語言生態擴展，Rust 版提供更高效能和安全性。

---

### 8. [TencentCloud/CubeSandbox](https://github.com/TencentCloud/CubeSandbox) -- ★ 5,222

語言：Rust  | 5222★

**描述**: Instant, Concurrent, Secure & Lightweight Sandbox for AI Agents -- 即時、並行、安全、輕量級的 Agent 沙箱。

騰訊雲推出的 Agent 安全執行環境，解決了 Agent 在雲端運行時的安全隔離問題。

---

### 9. [kevinrgu/autoagent](https://github.com/kevinrgu/autoagent) -- ★ 4,403

語言：Python  | 4403★

**描述**: autonomous harness engineering -- 自主執行環境工程。

聚焦於 Agent 運行環境的自動化工程和部署，屬於基礎設施工具鏈的一部分。

---

### 階段洞察

1. **Rust 崛起**: Obscura、Claw Code Parity、CubeSandbox 均使用 Rust，反映 Agent 基礎設施對效能和安全性的追求。
2. **自癒能力成為標配**: Browser Harness 的「自癒」特性代表未來 Agent 框架的關鍵需求--容錯和適應性。
3. **學術與工業並進**: HKUDS（港大）和騰訊雲同時推出 Agent 框架，顯示研究界和產業界都在加速布局。
4. **個人化趨勢**: Vibe-Trading、OpenHarness (Ohmo) 等專案指向「個人 Agent」的未來--每個人都有一個專屬 AI 代理。

---

## 總結

本階段共分析了 **11 個專案**，總計 **98,063★**。這些專案代表了 GitHub 上 AI/技術領域的最新趨勢和創新方向。

*報告生成時間: 2026-05-10 08:12:28*


---

## 我的觀察

這幾個月的趨勢很明顯：Agent 框架正在從「能跑」變成「好用」。gbrain 和 OpenHarness 的星數都破萬，代表開發者真的在投入這個領域。不過大部分專案還停留在概念驗證階段，真正上線運用的不多。

Rust 語言出現在 headless browser（Obscura）也值得注意——效能和安全性的需求已經明確了。