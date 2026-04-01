# 分析报告 & 流程图模板

## 一、MD 分析报告模板

```markdown
# RenderDoc截帧渲染流程分析 - [游戏名]

**捕获信息**:
- 文件: `[rdc文件名]`
- 图形API: [Vulkan / D3D11 / D3D12 / OpenGL]
- 分析日期: [YYYY-MM-DD]

---

## 总体概况

| 指标 | 数值 |
|------|------|
| 图形API | [API] |
| 总操作数 | [N]个 action |
| Draw Call数量 | [N]个 |
| Compute Dispatch | [N]个 |
| 渲染Pass | [N]个 |
| 纹理资源 | [N]个 |
| 缓冲区资源 | [N]个 |
| 总GPU耗时 | ~[X]ms |

---

## 渲染流程详解

### [N]. [Pass名称] (Event [起始]-[结束])

**配置**: [Color Target数] Color Target[s] [+ Depth Buffer] ([Clear/Load/Don't Care]模式)

**Draw Call列表**:

| Event | Indices | GPU Time | 说明 |
|-------|---------|----------|------|
| [id] | [n] | [x]ms | [推断用途] |

**Shader信息**:
- Vertex Shader: `ResourceId::[id]`
  - Uniform: [size] bytes, [描述]
- Pixel Shader: `ResourceId::[id]`
  - 采样纹理: [格式, 尺寸]

**Pass总耗时**: ~[X]ms

---

## 性能热点分析

### Top 5 最耗时操作

| 排名 | Event | 操作 | 耗时 | 占比 | 优化建议 |
|------|-------|------|------|------|----------|
| 1 | [id] | [名称] | [x]ms | [x]% | [建议] |

### Pass级别耗时分布

```
[Pass名]:  ████░░░░ [x]% ([x]ms)
[Pass名]:  ██░░░░░░ [x]% ([x]ms)
```

---

## 渲染架构特征

1. **渲染类型**: [Forward / Deferred / Forward+]
2. **后处理链**: [Pass数量和效果]
3. **Compute用途**: [推断]
4. **资源特征**: [纹理格式、HDR等]

---

## 优化建议

### 🔴 高优先级
- [具体建议，说明节省量]

### 🟡 中优先级
- [具体建议]

### 🟢 低优先级
- [具体建议]

---

## 附录: 关键资源ID

### 主要纹理
- `ResourceId::[id]`: [尺寸] [格式] ([推断用途])

### 主要Shader
- `ResourceId::[id]` / `[id]`: [Pass名] VS/PS

---
*分析工具: RenderDoc | 日期: [YYYY-MM-DD] | 工具: Claude AI*
```

---

## 二、贴图清单模板

```markdown
# 贴图资源清单 - [游戏名]

**分析时间**: [YYYY-MM-DD]
**来源帧**: [rdc文件名]

## 完整贴图列表

| # | Resource ID | 名称 | 尺寸 | 格式 | Mip层 | MSAA | 推测用途 | 使用Pass |
|---|-------------|------|------|------|-------|------|---------|---------|
| 1 | [id] | [name] | [WxH] | [fmt] | [n] | [n] | [用途] | [Pass] |

## 分类统计

| 类型 | 数量 |
|------|------|
| 渲染目标 (Render Target) | [n] |
| UI贴图 | [n] |
| 场景/角色贴图 | [n] |
| 后处理缓冲 | [n] |
| 其他 | [n] |
```

---

## 三、HTML 流程图结构要求

**参照模板**: `references/flowchart_template.html` — 包含完整的 CSS 样式和 JS 交互逻辑。

生成 HTML 流程图时，复制模板的完整 CSS 和 JS 代码，替换所有 `[PLACEHOLDER: xxx]` 标注的位置为实际数据。

### 颜色规范

| Pass 类型 | 颜色 | gradient ID |
|-----------|------|-------------|
| Compute Shader | 紫色 `#9b59b6` | `grad-compute` |
| 主场景几何 | 蓝色 `#3498db` | `grad-scene` |
| UI 层 | 靛蓝 `#5b6abf` | `grad-ui` |
| 后处理效果 | 橙色 `#f39c12` | `grad-postfx` |
| 屏幕空间效果 | 青绿 `#1abc9c` | `grad-screen` |
| 合成/输出 | 深灰 `#34495e` | `grad-composite` |
| 阴影 | 灰色 `#7f8c8d` | `grad-shadow` |
| Present | 绿色 `#27ae60` | `grad-present` |
| **性能热点标记** | **红色 `#e74c3c`** | 仅用于侧边条和热点标注文字 |

