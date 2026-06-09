---
name: issue-creator
description: 根据简短的测试人员记录，为 QA 和产品测试创建高质量的 GitHub issue。当用户想在 GitHub 中报告、起草、润色、规范化或创建 bug/feature issue 时使用，尤其适用于集中式 labring-sigs/sealos-issues 工作流，其中包括最少测试人员输入、gh CLI 预检、敏感信息检查、Priority/Effort/Type/Product/Environment 等 GitHub Project 字段，以及对可复用 issue catalog 的受控更新。
---

# Issue Creator

把一条简短的测试人员记录，转成一份完整的 GitHub issue，让测试人员低摩擦提交，同时给开发者提供高信号信息。

默认目标：`labring-sigs/sealos-issues`。加载 `assets/catalog.json` 以获取当前产品、环境、标签、里程碑、GitHub Issue 字段、Project 字段名和优先级规则。在处理 Sealos issue 或更新 catalog 时阅读 `references/sealos-issues.md`。在起草最终正文时阅读 `references/issue-template.md`。

## 核心规则

尽量少向测试人员追问。先从他们的记录中推断结构，然后预览草稿，并且只询问缺失的、会阻碍形成有用 issue 的信息。

在用户确认预览之前，不要创建真实的 GitHub issue。使用 `scripts/sealos_issue.py draft` 做 dry-run，只在确认后使用 `scripts/sealos_issue.py draft --create`。

## 最小输入

测试人员可能只提供：

- 环境：`10.70`、`12.38`、`bja`、`public cloud`、一个 URL，或一个集群名称。
- 产品/页面：近似名称，例如 `maestro`、`devbox`、`对象存储`、`应用管理`，或截图上下文。
- 操作：他们做了什么。
- 实际结果：出了什么问题。
- 证据：截图、日志、API URL、payload、response，或可用的录屏。

如果其中一项缺失但 issue 仍然可以理解，继续处理，并将该字段标记为推断或未知。当缺失信息会让 issue 不可用时，最多问 1-2 个简短问题，通常是产品或环境。

## 工作流

1. 如果可能会创建 issue，先运行预检：

   ```bash
   python3 skills/issue-creator/scripts/sealos_issue.py preflight
   ```

   如果缺少 `gh`，告诉用户安装 GitHub CLI。在 macOS 上，建议 `brew install gh`。如果 `gh` 未登录，建议 `gh auth login`。如果需要 Project 字段且权限失败，建议 `gh auth refresh -s project` 或 gh 错误中给出的确切 scope。
   预检还会检查是否有可用的图片上传器：PicList 本地服务、PicGo CLI，或 catalog 中的 `image_uploader.command`。

2. 从原始记录起草：

   ```bash
   python3 skills/issue-creator/scripts/sealos_issue.py draft --note "<tester note>"
   ```

3. 审查生成的标题、正文、标签、里程碑、原生 GitHub issue 元数据，以及可选的 project 字段。自行修正明显的推断错误。使用 catalog 处理已知别名，使用模板确定正文形态。

4. 创建前阻止或打码敏感信息：
   - 密码、token、access key、authorization header、私有凭据，或看起来像长凭据的值
   - 替换为 `见内部渠道` 或 `[REDACTED]`
   - 永远不要把测试密码发布到公开 issue

5. 只在用户确认后创建：

   ```bash
   python3 skills/issue-creator/scripts/sealos_issue.py draft --note "<tester note>" --create
   ```

   脚本会在创建后尽可能设置原生 GitHub Issue Type 和 Issue Fields。只有当还需要更新单独的 GitHub Project 时，才添加 `--project-number <number>`。

   如果测试人员提供了本地截图，用重复的 `--image` 参数传入：

   ```bash
   python3 skills/issue-creator/scripts/sealos_issue.py draft --note "<tester note>" --image "/path/to/screenshot.png" --create
   ```

   脚本会在创建 issue 前上传图片，并把 Markdown 图片 URL 插入到 `## 证据` 中。它永远不会发布本地文件系统路径。对于 dry-run 图片上传测试，添加 `--upload-images`；否则预览会显示 `本地截图待上传：<filename>`。

   如果截图在 macOS 剪贴板上，并且 PicList 本地服务正在运行，使用 `--clipboard-image`，而不是保存临时文件：

   ```bash
   python3 skills/issue-creator/scripts/sealos_issue.py draft --note "<tester note>" --clipboard-image --upload-images
   python3 skills/issue-creator/scripts/sealos_issue.py draft --note "<tester note>" --clipboard-image --create
   ```

   没有 `--upload-images` 或 `--create` 时，剪贴板预览会显示 `剪贴板截图待上传`，并且不会调用 PicList。PicList 通过空 JSON body 的 `POST /upload` 上传剪贴板内容，因此系统剪贴板必须包含实际图片数据。

