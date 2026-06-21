# Codex Skills

这个仓库收录两个教学场景用的 Codex Skill：

- `embedded-example-analysis`：对比嵌入式标准例程和学员工程，定位代码、配置和工程结构差异。
- `homework-review`：检查嵌入式/STM32/FreeRTOS 作业，按证据链、Rubric 和高质量作业标准给出评分与反馈。

## 安装

把需要使用的 skill 目录复制到 Codex 的 skills 目录：

```powershell
Copy-Item -Recurse .\skills\embedded-example-analysis $env:USERPROFILE\.codex\skills\
Copy-Item -Recurse .\skills\homework-review $env:USERPROFILE\.codex\skills\
```

也可以只复制其中一个目录。

## 使用：例程和学员代码比对

对 Codex 说：

```text
使用 embedded-example-analysis skill 分析。

标准例程路径：
<标准例程路径>

学员工程路径：
<学员工程路径>

故障现象：
<例如：不跳转、无串口输出、HardFault、HAL_Delay 卡死>

输出：
对话摘要 + Markdown 报告

只分析，不修改代码。
```

这个 skill 会先收集配置，再建立标准例程参考模型，最后对比学员工程，输出带证据路径和修复建议的报告。

## 使用：作业检查

对 Codex 说：

```text
使用 homework-review skill 检查这份作业。

作业要求：
<作业要求或文档路径>

学员提交：
<docx、工程目录、截图、日志或压缩包路径>

请按 100 分制评分，输出反馈报告，并判断是否是高质量作业候选。
```

这个 skill 会检查材料完整性、知识点、截图/日志、代码片段、面试题和工程规范，按 100 分制给出维度分、扣分原因、纠错表、必改项和高质量候选判断。

## 建议

- 例程比对时一定要提供标准例程路径。
- 作业检查时尽量同时提供作业要求、学员文档、源码、运行日志或截图。
- 默认先“只分析，不修改代码”；确认问题后再让 Codex 修复。
