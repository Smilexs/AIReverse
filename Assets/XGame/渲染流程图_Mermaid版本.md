# 渲染流程图 - 冰雪魅影 (Mermaid版本)

## 完整渲染管线流程图

```mermaid
flowchart TD
    Start([🎮 FRAME START<br/>Frame 12138])
    
    Start --> Compute[⚙️ COMPUTE SHADER<br/>Event 18 | 0.264ms 26%<br/>4x Storage Buffer 4MB<br/>GPU Culling / Instance Prep]
    
    Compute --> Pass1[🏔️ PASS #1: Main Scene<br/>1 Color + Depth Clear<br/>4 Draw Calls | 0.055ms<br/>━━━━━━━━━━━━━━━<br/>• Quad 6 idx<br/>• Geometry 32,823 idx × 2<br/>• Mesh 600 idx]
    
    Pass1 --> Pass2[🎨 PASS #2: UI/Effects<br/>1 Color + Depth Clear<br/>43 Draw Calls | 0.2ms 20%<br/>━━━━━━━━━━━━━━━<br/>• UI Elements 1024x2048 RGBA8<br/>• Hotspot: Event 95 0.075ms<br/>⚠️ Needs Batching]
    
    Pass2 --> PostFX[📦 POST-PROCESSING CHAIN<br/>6 Passes | 0.1ms]
    
    PostFX --> Pass3[✨ PASS #3: Bloom Extract<br/>1 Color | 1 Quad | 0.053ms<br/>HDR Buffer R16G16B16A16F]
    
    Pass3 --> Pass4[🌟 PASS #4: Bloom Blur 1<br/>1 Color | 1 Quad | 0.013ms]
    
    Pass4 --> Pass5[🌟 PASS #5: Bloom Blur 2<br/>1 Color | 1 Quad | 0.007ms]
    
    Pass5 --> Pass6[💫 PASS #6: Bloom Composite<br/>1 Color | 1 Quad | 0.007ms]
    
    Pass6 --> Pass7[🎭 PASS #7: Tone Mapping<br/>1 Color | 1 Quad | 0.009ms]
    
    Pass7 --> Pass8[🎨 PASS #8: Color Grading<br/>1 Color | 1 Quad | 0.012ms]
    
    Pass8 --> ScreenFX[📦 SCREEN SPACE EFFECTS<br/>3 Passes | 0.066ms]
    
    ScreenFX --> Pass9[🔆 PASS #9: SSAO Init<br/>1 Color Clear | 4 Verts | 0.053ms]
    
    Pass9 --> Pass10[🌫️ PASS #10: SSAO Blur<br/>1 Color Load | 4 Verts | 0.007ms]
    
    Pass10 --> Pass11[🖼️ PASS #11: SSAO Composite<br/>1 Color Load | 4 Verts | 0.006ms]
    
    Pass11 --> Pass12[🎯 PASS #12: UI Overlay<br/>1 Color + Depth Load<br/>7 Draw Calls | 0.124ms 12%<br/>━━━━━━━━━━━━━━━<br/>• Hotspot: Event 622 0.123ms]
    
    Pass12 --> Pass13[📐 PASS #13: Downsample<br/>1 Color Don't Care<br/>Draw 0.051ms + Blit 0.053ms<br/>Generate Mipmaps]
    
    Pass13 --> Pass14[🎬 PASS #14: Final Composite<br/>1 Color Load | 6 Ops | 0.037ms<br/>━━━━━━━━━━━━━━━<br/>• Composite 3 idx 0.027ms<br/>• 2x Buffer Copy<br/>• Debug Overlay FPS/Stats]
    
    Pass14 --> Present([📺 PRESENT<br/>vkQueuePresentKHR])
    
    style Compute fill:#9b59b6,stroke:#8e44ad,stroke-width:3px,color:#fff
    style Pass1 fill:#3498db,stroke:#2980b9,stroke-width:2px,color:#fff
    style Pass2 fill:#e74c3c,stroke:#c0392b,stroke-width:3px,color:#fff
    style Pass3 fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style Pass4 fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style Pass5 fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style Pass6 fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style Pass7 fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style Pass8 fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style Pass9 fill:#1abc9c,stroke:#16a085,stroke-width:2px,color:#fff
    style Pass10 fill:#1abc9c,stroke:#16a085,stroke-width:2px,color:#fff
    style Pass11 fill:#1abc9c,stroke:#16a085,stroke-width:2px,color:#fff
    style Pass12 fill:#e74c3c,stroke:#c0392b,stroke-width:3px,color:#fff
    style Pass13 fill:#34495e,stroke:#2c3e50,stroke-width:2px,color:#fff
    style Pass14 fill:#34495e,stroke:#2c3e50,stroke-width:2px,color:#fff
    style Present fill:#27ae60,stroke:#229954,stroke-width:2px,color:#fff
    style Start fill:#2ecc71,stroke:#27ae60,stroke-width:2px,color:#fff
    style PostFX fill:#95a5a6,stroke:#7f8c8d,stroke-width:1px,color:#fff
    style ScreenFX fill:#95a5a6,stroke:#7f8c8d,stroke-width:1px,color:#fff
```

