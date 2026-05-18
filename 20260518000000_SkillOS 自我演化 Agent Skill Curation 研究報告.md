# SkillOS 學習報告：自我演化 Agent 的 Skill Curation 學習方法

> **論文**: SkillOS: Learning Skill Curation for Self-Evolving Agents
> **作者**: Siru Ouyang, Jun Yan, Yanfei Chen, Rujun Han, Zifeng Wang, Bhavana Dalvi Mishra, Rui Meng, Chun-Liang Li, Yizhu Jiao, Kaiwen Zha, Maohao Shen, Vishy Tirumalashetty, George Lee, Jiawei Han, Tomas Pfister, Chen-Yu Lee
> **機構**: Google Cloud AI Research、UIUC、MIT
> **日期**: 2026-05-07
> **arXiv**: [2605.06614](https://arxiv.org/abs/2605.06614)
> **類別**: cs.AI, cs.CL

---

## 一、研究背景與問題定義

### 1.1 核心問題

目前基於 LLM 的 Agent 大多被部署為「一次性問題解決者」（one-off problem solvers）—— 每次從零開始處理新任務，無法從過去的互動經驗中學習。這在「串流式」（streaming）場景中尤其致命：任務依序到來，但 Agent 無法累積和复用經驗。

### 1.2 為什麼 Skill Curation 是瓶頸？

可重用的技能（skills）是從經驗中提取的程序性記憶（procedural memory），是自我演化的基礎。然而，**高品質的 Skill Curation**（技能的提取、整合、更新、刪除）仍是主要瓶頸：

| 現有方法 | 限制 |
|---------|------|
| **手動 curation**（如 Anthropic Skills） | 需要大量人類專家知識，無法擴展 |
| **提示/啟發式方法** | 依賴固定規則，缺乏下游效能反饋 |
| **RL 訓練短期技能操作** | 學習訊號稀疏，難以掌握複雜的管理操作（更新、刪除） |

### 1.3 研究目標

設計一個**經驗驅動的強化學習訓練方法**，讓 Agent 能夠：
1. 從過去互動中自動提取高品質技能
2. 持續更新和優化技能庫
3. 在串流式任務中自我演化，越來越強

---

## 二、SkillOS 核心方法

### 2.1 整體架構

SkillOS 採用**多代理模組化設計**，包含兩個核心組件：

```
┌─────────────────────────────────────────────────┐
│              SkillOS Framework                    │
│                                                   │
│  ┌──────────────┐       ┌──────────────────┐     │
│  │  Agent        │       │  Skill Curator    │     │
│  │  Executor     │◄─────►│  (可訓練)         │     │
│  │  (凍結)       │       │                   │     │
│  └──────────────┘       └──────────────────┘     │
│         ▲                       │                 │
│         │                       │                 │
│         └──── SkillRepo ────────┘                 │
│         (Markdown 技能庫)                          │
└─────────────────────────────────────────────────┘
```

- **Agent Executor**（凍結）：負責使用技能庫中的技能來解決任務。使用 BM25 檢索相關技能，然後執行任務。
- **Skill Curator**（可訓練）：觀察任務執行過程的軌跡（trajectory），生成結構化的 curation 操作（insert/update/delete），更新外部 SkillRepo。
- **SkillRepo**：外部技能倉庫，技能以 Markdown 文件格式儲存（YAML frontmatter + Markdown body）。

### 2.2 技能格式（Skill Format）

每個技能代表為單一 Markdown 文件：

```markdown
---
name: skill_name
description: 當...時使用此技能
---

## 技能內容
- 可執行的知識
- 工作流程
- 約束條件
- 可重用的啟發式方法
```

這參考了 Anthropic 的 SKILL.md 格式，但簡化為單一文件。

### 2.3 Curation 操作

Skill Curator 生成三種操作：
1. **insert_skill**：插入新技能
2. **update_skill**：更新現有技能
3. **delete_skill**：刪除有害或不必要的技能

### 2.4 訓練實例構造（Training Instance Construction）

關鍵設計：將相關任務**分組**作為訓練實例。

**步驟 1**：使用 Gemini-2.5-Pro 為每個任務標註 skill-relevant 屬性（如主題、常見陷阱）：
```
Z_i = {z_i^1, z_i^2, ..., z_i^{|Z_i|}}
```
例如數學推理中，屬性可能包括「代數」、「傅立葉變換」。

**步驟 2**：基於屬性相似度，將任務集分割為 M 個任務群組：
```
D = {G_1, G_2, ..., G_M}
G_m = {x_m,1, x_m,2, ..., x_m,|G_m|}
```

**關鍵洞察**：同一群組內的任務具有非平凡的依賴關係，早期經驗產生的技能可以用來評估是否有助於解決後續相關任務。這提供了更密集的學習訊號。

### 2.5 組合獎勵函數（Composite Reward）

這是 SkillOS 的另一個核心貢獻。獎勵函數由四個部分組成：

```
r = r_task + λ_f * r_fc + λ_u * r_cnt + λ_c * r_comp
```

| 獎勵項 | 符號 | 計算方式 | 目的 |
|--------|------|---------|------|
| **任務結果獎勵** | r_task | 剩餘任務的平均成功率 | 提供下游效能訊號 |
| **函數呼叫獎勵** | r_fc | 有效函數呼叫的比例 | 確保 curator 產生合法的操作 |
| **內容質量獎勵** | r_cnt | 外部評判者（Qwen3-32B）給分的平均值 | 確保技能語義有意義 |
| **壓縮獎勵** | r_comp | 1 - (SkillRepo 大小 / 輸入上下文大小) | 鼓勵提煉可复用技能，避免直接複製軌跡 |

### 2.6 GRPO 訓練演算法

使用 **Grouped Reward Policy Optimization (GRPO)**：

```
Algorithm 1: Training Skill Curator with Task Groups using GRPO
1: for each training step do
2:     G = (x_1, ..., x_|G|), S ← ∅
3:     for task index i = 1, ..., |G| do
4:         S̃ ← BM25(x_i, S)           # 檢索相關技能
5:         ξ_i ← RunTask(S̃, π_L, x_i)  # 在凍結的 executor 上執行
6:         c_i ~ π_S(· | ξ_i, S̃)      # 從 skill curator 採樣 rollouts
7:         S ← ApplyOps(S, c_i)        # 應用 insert/update/delete
8:     end for
9:     r ← CalculateReward(ξ, c)       # 計算組合獎勵
10:    Update π_S                       # 使用 GRPO 更新 curator
11: end for
```

**GRPO 優勢**：訓練穩定性高、樣本效率高，且不需要 critic 模型。

**關鍵設計**：對每個任務群組，採樣 N 個獨立的 curation sequence rollouts。不同 rollouts 演化不同的技能庫歷史，GRPO 優勢跨所有 rollouts 計算：
```
A^n = r^n - (1/N) * Σ_{n'=1}^{N} r^{n'}
```

---

## 三、實驗結果

### 3.1 使用的 Benchmark

| 類別 | Benchmark | 說明 |
|------|-----------|------|
| **多輪 Agent 任務** | ALFWorld | 家庭環境中的互動任務（Pick, Look, Clean, Heat, Cool, Pick2） |
| **多輪 Agent 任務** | WebShop | 模擬購物環境，根據自然語言指令購買商品 |
| **單輪推理任務** | DeepMath-103K | 約 103K 道高難度數學問題 |
| **單輪推理任務** | AIME24 / AIME25 | 美國數學邀請賽題目 |
| **單輪推理任務** | GPQA-Diamond | 研究生級別的科學問答（生物、物理、化學） |

### 3.2 使用的執行器

- **Qwen3-8B**（開源）
- **Qwen3-32B**（開源）
- **Gemini-2.5-Pro**（閉源）
- **Gemini-3.1-Flash-Lite**（閉源）

### 3.3 基線方法

| 基線 | 類型 | 特點 |
|------|------|------|
| No Memory | 無記憶 | 每次從零開始 |
| ReasoningBank | 經驗提煉 | 從軌跡中提煉推理洞見 |
| MemP | 程序記憶 | 基於規則的記憶體管理 |
| SkillOS-base | 無 RL 訓練 | 相同架構但無 GRPO 微調 |
| SkillOS-gemini | 閉源 curator | 使用 Gemini-2.5-Pro 作為 curator |

### 3.4 主要結果

#### ALFWorld（Qwen3-8B）

| 方法 | Avg. SR | Steps |
|------|---------|-------|
| No Memory | 47.9 | 21.1 |
| ReasoningBank | 54.1 | 19.2 |
| MemP | 48.4 | 20.0 |
| SkillOS-base | 51.3 | 19.5 |
| SkillOS-gemini | 62.8 | 18.8 |
| **SkillOS** | **68.6** | **18.5** |

**SkillOS 比最強基線（ReasoningBank）提升 +14.8% 成功率，並減少 -3.7% 步驟數。**

#### WebShop

| 方法 | Score | Steps |
|------|-------|-------|
| No Memory | 65.3 | 8.5 |
| ReasoningBank | 70.1 | 7.8 |
| MemP | 66.8 | 8.2 |
| SkillOS | **72.5** | **7.2** |

#### DeepMath + AIME + GPQA

| 方法 | DeepMath | AIME24 | AIME25 | GPQA | Avg |
|------|----------|--------|--------|------|-----|
| No Memory | 52.3 | 30.0 | 26.7 | 45.5 | 38.6 |
| ReasoningBank | 55.8 | 33.3 | 30.0 | 47.0 | 41.5 |
| **SkillOS** | **58.5** | **36.7** | **33.3** | **50.5** | **44.8** |

**SkillOS 在推理任務中也一致超越所有基線。**

### 3.5 跨執行器泛化

當使用 **Gemini-2.5-Pro** 作為凍結執行器時：
- SkillOS 的 8B 訓練 curator 甚至超越了 Gemini-2.5-Pro 直接作為 curator

當使用 **Gemini-3.1-Flash-Lite** 作為執行器時：
- SkillOS 達到 73.1% 成功率，比最強外部基線 ReasoningBank（66.0%）提升 +7.1 點

**關鍵結論**：訓練後的 curator 具有良好的跨模型和跨任務泛化能力。

---

## 四、深度分析與洞察

### 4.1 學習到的 Curator 行為

**有針對性的技能使用**：RL 訓練後的 curator 不像 baseline 那樣泛化地使用技能，而是更精準地針對特定任務類型提取和管理技能。

**技能的自主演化**：SkillRepo 中的技能會隨著時間演化成結構更豐富的 Markdown 文件，編碼更高層次的 meta-skills。

### 4.2 Case Study 分析

#### 案例 1：Agentic 任務的復原策略

SkillOS 學會提取**失敗復原的元策略**：
```
完整搜尋 → 確認不可用 → 識別替代物 → 使用替代物繼續
```
而非記憶特定物件搜尋軌跡。這展示了**組合式 curation** 能力。

#### 案例 2：數學推理的分支推理

針對數學推理，SkillOS 提取**分支推理技能**：單一技能編碼多個解題路徑，每個路徑都附有公式、應用場景和先決條件。

#### 案例 3：RL 訓練 vs. 無訓練

- **SkillOS-base**（無 RL）：只輸出泛化的分區食譜，沒有明確公式、約束或範例
- **SkillOS**（有 RL）：提取具體且可复用的計數框架，包含明確的約束 formulation、方程 setup 和 worked example

### 4.3 為什麼 SkillOS 有效？

1. **長期效用錨定**：分組訓練讓早期產生的技能被後續任務評估，迫使 curator 關注技能的長期可用性
2. **密集學習訊號**：組合獎勵將延遲、間接的監督轉化為直接的 curation 學習訊號
3. **模組化設計**：凍結的 executor 讓 curator 專注於技能管理，不受 executor 適應的干擾
4. **壓縮獎勵的規避效果**：防止 curator 簡單地複製原始軌跡，強制其提煉抽象模式

---

## 五、與 Hermes Agent 系統的關聯

### 5.1 直接啟發

這篇論文與 Hermes Agent 系統高度相關：

1. **Skills 整合**：Hermes 有超過 50+ 個 skills（編程、研究、瀏覽器自動化等）。SkillOS 的方法可以自動管理和優化這些 skills 的更新、刪除和組織。

2. **自我演化 Agent**：當前 Hermes 系統需要手動更新 skills。如果套用 SkillOS 的 RL 訓練方法，Agent 可以根據使用經驗自動：
   - 插入新發現的有效 workflow 作為 skills
   - 更新過時的 skills
   - 刪除無效或衝突的 skills

3. **Graphify 集成**：SkillRepo 的結構化技能可以與 Graphify 知識圖譜整合，形成更豐富的記憶表示。

4. **Cron 自動整理**：Hermes 的 cron job（每日 09:00 UTC）可以結合 SkillOS 的 curator，自動評估和優化 skills。

### 5.2 可能的改進方向

| 改進 | 說明 |
|------|------|
| RL 訓練的 Skill Curator | 為 Hermes 訓練一個專門的 skill curator，根據使用反饋自動管理 skills |
| 多 Agent 共享 SkillRepo | Hermes 的多代理系統（Master-Slave）可以共享 SkillRepo |
| 階層化 Skills | 擴展單一 Markdown 文件到多文件、階層化的技能結構 |
| 密集檢索 | 用向量檢索替換 BM25，提高技能檢索準確率 |

---

## 六、限制與未來方向

### 6.1 論文明確承認的限制

| 限制 | 說明 |
|------|------|
| **檢索機制簡單** | 目前僅使用 BM25，未來可採用密集檢索或混合檢索 |
| **技能表示簡化** | 單一 Markdown 文件，無法支援腳本、外部資源和階層組織 |
| **Executor 凍結** | 無法聯合優化 executor 和 curator，可能存在校準偏差 |

### 6.2 作者提出的未來方向

1. **Agentic Search over Experiential Memory**：將靜態檢索改為主動的 agent 搜索，curator 可以發出多次查詢、根據中間證據重新調整，決定哪些技能應該被呈現、引用或組合。

2. **Hierarchical and Compositional Skills**：支持技能的分層分解——curator 學會不僅插入、更新、刪除技能，還學會鏈接、組合和抽象它們。

3. **Multi-Agent and Shared Memory**：將 SkillRepo 擴展到多 Agent 環境，支持共享經驗記憶，解決不同 Agent 之間的 credit assignment 問題。

---

## 七、總結

SkillOS 是一篇重要的論文，提出了一個**經驗驅動的 RL 訓練方法**來學習自我演化 Agent 的 skill curation 能力。其核心貢獻包括：

1. **模組化多代理架構**：凍結的 executor + 可訓練的 curator，簡化 curation 問題
2. **任務分組訓練**：提供長效用的學習訊號
3. **組合獎勵函數**：將延遲的任務反饋轉化為密集的 curation 學習訊號
4. **跨模型和跨任務的泛化能力**：即使使用較小的 8B curator 也超越大型閉源模型

這篇論文為自我演化 Agent 系統提供了務實且有效的方法論，其思想可以直接啟發包括 Hermes Agent 在內的各種 Agent 框架的技能管理機制。

---

## 八、參考資料

1. Ouyang, S., et al. (2026). "SkillOS: Learning Skill Curation for Self-Evolving Agents." arXiv:2605.06614.
2. Schulman, et al. "Proximal Policy Optimization Algorithms." arXiv:1707.06347.
3. Yao, et al. "ReAct: Synergizing Reasoning and Acting in Language Models." ICLR 2023.
4. Anthropic. "Agent Skills Overview." https://docs.anthropic.com/en/docs/agent-tools/skills-overview
5. Guo, et al. "DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via RL." arXiv:2501.12948.

---

*報告生成時間：2026-05-18*
*語言：繁體中文*