> 注意：红色 `#e74c3c` **仅用于性能热点标记**（侧边条 + 百分比文字），不作为任何 Pass 类型的底色。

### 必须包含的交互组件

1. **顶部统计栏** — Pass 数、Draw Call 数、Dispatch 数、GPU 总耗时、纹理数、缓冲区数
2. **类型筛选栏** — 按钮筛选不同类型的 Pass（全部 / Compute / 场景 / UI / 后处理 / 屏幕空间 / 合成输出）
3. **SVG 流程图** — 纵向排列每个 Pass，带颜色编码和箭头连接
4. **丰富 Tooltip** — 悬停显示 GPU 耗时、Draw Calls、Dispatches、详情、依赖列表、百分比条
5. **点击高亮依赖链** — 点击任意 Pass，高亮其上下游依赖，非相关节点变暗
6. **性能时间线** — 横向条形图，可悬停查看详情，可点击跳转
7. **资源依赖分组卡片** — 按渲染阶段分组，每组显示名称 + 总耗时，组内列出 Pass 及其 Buffer 依赖类型
8. **图例说明** — 颜色对应含义
9. **关键指标** — 性能热点 Top 3 + 优化良好部分
10. **优化建议** — 高/中优先级 + 预期效果
11. **总体评价** — 一段话总结
12. **键盘提示** — 点击/Esc/滚轮

### 数据驱动说明

流程图的时间线和资源依赖卡片由 JS 从两个数据数组自动生成：

**passes 数组** — 每个 Pass 一项：

```javascript
{
    id: 'pass1',              // 唯一标识
    label: 'Pass 1: 主场景',  // 显示名称（中文 + 英文专有名词）
    time: 0.055,              // GPU 耗时 (ms)
    color: '#3498db',         // 按类型颜色
    type: 'scene',            // 类型标签
    deps: ['compute'],        // 依赖的上游 Pass id
    depType: ['SSBO']         // 对应的 Buffer 类型: Color | Depth | SSBO
}
```

**groups 数组** — 按渲染阶段分组：

```javascript
{
    label: '后处理链 Post-Processing',  // 分组标题
    color: '#f39c12',                    // 标题颜色
    ids: ['pass3', 'pass4', 'pass5']     // 组内 Pass id
}
```

### SVG Pass 节点格式

每个 Pass 在 SVG 中的节点必须包含以下 data 属性供 JS 交互使用：

```html
<g class="node" data-pass="[id]" data-type="[type]"
   data-time="[ms]" data-pct="[百分比]"
   data-draws="[N]" data-dispatches="[N]"
   data-details="[详细信息，用 | 分隔多行]">
```

性能热点 Pass（耗时 >10%）额外添加红色侧边条：
```html
<rect x="820" y="[Y+5]" width="20" height="[H-10]" fill="#e74c3c" opacity="0.6" rx="3"/>
<text fill="#e74c3c" font-size="11" font-weight="bold" opacity="0.7">[X]%</text>
```

---

## 四、Shader 说明文档模板

```markdown
# Shader 说明文档 - Event [id] | [Pass名]

**类型**: [Vertex Shader / Pixel Shader]
**Resource ID**: `ResourceId::[id]`
**所属Pass**: Pass #[N] [Pass语义名]

---

## 功能概述

[一句话描述这个 Shader 做什么]

---

## 输入/输出

### 顶点属性输入 (Vertex Attributes)
| 变量名 | 类型 | 语义 | 说明 |
|--------|------|------|------|
| [name] | [type] | [semantic] | [说明] |

### Uniform 参数
| 变量名 | 类型 | 推断含义 |
|--------|------|---------|
| [name] | [type] | [含义] |

### 采样贴图 (仅 Pixel Shader)
| 槽位 | Resource ID | 尺寸 | 格式 | 推断用途 |
|------|-------------|------|------|---------|
| [n] | [id] | [WxH] | [fmt] | [用途] |

---

## 核心算法分析

[分析 Shader 实现的渲染技术，如光照模型、后处理算法等]

---

## Unity ShaderLab 等价实现思路

```hlsl
Shader "[推断名称]" {
    Properties { ... }
    SubShader {
        Pass {
            CGPROGRAM
            // [关键实现要点]
            ENDCG
        }
    }
}
```
```
