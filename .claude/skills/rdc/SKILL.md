---
name: rdc
description: |
  RenderDoc截帧逆向分析工具集。用于分析 .rdc 捕获文件，提取渲染流程、Shader、贴图、模型等资产。

  触发场景（只要涉及以下任一情况，必须使用此 Skill）：
  - 用户输入 /rdc 开头的命令（/rdc analyze、/rdc shader、/rdc texture、/rdc model）
  - 用户提到"分析截帧"、"分析rdc"、"提取shader"、"反编译shader"、"提取贴图"、"提取模型"
  - 用户提到 RenderDoc、.rdc 文件、截帧分析、渲染逆向
  - 用户想了解某个游戏/应用的渲染流程、Pass结构、Shader逻辑
---

# RDC 截帧分析 Skill

## 子命令

| 命令 | 说明 |
|------|------|
| `/rdc analyze` | 分析当前截帧的完整渲染流程，生成 MD报告 + HTML流程图 |
| `/rdc shader [event_id]` | 提取并翻译指定事件的 Shader（GLSL/HLSL + 功能说明文档） |
| `/rdc texture [event_id]` | 提取指定事件绑定的贴图，生成资源清单 |
| `/rdc model [event_id]` | 提取指定 Draw Call 的几何信息 |

用户可以不带参数只写 `/rdc`，此时询问他想执行哪个操作。

---

## 输出目录规则

所有输出统一存放到 `Assets/XGame/`（或用户指定目录）下，按游戏名归类：

```
Assets/XGame/
├── 截帧分析_[游戏名].md            ← /rdc analyze 生成
├── 渲染流程图_[游戏名].html         ← /rdc analyze 生成
├── Shaders/
│   ├── event_[id]_vs.glsl          ← Vertex Shader（含注释）
│   ├── event_[id]_ps.glsl          ← Pixel Shader（含注释）
│   ├── event_[id]_shader.shader    ← 完整的 unity urp可用shader（含注释）
│   └── event_[id]_说明.md          ← 中文功能说明
├── Textures/
│   └── 贴图清单_[游戏名].md         ← /rdc texture 生成
└── Models/
    └── event_[id]_mesh_info.md     ← /rdc model 生成
```

**游戏名获取规则**：
1. 检查 memory 中是否已保存该 rdc 文件对应的游戏名
2. 若没有，询问用户，然后保存到 memory（key: `rdc_game_name: <rdc文件名前缀>`）
3. 若已有分析文件则追加，不覆盖

---

## /rdc analyze — 渲染流程分析

### 执行步骤

**Step 1: 确认状态**
调用 `mcp__renderdoc__get_capture_status`，确认有 rdc 已加载。若未加载，提示用户在 RenderDoc 中打开捕获文件。

**Step 2: 并行收集帧数据**
同时调用以下三个工具：
- `mcp__renderdoc__get_frame_summary` — 总体统计
- `mcp__renderdoc__get_draw_calls` — 完整 Pass 树（含子节点）
- `mcp__renderdoc__get_action_timings` — 所有操作的 GPU 耗时

**Step 3: 分析关键 Pass（并行）**
对耗时排名前5的 Draw Call 和每个顶层 Pass 的第一个 Draw Call，并行调用 `mcp__renderdoc__get_pipeline_state(event_id)` 获取 Shader 绑定和资源信息。

**Step 4: 推断 Pass 语义**
根据以下规则判断每个 Pass 的渲染目的：
- `Clear + Depth`：主场景几何 Pass 或阴影 Pass
- `Don't Care`：后处理中间 Pass（输入不重要，只关心输出）
- `Load + Depth`：叠加渲染（保留上一 Pass 结果）
- 仅1个 Draw Call + 4-6 indices + 无 Depth：全屏后处理 quad
- 大量 Draw Call（>10）+ 小 indices：UI 层
- Compute Dispatch + 多个 SSBO：GPU Culling / 实例化准备

**Step 5: 生成两个输出文件**
参考 `references/analyze_template.md` 的格式生成报告和流程图。

HTML 流程图**必须**参照 `references/flowchart_template.html` 的样式和交互方案生成：
- 复制模板中完整的 CSS 样式和 JS 交互逻辑（Tooltip、点击高亮依赖链、类型筛选栏、时间线、资源依赖分组卡片、滚轮缩放、键盘提示）
- 替换所有 `[PLACEHOLDER: xxx]` 标注的位置为实际分析数据
- 填充 JS 中的 `passes` 数组和 `groups` 数组（字段规范见模板注释）
- SVG 节点的 `data-*` 属性必须完整，供 JS 交互使用
- 颜色严格遵循模板规范：UI 层用靛蓝 `#5b6abf`，红色 `#e74c3c` 仅用于性能热点标记
- 界面文字以中文为主，渲染专有名词保持英文

