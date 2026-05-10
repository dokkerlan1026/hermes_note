# Phase 1: 基礎建設與環境設定 (Infrastructure)

## 1.1 前端專案初始化 (Frontend Init)
使用 Vite + React + TypeScript 建立專案結構，確保效能與開發體驗。

- **執行指令**：
  ```bash
  npm create vite@latest hermes-game -- --template react-ts
  cd hermes-game
  npm install
  ```
- **安裝核心依賴 (Dependencies)**：
  - `pixi.js`: 2D 渲染引擎。
  - `socket.io-client`: WebSocket 通訊。
  - `zustand`: 狀態管理。
  - `pathfinding`: A* 演算法庫（或自製）。

## 1.2 專案目錄結構 (Directory Structure)
```text
/hermes-game
├── /public/assets        # 存放像素美術資源 (Sprite Sheets, Backgrounds)
└── /src
    ├── /components       # React UI 組件 (HUD, Settings Modal)
    ├── /engine           # PixiJS 遊戲邏輯核心
    │   ├── GameScene.ts  # 場景管理
    │   ├── GridSystem.ts # 網格系統定義
    │   └── AgentSprite.ts# 角色渲染類別
    ├── /store            # Zustand State Management
    ├── /utils            # A* Algorithm, Socket Helpers
    └── main.tsx          # Entry Point
```

## 1.3 後端 Agent 配置確認 (Backend Config)
- **Hermes Configuration**: 確保 `manufacturing-main` Master Agent 與 SubAgents (`quote`, `sales`, `cnc`) 的設定檔已就緒。
- **Log Format**: 各 Agent 需輸出標準化 JSON Log，以便中間層解析：
  ```json
  {"timestamp": "2026-05-10T10:00:00Z", "agent_id": "cnc_01", "status": "RUNNING", "log_msg": "Cutting steel..."}
  