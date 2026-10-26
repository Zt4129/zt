# Codex Skills

这个仓库收录两个教学场景用的 Codex Skill：

- `embedded-example-analysis`：对比嵌入式标准例程和学员工程，定位代码、配置和工程结构差异。
- `homework-review`：按学员提交顺序检查嵌入式、STM32、HAL、CubeMX、FreeRTOS、ADC、DMA 等作业，以证据为依据完成纠错、评分和学员反馈。

## 安装

在仓库根目录运行，把需要的 skill 目录复制到 Codex 的 skills 目录：

```powershell
Copy-Item -Recurse .\skills\embedded-example-analysis $env:USERPROFILE\.codex\skills\
Copy-Item -Recurse .\skills\homework-review $env:USERPROFILE\.codex\skills\
```

只安装一个时，只运行对应的一行。目录中的 `SKILL.md` 和 `references` 要一起复制。

## 使用：例程和学员工程比对

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

尽量同时提供作业要求、学员提交和工程环境信息：

```text
使用 homework-review skill 检查这份作业。

作业要求或评分口径：
<作业要求、课程文档或评分标准>

学员提交：
<飞书 /docx/ 或 /wiki/ 链接、本地文档、工程目录、截图、视频、日志或压缩包>

硬件与工具版本：
<开发板、芯片型号、Keil/CubeMX/SDK 版本，以及能否运行复现>

请按作业原顺序逐项检查，按 100 分制评分，给出可直接发给学员的纠错反馈和 Q&A，并判断是否为高质量作业候选。
```

检查飞书、Lark 或豆包在线文档时，直接提供完整链接即可，不必先下载。Skill 会先读取文档正文并锁定 revision；图片、视频和附件只有实际预览或检查后才作为证据。不能验证的内容会明确标出，并列出具体补交材料。

默认交付是一份新的飞书反馈文档，学员作业源文档不会被修改；评分和整体检查结论会在对话中单独说明。只有明确要求记录或追加 QA 时，才会更新统一 QA 总表。若希望直接在对话中返回纯文本反馈或指定本地文件，请在请求中说明。

## 使用建议

- 作业检查尽量同时提供作业要求、学员提交、源码和运行证据。
- 缺少部分材料时仍会按现有证据检查，并说明需要补交什么。
- 默认只检查，不修改学员代码；需要修复时请在请求中明确说明。
