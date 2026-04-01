提取并翻译当前截帧中指定 Pass 的 Shader，生成带注释的 GLSL/HLSL 代码和中文功能说明文档。

用法：/rdc-shader [event_id]
若未提供 event_id，列出所有可用 Pass 让用户选择。

分析结果默认的保存路径为 Assets/RDC_Analysis/Shaders 目录。

先提供默认目录、用户手动输入路径，让用户选择保存的路径，如果保存的目录不存在，请先创建。

再执行 /rdc shader $ARGUMENTS 操作。