## 简化流程图（高层架构）

```mermaid
flowchart LR
    A[Frame Start] --> B[Compute<br/>预处理<br/>0.26ms]
    B --> C[主场景<br/>Pass #1<br/>0.06ms]
    C --> D[UI层<br/>Pass #2<br/>0.20ms]
    D --> E[后处理<br/>Pass #3-8<br/>0.10ms]
    E --> F[屏幕效果<br/>Pass #9-11<br/>0.07ms]
    F --> G[UI叠加<br/>Pass #12<br/>0.12ms]
    G --> H[下采样<br/>Pass #13<br/>0.10ms]
    H --> I[最终合成<br/>Pass #14<br/>0.04ms]
    I --> J[Present]
    
    style B fill:#9b59b6,stroke:#8e44ad,stroke-width:3px,color:#fff
    style C fill:#3498db,stroke:#2980b9,stroke-width:2px,color:#fff
    style D fill:#e74c3c,stroke:#c0392b,stroke-width:3px,color:#fff
    style E fill:#f39c12,stroke:#e67e22,stroke-width:2px,color:#fff
    style F fill:#1abc9c,stroke:#16a085,stroke-width:2px,color:#fff
    style G fill:#e74c3c,stroke:#c0392b,stroke-width:3px,color:#fff
    style H fill:#34495e,stroke:#2c3e50,stroke-width:2px,color:#fff
    style I fill:#34495e,stroke:#2c3e50,stroke-width:2px,color:#fff
    style J fill:#27ae60,stroke:#229954,stroke-width:2px,color:#fff
```

## 性能热点分布图

```mermaid
pie title 帧时间分布 (1.01ms total)
    "Compute预处理" : 26.1
    "UI Pass #2" : 19.8
    "UI Pass #12" : 12.3
    "Pass #13 Blit" : 10.3
    "后处理链" : 9.9
    "屏幕空间效果" : 6.5
    "主场景Pass #1" : 5.4
    "Pass #14合成" : 3.7
    "其他" : 6.0
```

## 渲染Pass依赖关系图

```mermaid
graph TD
    subgraph Geometry["几何渲染阶段"]
        P1[Pass #1<br/>主场景]
        P2[Pass #2<br/>UI/Effects]
    end
    
    subgraph PostProcessing["后处理阶段"]
        P3[Pass #3 Bloom Extract]
        P4[Pass #4 Blur 1]
        P5[Pass #5 Blur 2]
        P6[Pass #6 Composite]
        P7[Pass #7 ToneMap]
        P8[Pass #8 ColorGrade]
    end
    
    subgraph ScreenSpace["屏幕空间阶段"]
        P9[Pass #9 SSAO Init]
        P10[Pass #10 SSAO Blur]
        P11[Pass #11 SSAO Comp]
    end
    
    subgraph Final["最终合成阶段"]
        P12[Pass #12 UI Overlay]
        P13[Pass #13 Downsample]
        P14[Pass #14 Composite]
    end
    
    Compute[Compute Shader] --> P1
    P1 -->|Color+Depth| P2
    P2 -->|Color| P3
    P3 --> P4
    P4 --> P5
    P5 --> P6
    P6 --> P7
    P7 --> P8
    P8 --> P9
    P9 --> P10
    P10 --> P11
    P11 -->|Load| P12
    P12 --> P13
    P13 --> P14
    P14 --> Present[Present]
    
    P1 -.->|Depth| P12
    
    style Compute fill:#9b59b6,color:#fff
    style P2 fill:#e74c3c,color:#fff
    style P12 fill:#e74c3c,color:#fff
    style Present fill:#27ae60,color:#fff
```

## 资源数据流图

