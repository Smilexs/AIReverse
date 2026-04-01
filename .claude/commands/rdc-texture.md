提取当前截帧中的贴图资源，生成贴图清单（Resource ID、尺寸、格式、推断用途）。

用法：/rdc-texture [event_id]
若未提供 event_id，提取整帧所有贴图，要先提示用户，用户确认后方可继续。

分析结果默认的保存路径为 Assets/RDC_Analysis/Textures 目录。

先提供默认目录、用户手动输入路径，让用户选择保存的路径，如果保存的目录不存在，请先创建。

再执行 /rdc texture $ARGUMENTS 操作。
