# Hermes Manufacturing Agent Teams - 系統架構設計書 (System Architecture)

## 1. 系統總覽 (System Overview)
本專案旨在復刻並實現「製造業 Agent Teams 可視化原型遊戲」。核心設計理念為 **「雙向同步 (Bi-directional Sync)」**：
1.  **後端驅動前端**：Hermes AI 的任務派工與執行狀態（如：Quote 完成、CNC 運轉中）即時驅動像素角色的移動。
2.  **前端反饋後端**：使用者的互動（點擊機台、調整參數）透過 Agent 傳遞回 Hermes Master 進行處理。

## 2. 技術棧建議 (Tech Stack)

| 模組 | 技術選擇 | 詳細理由與版本要求 |
| :--- | :--- | :--- |
| **前端框架** | **React 18 + TypeScript** | 利用 React 的 Component 模型管理非遊戲 UI（設定選單、狀態面板）。TypeScript 確保大型專案的類型安全。 |
| **遊戲引擎** | **PixiJS v7 / v8** | 輕量級 2D WebGL 渲染器，支援 GPU 加速，極適合處理大量像素 Sprite 與即時動畫。 |
| **路徑規劃** | **A* Algorithm (自製)** | 用於計算 Agent 在工廠網格地圖上的最短移動路徑，需實作避開障礙物邏輯。 |
| **狀態管理** | **Zustand** | 輕量級且高效的 Store，比 Redux 更適合管理分散式的 Agent 座標與任務狀態。 |
| **通訊協定** | **Socket.io** | 確保 Hermes 的事件能毫秒級推送到前端進行渲染，支援自動重連機制。 |
| **資源格式** | **Sprite Sheets (PNG)** | 將角色動畫（行走、工作、閒置）打包為單一圖片以提升效能。 |

## 3. 架構圖 (Architecture Diagram)

```text
[使用者瀏覽器]
      |
      +--- [Web Frontend: Game Engine & UI] ---+
      |   (React + PixiJS/Phaser.js)            |
      |                                           |
      |   - Factory Map (Grid System)             |
      |   - Agent Sprites (Pixel Art)             | <--- 即時動畫渲染
      |   - Status Panel (HUD)                    |
      |                                           |
      +-------------------+-----------------------+
                          | WebSocket / API Bridge
                          v
[中間層: Hermes Event Bridge]
      |
      +--- [Event Listener] ---+  [State Manager]
      | (監聽 Hermes Logs)     | (管理 Agent 座標與狀態)
      |                        |
      +------------------------+
                          |
                          v
[Hermes Manufacturing Backend]
      |
      +--- manufacturing-main (Master)
      +--- SubAgents: quote, sales, status, cnc, image-analysis, line-info
```

## 4. 核心通訊協議 (Communication Protocol)

### Agent Update (後端 -> 前端)
```json
{
  "type": "AGENT_UPDATE",
  "payload": {
    "agent_id": "quote_agent_01",
    "role": "Quote Engineer",
    "grid_pos": { "x": 5, "y": 12 },
    "status": "WORKING", // IDLE, WALKING, WORKING, ERROR
    "current_task": "Processing Order #2026-05"
  }
}
```

### User Action (前端 -> 後端)
```json
{
  "type": "USER_ACTION",
  "payload": {
    "target_station_id": "CNC_MACHINE_01",
    "action_type": "CHECK_STATUS",
    "timestamp": 1715328000
  }
}
