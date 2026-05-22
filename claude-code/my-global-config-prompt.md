请按以下规范为我配置 Claude Code 的【用户全局层】定制。
本文件可出现具体字段名、文件名(如 settings.json、MEMORY.md),
但不写绝对路径——请按当前系统的 home 目录展开,统一落在用户级 .claude/ 下。

═══════════════════════════════════════════════════════════
1. 权限模式
═══════════════════════════════════════════════════════════

在用户全局 settings.json 中:

- permissions.defaultMode 设为 "bypassPermissions",
  让每次开会话不被工具确认弹窗打断。
- skipDangerousModePermissionPrompt 设为 true,
  禁掉 "你正在以绕过权限模式启动 Claude Code" 的二次确认。

绕过模式下 allow 列表不起作用,唯一安全网是 permissions.deny。
显式拉黑以下类型(Bash / PowerShell 各写一份):

- 磁盘根、家目录、Windows 盘符根的递归强删
  (rm -rf /、rm -rf ~、Remove-Item -Recurse -Force C:\ 等)
- 卷格式化(format、Format-Volume)
- 修改 PowerShell ExecutionPolicy
- Git 破坏性操作:push --force / push -f / reset --hard / clean -fd / branch -D
- 把网络下载的内容直接管道给 shell (curl ... | sh、iwr | iex)

效果:日常零确认,真正会丢数据 / 丢提交的命令仍被硬拦。

═══════════════════════════════════════════════════════════
2. 默认模型与外观
═══════════════════════════════════════════════════════════

settings.json 中:

- model:选当前可用的最高能力档(目前是 "opus")。
- theme:用 "dark-daltonized" 一类的暗色 + 色弱友好方案。
- autoUpdatesChannel:"latest",跟最新发布通道。
- autoCompactEnabled:true,逼近上下文上限时让 Claude Code 自己截短历史,
  避免被硬截断丢内容。

═══════════════════════════════════════════════════════════
3. 状态栏
═══════════════════════════════════════════════════════════

settings.json 的 statusLine 挂一个自定义脚本,常驻显示:
  当前模型名 + 上下文使用百分比 + 直观进度条。

Windows 用 PowerShell(.ps1),类 Unix 用 bash/python 都可以。
脚本放在用户级 .claude/ 目录下,statusLine.command 指过去即可。

═══════════════════════════════════════════════════════════
4. SessionEnd 钩子:自动清理"垃圾会话"
═══════════════════════════════════════════════════════════

settings.json 的 hooks.SessionEnd 注册一个 Node 脚本,
对本次会话的 transcript .jsonl 做以下判定,命中即删除该 .jsonl:

  规则 A —— 用户根本没说过话:
            遍历 type=="user" 的消息,凡是以 "<" 开头(系统注入的命令名 / 系统提醒等)
            或以 "Caveat:" 开头的,都视为"非真实用户输入"。
            如果真实用户消息数为 0,删。
  规则 B —— 用户最多说了一句:用户消息总条数 ≤ 1,删。

钩子读取 stdin 的 JSON(包含 transcript_path),据此定位文件。

硬约束:
- 任何异常都必须静默 exit 0,绝不阻塞 Claude Code 退出。
- 加一个几秒级兜底超时,防止脚本卡死把会话挂住。
- 删除与异常都追加写到一个 .log 文件(与脚本同目录),便于事后审计。
- 只处理本次会话的 transcript_path,不要追溯扫描其他历史 .jsonl。

效果:/resume 列表不再被"误开关""单字符回复""纯命令调用"刷屏,
     真正写过内容的会话一条都不会丢。

═══════════════════════════════════════════════════════════
5. 长期记忆偏好
═══════════════════════════════════════════════════════════

向 Claude Code 的项目级记忆(MEMORY.md 索引 + 同目录单文件 .md)
写入以下 feedback 类记忆,使其在所有项目生效:

(1) 输出文件后必须报告绝对路径
    每次写 / 改文件后,回复里要明确列出落盘的绝对路径;
    一次改多个文件时用表格列出。
    原因:用户要立刻能跳到改动位置去复核。

(2) 删除前必须列清单 + 显式确认
    删除任何文件前,先列要删的清单,等用户明确同意后再动手。
    原因:避免误删用户在做的工作。

(3) 全局配置同步规则
    每当用户提到"用户全局""全局配置"范畴的修改且实际落地,
    完成改动后**同步更新所有相关全局配置文件**,保持它们与真实全局配置始终一致。
    相关文件至少包括:
      - my-global-config-prompt.md(用户级 .claude/ 下):可分发的复刻 prompt
      - CLAUDE.md(用户级 .claude/ 下):跨项目自动加载的行为偏好
      - settings.json 引用的钩子脚本 / statusline 脚本等周边文件
    判断哪几份要改:settings 字段翻转→prompt;新增行为规则→CLAUDE.md + prompt;
    脚本本体改动→脚本本身 + prompt 中对它的描述。

    如果用户为这些全局配置文件设置了远程 git 备份仓库,
    本地改完后**自动**复制 → commit → push 到该仓库的对应子目录,无需每次确认。
    硬约束:
      - add / commit / push 串成一条 Bash 命令,避免与用户并行编辑产生的 index race。
      - 只 stage 自己刚拷过去的文件,绝不 `git add .` / `git add -A`,
        以免把用户在其他子目录里的 WIP 一起带走。
      - 授权范围仅限该具体仓库与普通 push;force-push / 破坏性 git 操作仍要单独确认。
    原因:这些文件互为参照,任一漂移都会让下次复刻或自动行为对不上账;
         远程备份能在换机或本地丢失时立刻恢复。

记忆文件按 Claude Code 约定写 frontmatter(name / description / metadata.type=feedback),
正文用 "规则 → Why → How to apply" 三段结构。

═══════════════════════════════════════════════════════════
6. 验证
═══════════════════════════════════════════════════════════

- 开一个新 Claude Code 窗口立刻 /exit,下次 /resume 列表中不应看到这条;
  SessionEnd 钩子日志应多一行"已删除…规则 A 或 A+B"。
- 试一条 permissions.deny 内的破坏命令(如 git push --force),
  即使在绕过模式下也应被拦截。
- statusLine 装好后窗口底部应出现模型名 + 上下文占用条。
- 后续任何一次写文件操作,Claude 的回复里应自动带绝对路径。

═══════════════════════════════════════════════════════════
说明
═══════════════════════════════════════════════════════════

- 所有 settings.json 改动都做"追加 / 合并",
  不要覆盖已有的 plugins / mcpServers / 其他已注册 hooks。
- 平台差异自己处理:Windows 专属的 deny 项(如 ExecutionPolicy)不要带去 macOS/Linux,
  反之亦然。
- 任何环节如果当前 Claude Code 版本不支持(没有对应 hook 事件 / 没有 statusLine 接口),
  在回复里明确告诉我"该项跳过,原因 XX",不要强行模拟。
