# Phase 4: Agent 邏輯與動畫 (Movement & Pathfinding)

## 4.1 A* 路徑規劃演算法整合
當後端指派新任務（例如：從倉庫走到 CNC）時，計算避開牆壁的最短路徑點陣列。

- **實作細節**:
  - **Node Class**: `x`, `y`, `g` (cost from start), `h` (heuristic to goal), `f = g + h`.
  - **Heuristic Function**: Manhattan Distance `Math.abs(dx) + Math.abs(dy)`。
  - **Open/Closed Lists**: 標準 A* 邏輯，確保不走回頭路且避開障礙物。

## 4.2 動畫狀態機 (State Machine)
為每個 Sprite 實作狀態切換邏輯：

- **狀態定義**:
  - `IDLE`: 靜止或微小呼吸動畫（無移動）。
  - `WALKING`: 根據移動方向播放行走幀，更新座標。
  - `WORKING`: 到達目標後，播放工作動畫（如機械臂動作），停留一定時間。

- **狀態轉換圖**:
  ```text
  [IDLE] --(收到新任務)--> [WALKING] --(抵達目標)--> [WORKING] --(任務完成)--> [IDLE]
   ^                                                        |
   +--------------------------------------------------------+
  ```

## 4.3 雙向同步：後端 -> 前端
- **監聽 WebSocket**: 接收 `AGENT_UPDATE` 訊息。
- **平滑移動 (Lerp)**: 不直接瞬移，而是使用線性插值 (Linear Interpolation) 讓 Agent 看起來是在「走」過去。
  ```typescript
  agent.x += (targetX - agent.x) * 0.1; // Smooth transition
  ```

## 4.4 任務佇列管理
- 每個 Agent 應具備一個任務佇列 (`Task Queue`)。
- 當 `WORKING` 狀態結束，自動從佇列取出下一個任務並開始移動。