```mermaid
flowchart TD
    subgraph Buffers["存储缓冲区 (20个)"]
        SSBO1[Buffer 50581<br/>4MB SSBO]
        SSBO2[Buffer 49677<br/>4MB SSBO]
        SSBO3[Buffer 50663<br/>4MB SSBO]
        VBO[Vertex/Index<br/>Buffers]
        UBO[Uniform<br/>Buffers]
    end
    
    subgraph Textures["纹理资源 (50个)"]
        UITex[UI Textures<br/>1024x2048 RGBA8]
        HDR[HDR Buffer<br/>1080x1920 R16F]
        RT1[Render Target 1]
        RT2[Render Target 2]
        Depth[Depth Buffer]
    end
    
    Compute[Compute Shader] -->|Read/Write| SSBO1
    Compute -->|Read/Write| SSBO2
    Compute -->|Read/Write| SSBO3
    
    SSBO1 -.->|Instance Data| Pass1[Pass #1 Scene]
    VBO --> Pass1
    Pass1 -->|Output| RT1
    Pass1 -->|Output| Depth
    
    RT1 --> Pass2[Pass #2 UI]
    Depth --> Pass2
    UITex --> Pass2
    Pass2 -->|Output| HDR
    
    HDR --> PostFX[Post-Processing<br/>Pass #3-8]
    PostFX -->|Ping-Pong| RT2
    RT2 -->|Ping-Pong| HDR
    
    HDR --> ScreenFX[Screen Space FX<br/>Pass #9-11]
    
    ScreenFX --> Pass12[Pass #12 UI Overlay]
    Depth -.->|Load| Pass12
    
    Pass12 --> Blit[Pass #13 Blit]
    Blit --> Final[Pass #14 Final]
    Final --> Swapchain[Swapchain]
    
    style Compute fill:#9b59b6,color:#fff
    style Pass2 fill:#e74c3c,color:#fff
    style Pass12 fill:#e74c3c,color:#fff
    style PostFX fill:#f39c12,color:#fff
    style ScreenFX fill:#1abc9c,color:#fff
```

## GPU时间线（按比例）

```
0ms                    0.26ms              0.46ms         0.56ms    0.62ms    0.74ms     0.85ms  0.89ms  1.01ms
|                         |                   |              |         |         |          |       |       |
├─────────────────────────┼───────────────────┼──────────────┼─────────┼─────────┼──────────┼───────┼───────┤
│    COMPUTE SHADER       │    UI Pass #2     │  Post FX     │  SSAO   │  Pass12 │  Pass13  │ P14   │ Other │
│      (26.1%)            │      (19.8%)      │   (9.9%)     │ (6.5%)  │ (12.3%) │ (10.3%)  │(3.7%) │ (6%)  │
└─────────────────────────┴───────────────────┴──────────────┴─────────┴─────────┴──────────┴───────┴───────┘
         🔴 HOT                   🔴 HOT                                     🔴 HOT

图例:
🔴 性能热点 (>10% frame time)
🟢 优化良好 (<5% frame time)
```

## 优化优先级矩阵

```mermaid
quadrantChart
    title 优化优先级分析 (影响 vs 难度)
    x-axis 实施难度低 --> 实施难度高
    y-axis 性能影响小 --> 性能影响大
    
    quadrant-1 "高优先级"
    quadrant-2 "快速优化"
    quadrant-3 "低优先级"
    quadrant-4 "长期规划"
    
    Compute优化: [0.6, 0.9]
    UI Batching: [0.3, 0.8]
    Shader优化: [0.5, 0.7]
    合并PostFX: [0.4, 0.5]
    减少Blit: [0.6, 0.4]
    LOD优化: [0.8, 0.3]
```

## 渲染管线架构总览

```mermaid
graph TB
    subgraph Input["输入阶段"]
        CPU[CPU Submit]
        CS[Compute Shader<br/>GPU Culling]
    end
    
    subgraph GeomPass["几何通道"]
        Opaque[不透明物体<br/>Pass #1]
        UI1[UI/特效<br/>Pass #2]
    end
    
    subgraph PostPass["后处理通道"]
        Bloom[Bloom提取与模糊<br/>Pass #3-6]
        ToneMap[色调映射与调色<br/>Pass #7-8]
    end
    
    subgraph ScreenPass["屏幕空间通道"]
        SSAO[环境光遮蔽<br/>Pass #9-11]
    end
    
    subgraph CompPass["合成通道"]
        UIOver[UI叠加<br/>Pass #12]
        Down[下采样/Mipmap<br/>Pass #13]
        Comp[最终合成<br/>Pass #14]
    end
    
    subgraph Output["输出阶段"]
        Present[Present<br/>显示输出]
    end
    
    CPU --> CS
    CS --> Opaque
    Opaque --> UI1
    UI1 --> Bloom
    Bloom --> ToneMap
    ToneMap --> SSAO
    SSAO --> UIOver
    UIOver --> Down
    Down --> Comp
    Comp --> Present
    
    style CS fill:#9b59b6,color:#fff
    style UI1 fill:#e74c3c,color:#fff
    style UIOver fill:#e74c3c,color:#fff
    style Bloom fill:#f39c12,color:#fff
    style ToneMap fill:#f39c12,color:#fff
    style SSAO fill:#1abc9c,color:#fff
    style Present fill:#27ae60,color:#fff
```

---

## 使用说明

1. **查看HTML版本**: 双击 `渲染流程图_冰雪魅影.html` 在浏览器中打开，可交互、可缩放
2. **查看Mermaid版本**: 在支持Mermaid的Markdown编辑器中查看此文件
3. **GitHub/GitLab**: 自动渲染Mermaid图表
4. **VS Code**: 安装 "Markdown Preview Mermaid Support" 插件

---

*生成时间: 2026-03-31*  
*工具: RenderDoc + Claude AI*
