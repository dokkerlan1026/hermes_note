# Phase 2: 中間層：Hermes Event Bridge (核心橋接)

## 2.1 開發 Event Listener (事件監聽器)
建立一個守護進程/服務，持續監控 Hermes 的 Logs。

- **實作邏輯**：
  - 使用 `tail -f` 或檔案系統 Watcher 監測 Log 檔。
  - 解析 Log 內容，提取關鍵狀態：`Agent ID`, `Current Task`, `Status`, `Target Location`.
- **Log Parser (JSON Extraction)**：
  ```python
  import re, json
  
  def parse_hermes_log(line):
      # Regex to find JSON object in log line
      match = re.search(r'({.*})', line)
      if match:
          return json.loads(match.group(1))
      return None
  ```

## 2.2 開發 State Manager (狀態管理器)
在記憶體中維護一個全局狀態樹，記錄所有 Agent 的虛擬座標與職責。

- **State Schema**:
  ```typescript
  interface GameState {
    agents: Record<string, Agent>; // Keyed by agent_id
    grid: GridMap;                 // Map layout
    logs: LogEntry[];              // Recent activity log
  }
  
  interface Agent {
    id: string;
    role: string;
    x: number;
    y: number;
    status: 'IDLE' | 'WALKING' | 'WORKING';
    task?: string;
  }
  ```

## 2.3 WebSocket Server 實作 (Socket.io)
建立 Socket.io Server，監聽前端連線並推送狀態。

- **Server Code Snippet**:
  ```javascript
  import { Server } from "socket.io";
  
  const io = new Server(3001, { cors: { origin: "*" } });
  
  io.on('connection', (socket) => {
    console.log('Client connected:', socket.id);
    
    // Push initial state
    socket.emit('init_state', currentState);
    
    // Listen for user actions from frontend
    socket.on('user_action', (data) => {
      bridgeToHermes(data); // Forward to Hermes Master
    });
  });
  
  // Function to update all clients when Hermes changes
  function broadcastAgentUpdate(agentData) {
    io.emit('agent_update', agentData);
  }
  