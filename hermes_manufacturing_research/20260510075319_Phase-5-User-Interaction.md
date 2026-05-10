# Phase 5: 互動與反饋 (Interaction & Feedback)

## 5.1 物件點擊事件偵測
在 PixiJS 中實作 `interactive = true` 與 `click` 事件監聽。

- **實作邏輯**:
  - 當使用者點擊機台或 Agent 時，獲取該格子的 ID (例如 `Station_CNC_01`)。
  - 高亮顯示被選中的物件（改變 Sprite 透明度或增加邊框）。

## 5.2 發送指令回後端
將點擊事件透過 Socket.io 發送給中間層。

- **通訊流程**:
  1. 使用者點擊 `CNC_01`。
  2. 前端執行: `socket.emit('user_action', { target: 'CNC_01', action: 'check_status' })`.
  3. 中間層接收並轉發給 Hermes Master Agent。
  4. Master Agent 決定要呼叫哪個 SubAgent (如 `status` agent) 來回應。

## 5.3 UI Overlay (HUD) 開發規範
- **資訊呈現**: 當選中機台時，顯示其當前狀態（運轉中/閒置）、加工進度、預計完成時間。
- **操作按鈕**: 提供「暫停」、「加速」或「重新排程」按鈕（若權限允許）。

## 5.4 錯誤處理與反饋
- 當 Agent 發生錯誤（如機台故障），前端應顯示警示圖示或紅色閃爍效果。
- WebSocket 斷線時，提示使用者「連線中斷」並嘗試自動重連。