6. 如果新的产品、环境、里程碑或 Project 字段选项被确认为持久可用，以受控方式更新 catalog：

   ```bash
   python3 skills/issue-creator/scripts/sealos_issue.py draft --note "<tester note>" --learn
   ```

   学习前优先询问。不要学习拼写错误、临时环境或一次性表述。

## 标题规则

使用：

```text
<产品或模块>: <具体的用户可见问题>
```

示例：

- `Devbox: 公网地址协议为 grpcs/wss 时内网协议错误`
- `GPU Maestro: 创建沙箱后 VS Code 连接超时`
- `AppLaunchpad: PVC 扩容接口返回 500`
- `对象存储: 长桶名导致权限和更多操作按钮不可见`

不要在标题中添加 `bug`。把 Bug/Feature 放在 Issue Type 或 Project `Type` 中。不要使用逗号式标题，例如 `Devbox，xxx`。

## 字段策略

优先使用原生 GitHub issue 元数据：

- Issue Type：`Bug`、`Feature`、`Task`
- Issue Fields：`Priority`、`Effort`，以及 issue 侧边栏中显示的其他 repo 级字段

只有当单独的 Project 看板也需要重复值时，才使用 GitHub Projects 字段：

- `Priority`：`P0`、`P1`、`P2`
- `Type`：`Bug`、`Feature`、`Task`
- `Effort`：除非明确给出或可以自信建议，否则留空；开发可能拥有最终 effort
- `Product`：规范化后的产品/模块
- `Environment`：规范化后的测试环境

issue labels 只用于状态或特殊分类：

- 允许：`待测试`、`功能相关`、`部署脚本`
- 不要添加优先级标签：`P0`、`P1`、`P2`
- 测试人员首次创建 issue 时不要添加 `待测试`；该标签表示开发已经完成并部署待测试

当测试人员提到里程碑，或当前任务明显属于某个版本/测试批次时，使用里程碑，例如 `v5.1.2`、`Offline-v5.1` 或 `public cloud`。

通过 catalog 将内部优先级映射到 GitHub 当前 repo 字段选项。对于 `labring-sigs/sealos-issues`，观察到的原生 Priority 选项是 `Urgent`、`High`、`Medium` 和 `Low`；使用 catalog 映射，而不是 labels。

## 优先级启发

- `P0`：阻塞、核心流程不可用、数据丢失/损坏、安全风险，或无法在关键路径上创建/连接/部署。
- `P1`：重要产品 bug、API/server error、功能损坏、数据错误，或影响正常使用但有 workaround 或范围较窄的行为。
- `P2`：文案、本地化、视觉打磨、小屏布局、轻微边缘兼容性、低风险改进。

不确定时，建议 `P1` 并说明理由。不要强迫测试人员成为发布经理。

## Catalog 更新

把稳定知识保存在 `assets/catalog.json` 中。使用受控学习：

- 如果检测到新产品/环境，将其预览为 `catalog_suggestions`。
- 当它看起来持久可用时，询问是否添加。
- 添加时，如果有可用信息，包含 aliases 和 source issue URL/date。
- 不要添加拼写错误、一次性测试 URL、个人账号或临时密码。

## 图片上传器

优先使用 PicList 或 PicGo，这样测试人员不需要操作 GitHub 的附件 UI：

- PicList：启用其本地服务并保持 PicList 运行。脚本默认读取 `~/Library/Application Support/piclist/data.json`，并通过 `POST /upload` 上传。
- 剪贴板截图：如果 PicList 本地服务已启用，传入 `--clipboard-image`；脚本会让 PicList 上传当前系统剪贴板图片。
- PicGo CLI：用 `npm install -g picgo` 和 `picgo set uploader` 安装/配置；脚本使用 `picgo upload <files...>`。
- 自定义命令：在 `assets/catalog.json` 中设置 `image_uploader.command`，用 `{files}` 表示所有路径，或用 `{file}` 表示第一个路径。命令必须打印最终公开图片 URL 或 Markdown 图片链接。

如果 `--create` 过程中上传失败，停止，不要创建包含不可用本地路径的 issue。

## 输出预期

回复用户时，展示简短预览：

- 标题
- 正文摘要，或在被要求时展示完整正文
- 推断字段和置信度
- 敏感信息警告
- 确切操作状态：仅 dry-run、已创建 URL，或因缺少 gh/project 权限而阻塞

保持中文 issue 草稿自然且简洁。开发者应该能复现问题，而不需要测试人员重写报告。