### 报告必须包含

- 总体统计表（Pass 数、Draw Call 数、总耗时、资源数量）
- 逐 Pass 详情（配置、Draw Call 数、耗时、Shader 绑定、资源格式）
- 性能热点分析（Top 5，含耗时百分比）
- 渲染架构特征总结（Forward/延迟、后处理链、Compute 用途）
- 优化建议（高/中/低优先级）

---

## /rdc shader — Shader 提取与翻译

### 执行步骤

**Step 1: 确认 event_id**
若未提供，列出所有顶层 Pass 及其耗时，让用户选择感兴趣的 Pass。

**Step 2: 获取 Shader**
对该 event 的 VS 和 PS 分别调用：
```
mcp__renderdoc__get_shader_info(event_id, "vertex")
mcp__renderdoc__get_shader_info(event_id, "pixel")
```

**Step 3: 翻译与注释**
将反编译结果整理后：
- 为每个 uniform/varying 变量添加语义注释（推断其含义）
- 识别并标注常见渲染模式（见下方"识别模式"）
- 保留原始变量名，注释写在同行或上方

**识别的渲染模式**：
```
MVP 矩阵        → // [变换] 模型-视图-投影矩阵
贴图坐标        → // [UV] 贴图采样坐标
颜色调制        → // [颜色] Alpha/RGB 调制因子
时间变量        → // [动画] 用于 UV 动画或顶点动画
Lambert 光照    → // [光照] 漫反射光照计算
Alpha 测试      → // [透明] 丢弃低透明度像素
Bloom 提取      → // [后处理] 提取高亮区域
Tone Mapping    → // [后处理] HDR 转 LDR
```

**Step 4: 生成文件**
- `Shaders/event_[id]_vs.glsl` — 带注释的顶点 Shader
- `Shaders/event_[id]_ps.glsl` — 带注释的像素 Shader
- `Shaders/event_[id]_shader.shader` — 完整的 unity urp可用shader（含注释）
- `Shaders/event_[id]_说明.md` — 中文说明文档，包含：
  - 一句话功能概述
  - 输入/输出变量说明表
  - 核心算法分析
  - Unity ShaderLab 等价实现思路（可选）

---

## /rdc texture — 贴图资源提取

### 执行步骤

**Step 1: 确认范围**
- 若提供了 event_id：获取该 Draw Call 的管线状态，列出绑定的纹理
- 若未提供：获取整帧所有 Pass 的管线状态，汇总所有唯一纹理

**Step 2: 获取贴图元信息**
对每个纹理资源调用 `mcp__renderdoc__get_texture_info(resource_id)`

**Step 3: 推断用途**
| 特征 | 推断用途 |
|------|---------|
| 1080x1920 R16G16B16A16F | HDR 渲染缓冲 |
| 1x1 RGBA8 | 常量颜色 / LUT |
| 宽高比 1:2，RGBA8，无 Mip | UI 图集 |
| 方形大尺寸，有 Mip | 场景/角色贴图 |
| 与 Render Target 同尺寸 | 后处理中间缓冲 |

**Step 4: 生成贴图清单**
输出 `Textures/贴图清单_[游戏名].md`，格式见 `references/analyze_template.md`。

---

## /rdc model — 几何数据提取

### 执行步骤

**Step 1: 确认 event_id**
若未提供，列出 indices 数量较大的 Draw Call（通常 >1000 为主要模型）。

**Step 2: 获取 Draw Call 详情**
调用 `mcp__renderdoc__get_draw_call_details(event_id)` 和 `mcp__renderdoc__get_pipeline_state(event_id)`。

**Step 3: 生成模型信息文档**
输出 `Models/event_[id]_mesh_info.md`，包含：
- 顶点数 / 索引数 / 三角面数
- 图元类型（TriangleList / TriangleStrip 等）
- 绑定的 Vertex Buffer 和 Index Buffer 资源 ID
- Vertex Shader 输入属性推断（position、normal、uv、color 等）
- 推断的模型类型（角色 / 场景 / UI / 特效）

---

## 交互原则

- **首次遇到新游戏**：询问游戏名并保存到 memory
- **操作完成后**：告知生成了哪些文件及完整路径
- **rdc 未加载时**：直接提示，不做其他操作
- **event_id 无效时**：列出可用 Draw Call 供选择
- **所有文档输出使用中文**

## 参考文件

- `references/analyze_template.md` — 分析报告和流程图的完整格式模板
- `references/flowchart_template.html` — HTML 流程图的样式、交互和数据结构模板
