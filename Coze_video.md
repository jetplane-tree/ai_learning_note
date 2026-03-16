---
tags:
  - 学习/AI/笔记
created: 2026-03-16
---
# 学习笔记

## Day1 - 了解
### AI视频自动化实现原理
- 视频导入剪映时会生成草稿文件夹，包含两个核心文件：
	- draft_content.json 存储配置信息（时间线，素材引用，特效参数）
	- draft_meta_info.json 存储草稿基本信息
- 剪映通过json来还原剪辑的视频 => 工作流来生成json文件
### 流程和分工
 - 工作流：LLM生成文案、LLM生成图片/语音、构建timeline、调用插件组装
	 - 调用插件组装包括：
		 - create_draft创建草稿（空白画布）
		 - add_* 添加素材（视频/音频/图片/字幕等）
		 - save_draft 保存草稿生成json
	 - output：组装草稿json （在云端）
 - 剪映小助手：接收工作流云端输出
	 - 把云端草稿下载到本地
	 - 把json内的云端路径改写成本地路径
	 - 目录同步
	 - output：替换为本地路径的草稿json
- 剪映：读取本地json，渲染视频画面，导出视频

### 开源项目
- [capcut-mate-electron](https://github.com/Hommy-master/capcut-mate-electron) — 剪映小助手的核心
- [pyJianYingDraft](https://github.com/GuanYixuan/pyJianYingDraft) — 剪映草稿格式解析