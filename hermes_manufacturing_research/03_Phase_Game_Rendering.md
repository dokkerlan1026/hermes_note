# Phase 3: 遊戲場景與渲染 (Game Engine & Scene)

## 3.1 網格系統 (Grid System) 實作
工廠被劃分為網格（Grid），每個格子有屬性：`Floor`, `Wall`, `Station_Quote`, `Machine_CNC`.

- **定義地圖陣列**:
  ```typescript
  const TILE_SIZE = 32; // pixels
  const MAP_WIDTH = 20;
  const MAP_HEIGHT = 15;
  
  type TileType = 'FLOOR' | 'WALL' | 'STATION_QUOTE' | 'MACHINE_CNC';
  
  interface GridTile {
    x: number;
    y: number;
    type: TileType;
    interactable?: boolean;
  }
  ```

## 3.2 Agent Sprites (像素角色) 渲染
建立 `AgentEntity` 類別，繼承自 PixiJS `Sprite`。

- **座標映射**:
  - 邏輯網格座標 `(x, y)` -> 畫素座標 `(x * TILE_SIZE, y * TILE_SIZE)`.
  - 使用 `PixiJS Graphics` 繪製基礎地板與牆壁，或使用 `TileSprite` 提升效能。

## 3.3 狀態面板 (HUD) 開發
使用 React 覆蓋層 (Overlay) 顯示當前選中 Agent 的詳細資訊。

- **UI 組件結構**:
  - `AgentInfoPanel`: 顯示 Agent 頭像、任務進度條、當前狀態文字。
  - `GlobalStatusBoard`: 顯示全廠機台運轉概況（綠燈/紅燈）。
- **實作重點**:
  - HUD 應位於 Canvas 之上 (`position: absolute; top: 0; left: 0; pointer-events: none`)，但按鈕需設 `pointer-events: auto`。

## 3.4 資源管理 (Asset Loading)
- **Sprite Sheets**: 將角色動畫（行走、工作、閒置）打包為單一圖片。
- **Loader**: 使用 PixiJS Loader 預加載所有資源，確保遊戲開始時不卡顿。
