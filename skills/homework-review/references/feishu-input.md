# 飞书在线文档输入

## 目标

把飞书、Lark 或豆包的 `/docx/`、`/wiki/` 链接直接作为作业材料读取。优先在线读取；只有在线正文或关键素材无法可靠取得时才导出本地文件。下载或导出不能绕过登录、scope 或文档访问权限。

## 快速路径

1. 保留用户提供的完整 URL，不裁剪参数。
2. 直接尝试读取目标内容；不要先单独检查飞书桌面客户端，也不要每次先运行完整认证检查。
3. 当前环境提供 `lark-doc` 能力时优先使用；否则使用 `lark-cli docs +fetch`。
4. 首次成功后，在当前检查任务中复用已确认的客户端和身份状态；每个链接仍要分别验证是否可读。
5. 不保存自定义“本机已配置”标记。客户端配置和登录状态会变化，以实际读取结果为准。

先取目录并拿到 block ID 和 revision：

```bash
lark-cli docs +fetch --doc "<完整文档URL>" --scope outline --max-depth 3 --detail with-ids --as user
```

文档较短时读取整篇 `simple` 内容；文档较长时按目录读取与评分有关的章节，并固定首次返回的 `revision_id`：

```bash
lark-cli docs +fetch --doc "<完整文档URL>" --scope section \
  --start-block-id "<标题block_id>" --detail simple \
  --revision-id <首次revision_id> --as user
```

不要只读命中片段就判断整章完整。结果含 `<excerpt>` 时，根据 `top-block-id` 再读完整章节或范围。高质量候选必须覆盖全部评分维度对应的正文和证据。

## 图片、附件和嵌入资源

- 看到 `<img>`、`<source>` 或附件占位符时，只把它记录为“存在”，不要直接计入已验证证据。
- 只预览与运行结果、配置、日志、代码或问题复盘有关的图片；有 URL 时定向读取，没有 URL 时使用 `lark-cli docs +media-preview`。
- 只有附件内容影响评分时才下载；源码压缩包、日志和报告下载后按本 Skill 的本地材料流程检查。
- 遇到 `<sheet>`、`<bitable>` 或对应引用时，使用匹配的 Sheets/Base 能力读取内部数据；不要只呈现标签。
- 画板、视频或无法解析的复杂排版影响评分时，进入导出兜底。

## 实验效果视频核验

- 检查是否有对应实验效果视频和截图；视频/图片占位标签、文件名或 alt 描述只能证明材料条目存在，不能证明实验现象。
- 对视频 token 使用 `lark-cli docs +media-preview --token "<token>" --output ./work/<name>` 获取素材并读取返回的内容类型与路径。CLI 返回文件不等于已播放；继续用当前环境可用的视频预览或播放器实际查看相关片段。
- 至少核对能覆盖实验设置、关键操作、结果输出的片段；根据视频长度与作业目标查看异常/恢复过程是否需要验证。记录实际播放的内容范围及未能查看的部分。
- 若当前环境无法播放，标为视频内容未验证。结合可读截图、日志等证据继续检查，不把播放失败直接判作学员实验失败。

## 失败分流

### 客户端或配置不存在

若 `lark-cli` 命令不存在，先使用宿主已有的飞书文档连接能力。仍不可用时，说明需要一次性安装 `lark-cli`；不要把安装飞书桌面应用说成解决方案，也不要未经用户同意静默安装系统软件。

若 CLI 已安装但未配置，以后台方式启动 `lark-cli config init --new` 并读取输出。把命令产生的配置或授权 URL 原样发给用户，同时用 `lark-cli auth qrcode "<URL>" --output "./work/lark-auth.png"` 生成二维码；完成后继续原文档读取，不要求用户重新提供链接。

### 未登录、登录过期或缺少 scope

仅在读取失败后运行：

```bash
lark-cli auth status --json --verify
```

需要用户身份授权时先申请文档域权限：

```bash
lark-cli auth login --domain docs --no-wait --json
```

把返回的授权 URL 原样展示并生成二维码，本轮停止继续访问。用户明确完成授权后，由 Agent 执行 `lark-cli auth login --device-code "<本次device_code>"`，成功后重试原文档；授权链接或 device code 过期时重新发起，不持久化到项目文件。错误返回 `permission_violations`、`hint` 或 `console_url` 时，按错误给出的最小 scope 处理，不猜权限名；只有进入导出或权限申请流程时才补充 Drive 相关 scope。

### 文档无权访问或链接类型不匹配

- `permission denied`：说明当前登录账号和缺少的文档访问权。若要调用 `drive +apply-permission` 给 Owner 发送申请，必须先征得用户确认，因为这会发送通知。
- `not exist`：先检查完整 URL 和 token 类型；`/wiki/` 需要时用 `lark-cli drive +inspect --url "<完整URL>"` 确认底层对象，不要直接断言文档已删除。
- Wiki 底层是 Sheet、Base、Slides 或普通文件时，切换对应读取能力或进入导出流程。
- scope 已满足但租户策略、外部分享或文档密级仍阻止访问时，停止重复登录，让用户联系 Owner 授权或提供导出文件。
- 只对限流或临时网络错误有限重试一次；权限错误不要循环重试。

## 自动导出兜底

正文能读但复杂排版、图片或附件无法可靠检查时，先用 `drive +inspect` 取得底层 token 和类型，再把在线文档导出到当前任务目录下的相对路径，例如 `./work/feishu-export`：

```bash
lark-cli drive +export --token "<DOCX_TOKEN>" --doc-type docx \
  --file-extension pdf --output-dir ./work/feishu-export --as user
```

导出超时但返回 `ticket` 时，按返回的 `next_command` 继续查询，再用 `drive +export-download` 下载。优先 PDF 检查图文和版式；需要检索正文时可导出 Markdown。只有自动读取、权限申请和导出均不可用时，才请用户手动下载 DOCX/PDF 后提供。

## 版本与证据记录

记录以下内容后再评分：

- 文档标题、原始 URL、document ID。
- 首次读取并锁定的 revision ID。
- 实际读取的章节和范围。
- 实际预览或下载的图片、附件、嵌入资源。
- 未能验证的内容及原因。

报告提交前再次读取最新目录并比较 revision。revision 变化时，重新读取受影响内容；无法确认影响范围时重新检查整份材料。不要把旧 revision 的结论表述为当前最新版结论。
