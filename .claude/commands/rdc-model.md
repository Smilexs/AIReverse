提取当前截帧中指定 Draw Call 的几何数据信息，生成模型信息文档。

用法：/rdc-model [event_id]
若未提供 event_id，列出大模型 Draw Call（indices > 1000）供用户选择。

分析结果默认的保存路径为 Assets/RDC_Analysis/Models 目录。

先提供默认目录、用户手动输入路径，让用户选择保存的路径，如果保存的目录不存在，请先创建。

再执行 /rdc model $ARGUMENTS 操作。
