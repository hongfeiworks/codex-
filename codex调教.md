# Codex Windows 调教与迁移说明

## 1. 这次调整的目标

这次不是做“花里胡哨的人设调教”，而是把你最在意、而且长期真能省事的几件事固化下来：

1. 默认使用简体中文回答、写注释、写文档
2. 尽量降低 Windows 下中文注释和文档乱码的概率
3. 默认串行工作，不启用并行或多代理
4. 少拆一堆只调用一次的小方法，优先保持主流程清晰
5. 该写的中文注释要写，不该写的废话注释不写
6. 风格保持直接、务实、敢指出问题，但不做阴阳怪气的人设表演

## 2. 本次实际修改了哪些文件

### 2.1 已修改文件

1. `C:\Users\56547\.codex\AGENTS.md`
2. `C:\Users\56547\.codex\config.toml`
3. `C:\Users\56547\.codex\prompts\refactor-compact-zh.md`
4. `C:\Users\56547\.codex\prompts\comment-doc-zh.md`
5. `C:\Users\56547\.codex\prompts\debug-root-cause-zh.md`
6. `C:\Users\56547\.codex\prompts\instruction-reflector-zh.md`
7. `C:\Users\56547\.codex\skills\windows-utf8-guard\SKILL.md`
8. `C:\Users\56547\.codex\skills\compact-refactor\SKILL.md`
9. `C:\Users\56547\.codex\skills\zh-comment-doc\SKILL.md`
10. `C:\Users\56547\.codex\skills\verification-before-completion-zh\SKILL.md`
11. `C:\Users\56547\Documents\Codex\2026-05-07\https-linux-do-t-topic-1413296\codex调教.md`

其中：

- `AGENTS.md` 已额外处理为 `UTF-8 with BOM`
- 这份迁移文档也已保存为 `UTF-8 with BOM`

这样做是为了兼容 Windows PowerShell 5.1。它对“无 BOM 的 UTF-8 中文文件”经常会按 ANSI 方式读取，终端里看着就像乱码，但文件本身其实没坏。

### 2.2 已生成备份

1. `C:\Users\56547\.codex\AGENTS.md.bak-20260507-113008`
2. `C:\Users\56547\.codex\config.toml.bak-20260507-113008`

如果后面你觉得某条规则不合适，可以直接回滚，或者只改对应段落。

## 3. 为什么这样改

### 3.1 把长期规则放进全局 `AGENTS.md`

这是最适合放“默认行为”的地方，适合约束：

- 默认语言
- 注释和文档习惯
- 代码抽象尺度
- Windows 操作偏好
- 危险操作确认机制

你的诉求本质上不是“换个模型就行”，而是“让 Codex 以后每次都更像你想要的搭档”。这类规则放进 `AGENTS.md` 最稳。

### 3.2 用 `config.toml` 关掉多代理，而不是只靠口头提示

你明确说了不需要并行处理，所以这件事不能只靠每次都提醒一句“别并行”。最稳的做法是配置层直接关掉：

```toml
[features]
multi_agent = false
```

这样比写一大段 prompt 更干脆。

### 3.3 为什么加入 `personality = "pragmatic"`

你想要的不是“过度热情的客服腔”，也不是“演技太重的人设腔”，而是：

- 直接
- 务实
- 少废话
- 能指出问题

`pragmatic` 这个配置方向比较贴近你的诉求，而且比写一大段“东北大佬吐槽人格”更稳、更不容易跑偏。

### 3.4 为什么没有采用“东北吐槽、适当嘲讽”的人格写法

这个我做了优化，没有原样照抄，原因很简单：

1. 短期看起来很爽，长期协作容易跑偏
2. 一旦把“嘲讽”写死，模型容易把无关场景也说得很冲
3. 它会影响代码任务里的稳定性，不如“直接、务实、敢质疑”来得耐用

所以我保留了你想要的“别瞎迎合、发现问题直接说”，去掉了“刻意冒犯”和“人设表演”。

### 3.5 为什么没有采用“只能用 Read/Glob/Edit/Write，禁止 PowerShell”

你是在 Windows 上开发，这条如果原样加进去，其实是反着来的。

不建议原样加入的原因：

1. 你的真实环境就是 Windows，PowerShell 本来就应该是优先工具之一
2. 把工具限制写得过死，很容易误伤正常工作流
3. 真正跟乱码强相关的，不是“用了 PowerShell 就不行”，而是“用了容易破坏编码的写法”

所以我改成了更可长期使用的版本：

- Windows 下优先 PowerShell 和 Windows 路径语义
- 搜索优先 `rg` / `rg --files`
- 避免 `sed`、`awk`、`echo >`、`>`、`>>` 这种容易引入编码或行为歧义的写法

这才是适合你环境的版本。

### 3.6 为什么没有用 `AGENTS.override.md`

这次我没有长期使用 `C:\Users\56547\.codex\AGENTS.override.md`，而是把规则直接合并进 `C:\Users\56547\.codex\AGENTS.md`。

原因：

1. `override` 更适合临时覆盖，不适合当长期主配置
2. 长期维护时，主规则集中在一个文件更清楚
3. 以后换新 Windows 电脑时，直接复制一个 `AGENTS.md` 更省心

## 4. 这次具体改了什么

## 4.1 `C:\Users\56547\.codex\config.toml`

只新增了两处，其他原有 provider、MCP、插件配置完全没动。

### 新增内容

```toml
personality = "pragmatic"

[features]
multi_agent = false
```

### 作用说明

1. `personality = "pragmatic"`
   - 让整体输出更偏务实、直接、少空话
   - 更贴近你的长期使用习惯

2. `[features] multi_agent = false`
   - 默认关闭多代理
   - 避免模型一上来就走并行、拆任务、拉子代理那套
   - 对你这种偏单线程写代码的习惯更合适

### 为什么没有加别的“并行优化字段”

你给的文章里有些“并行/协作”方向的配置思路，但不是你当前的核心诉求，而且旧文章里有些字段在当前版本下并不是最该抄的内容。

这次原则是：

- 只加你明确需要的
- 只加当前版本下能自圆其说的
- 不为了“看起来会调教”硬塞一堆配置

## 4.2 `C:\Users\56547\.codex\AGENTS.md`

这个文件现在承载的是你的长期全局规则，重点包含下面几块。

### 1. 语言与输出规范

作用：

- 默认中文回答
- 默认中文注释
- 默认中文文档
- 除非你明确要求，否则不写英文说明

这能直接解决你第一条痛点：每次写英文注释、英文文档。

### 2. 代码风格与抽象尺度

我明确写进去了这些要求：

- 不要为了“显得模块化”拆出大量小方法
- 只有在复用、隔离复杂度、提升测试性、隔离副作用时才抽方法
- 单一流程中的短逻辑优先内联

这块就是专门针对你第四条痛点：代码拆太碎，看起来乱。

### 3. 注释规范

我没有写成“到处加注释”，而是写成“必要注释”：

- 关键流程写
- 核心逻辑写
- 边界条件写
- 业务约束写
- 非直觉实现写
- 显而易见的代码不写废话注释

这样能解决你第五条痛点：代码没有注释；同时避免另一个常见副作用，就是注释比代码还水。

### 4. 编码、换行与乱码防护

这里是专门为你第二条痛点写的。

核心规则有四个：

1. 默认 UTF-8
2. 不主动乱改 BOM、编码、CRLF/LF
3. 乱码先判断根因，不瞎转码
4. 避免用 `>`、`>>`、`echo >` 这类容易出编码问题的写法

注意，这一块是“降低乱码概率”，不是“100% 消灭一切乱码”。因为 Windows 下乱码可能来自多个层面，后面我会单独说明。

### 5. Windows 执行环境规范

这里明确了：

- 你是 Windows 环境
- 优先 PowerShell
- 优先 Windows 路径语义
- 搜索优先 `rg`
- 避免拿 Linux 一行命令硬套 Windows

这个改法比“禁止 PowerShell、必须 Bash”更符合你的真实开发环境。

### 6. 执行模型

这里明确写了：

- 默认串行
- 不主动并行
- 不主动启用子代理
- 只有你明确要求，或者收益非常明显时才考虑

这块就是针对你第三条痛点。

### 7. 危险操作确认机制

你给的这部分我保留了，而且做了增强。

现在规则要求在下面这些场景必须先确认：

- 删除文件/目录
- 批量修改大量文件
- 改系统配置、环境变量、权限
- 数据库结构变更或批量更新
- 调用生产 API 或发送敏感数据
- 全局安装/卸载/升级核心依赖
- 批量修改编码、BOM、换行风格

确认前必须说明：

1. 操作类型
2. 影响范围
3. 风险评估

这个规则很值钱，能防止“为了修一点小问题，顺手把半个环境搞乱”。

## 4.3 `C:\Users\56547\.codex\prompts\`

这次新增了 4 个高频 prompt，目的不是堆数量，而是把最常重复的几类任务做成一键调用。

### 1. `refactor-compact-zh.md`

用途：

- 紧凑型重构
- 少拆小方法
- 保持主流程线性可读

适合场景：

- 代码被拆得太碎
- 想清理结构但不想重架构
- 希望 Codex 少做形式主义抽象

### 2. `comment-doc-zh.md`

用途：

- 补必要中文注释
- 补必要中文说明文档
- 避免英文注释和废话注释

适合场景：

- 老代码缺注释
- 改动后需要补说明
- 需要中文变更说明或迁移说明

### 3. `debug-root-cause-zh.md`

用途：

- 排查 bug 时先找根因，再修复
- 避免一上来就拍脑袋改代码

适合场景：

- 线上或本地异常排查
- 回归 bug
- 不确定到底是环境问题、数据问题还是代码问题

### 4. `instruction-reflector-zh.md`

用途：

- 回顾最近几轮对话和任务结果
- 反推 `AGENTS.md` 或项目规则哪里还需要微调

适合场景：

- 发现 Codex 还在反复犯同类问题
- 想逐步迭代自己的长期规则

## 4.4 `C:\Users\56547\.codex\skills\`

这次新增了 4 个偏执行型 skill，用来把高频复杂任务固定成稳定流程。

### 1. `windows-utf8-guard`

用途：

- 处理 Windows 下中文乱码、UTF-8、BOM、PowerShell 误读、CRLF/LF 混乱

价值：

- 这是你当前环境里最有针对性的 skill
- 能把“文件坏了”和“只是控制台读花了”区分开

### 2. `compact-refactor`

用途：

- 执行“清理代码但不要拆碎”的重构

价值：

- 把“什么时候该内联、什么时候该抽方法”写成明确规则
- 比每次口头提醒更稳定

### 3. `zh-comment-doc`

用途：

- 统一中文注释、中文 README、中文变更说明、中文迁移说明的产出方式

价值：

- 避免英文说明反复冒出来
- 避免注释质量忽高忽低

### 4. `verification-before-completion-zh`

用途：

- 在宣布完成前，强制补一轮验证

价值：

- 防止“代码改了，但没真正验证”
- 尤其适合修 bug、重构、配置改动后的收尾阶段

## 5. 关于中文乱码，这次到底解决了多少

先说结论：

这次改动能明显降低“Codex 自己生成内容时继续制造乱码”的概率，但不能单靠一份 `AGENTS.md` 彻底修复所有 Windows 编码问题。

### 5.1 这次改动能解决的

1. 减少英文注释和英文文档输出
2. 提醒模型默认按 UTF-8 处理
3. 避免使用容易破坏编码的覆盖写法
4. 避免无关的 BOM、换行、编码改动
5. 让模型在看到乱码时先判断原因，而不是直接乱转码
6. 对中文为主的 Markdown 规则文件，在当前这台 Windows 机器上额外落成 `UTF-8 with BOM`，降低 PowerShell 5.1 读取时的假乱码问题

### 5.2 这次改动解决不了的

1. 你本来就有一些旧文件是 GBK / ANSI 编码
2. 某些编辑器默认不是 UTF-8
3. 某些终端只是“显示乱码”，文件本身其实没坏
4. 个别工具链或插件在 Windows 下存在编码 bug

### 5.3 Windows 下最常见的乱码来源

1. 文件原本就是 ANSI / GBK
2. PowerShell 或终端显示编码不一致
3. 用重定向写文件，把 UTF-8 内容写坏了
4. 项目里不同文件混用了多种编码

### 5.4 真想进一步稳住编码，建议你后面再补两件事

这次我没有直接帮你加，是因为你这轮主要是在调 Codex 本身，不是改具体项目。

#### 可选项 1：给项目加 `.editorconfig`

推荐内容示例：

```ini
root = true

[*]
charset = utf-8
end_of_line = crlf
insert_final_newline = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

作用：

- 明确项目默认编码
- 统一换行风格
- 进一步减少“我这边是 UTF-8，你那边不是”的混乱

#### 可选项 2：编辑器默认编码改成 UTF-8

如果你主要用 VS Code，建议确认这些设置：

1. `files.encoding = utf8`
2. `files.autoGuessEncoding = false` 或按你的习惯设置
3. 保存时不要随手切成 GBK

这一步比单纯写 prompt 更直接。

## 6. 为什么这份方案比“抄一大堆调教文章”更适合你

你给的两篇文章都有参考价值，但要分清楚“能学的”和“别硬搬的”。

### 可以吸收的部分

1. 把稳定偏好固化到 `AGENTS.md`
2. 把风格、质量标准、危险操作规则写明确
3. 尽量让输出行为一致，而不是每次重新解释需求

### 这次没有照抄的部分

1. 多代理、并行处理
   - 你明确不需要
   - 开了反而增加噪音和复杂度

2. 过重的人设
   - 容易让模型沉迷演风格，反而影响干活

3. 过死的工具禁令
   - 特别是“Windows 上禁止 PowerShell”这种，明显不符合你的环境

4. 过时或不必要的配置项
   - 旧文章常见这个毛病，看着很热闹，真落地不一定值

所以这次的思路不是“抄满”，而是“只留下对你长期真正有用的”。

## 7. 以后换一台新的 Windows 电脑，怎么迁移

你后面说得很清楚，所谓“换环境”就是“换一台新的 Windows 电脑”。那就按这个流程来。

## 7.1 迁移目标

把下面两类东西带过去：

1. 全局行为规则
2. 关键配置开关
3. 高频 prompts
4. 高频 skills

对应文件就是：

1. `C:\Users\你的用户名\.codex\AGENTS.md`
2. `C:\Users\你的用户名\.codex\config.toml`
3. `C:\Users\你的用户名\.codex\prompts\`
4. `C:\Users\你的用户名\.codex\skills\`

## 7.2 推荐迁移步骤

### 第一步：在新电脑安装 Codex

先正常安装并登录，让新电脑先生成自己的 `.codex` 目录。

### 第二步：关闭 Codex

在替换配置文件前，先退出 Codex，避免它正在读写配置。

### 第三步：备份新电脑自己的默认配置

建议先备份：

1. `C:\Users\你的用户名\.codex\AGENTS.md`
2. `C:\Users\你的用户名\.codex\config.toml`
3. `C:\Users\你的用户名\.codex\prompts\`
4. `C:\Users\你的用户名\.codex\skills\`

PowerShell 示例：

```powershell
$stamp = Get-Date -Format "yyyyMMdd-HHmmss"
Copy-Item -LiteralPath "$env:USERPROFILE\.codex\AGENTS.md" -Destination "$env:USERPROFILE\.codex\AGENTS.md.bak-$stamp" -Force
Copy-Item -LiteralPath "$env:USERPROFILE\.codex\config.toml" -Destination "$env:USERPROFILE\.codex\config.toml.bak-$stamp" -Force
if (Test-Path -LiteralPath "$env:USERPROFILE\.codex\prompts") {
    Copy-Item -LiteralPath "$env:USERPROFILE\.codex\prompts" -Destination "$env:USERPROFILE\.codex\prompts.bak-$stamp" -Recurse -Force
}
if (Test-Path -LiteralPath "$env:USERPROFILE\.codex\skills") {
    Copy-Item -LiteralPath "$env:USERPROFILE\.codex\skills" -Destination "$env:USERPROFILE\.codex\skills.bak-$stamp" -Recurse -Force
}
```

### 第四步：迁移 `AGENTS.md`

最简单的方法就是直接复制这台老电脑上的文件内容到新电脑：

源文件：

`C:\Users\56547\.codex\AGENTS.md`

目标文件：

`C:\Users\你的用户名\.codex\AGENTS.md`

这一步基本可以整文件覆盖，因为它本来就是你的全局个人规则。

### 第五步：迁移 `config.toml`

这里分两种情况。

#### 情况 A：新电脑也继续用同一个 provider 和相同配置

可以直接复制整份：

源文件：

`C:\Users\56547\.codex\config.toml`

目标文件：

`C:\Users\你的用户名\.codex\config.toml`

#### 情况 B：新电脑的 provider、API 地址或认证方式不一样

不要整份覆盖，改成“手动合并”。

至少把下面两处加进去：

```toml
personality = "pragmatic"

[features]
multi_agent = false
```

然后保留新电脑自己的这些内容：

- `model_provider`
- provider 认证信息
- `base_url`
- MCP 配置
- 插件配置

### 第六步：迁移 `prompts` 和 `skills`

把下面这些一并复制到新电脑：

#### prompts

1. `C:\Users\56547\.codex\prompts\refactor-compact-zh.md`
2. `C:\Users\56547\.codex\prompts\comment-doc-zh.md`
3. `C:\Users\56547\.codex\prompts\debug-root-cause-zh.md`
4. `C:\Users\56547\.codex\prompts\instruction-reflector-zh.md`

目标目录：

`C:\Users\你的用户名\.codex\prompts\`

#### skills

1. `C:\Users\56547\.codex\skills\windows-utf8-guard\SKILL.md`
2. `C:\Users\56547\.codex\skills\compact-refactor\SKILL.md`
3. `C:\Users\56547\.codex\skills\zh-comment-doc\SKILL.md`
4. `C:\Users\56547\.codex\skills\verification-before-completion-zh\SKILL.md`

目标目录：

`C:\Users\你的用户名\.codex\skills\`

注意：

- `prompts` 是单文件复制
- `skills` 是按目录复制，目录名本身就是 skill 名
- 只复制 `SKILL.md` 不够稳，最好把整个 skill 目录复制过去，后续如果你再补脚本或 references 就不会丢

### 第七步：确认文件编码

把新电脑上的关键中文文件用编辑器打开，确认是 UTF-8。

重点确认：

1. `AGENTS.md`
2. `config.toml`
3. `prompts` 里的中文 `.md`
4. `skills` 里的中文 `SKILL.md`

如果编辑器状态栏能看编码，直接看状态栏最省事。

补充说明：

- 如果 `AGENTS.md` 里有大量中文，Windows 上建议直接保存成 `UTF-8 with BOM`
- `config.toml` 这份文件目前几乎都是 ASCII，通常普通 UTF-8 就够了
- 中文 prompt 和中文 `SKILL.md` 在 Windows 上也建议优先保存成 `UTF-8 with BOM`

### 第八步：重新打开 Codex

打开后做三次简单验证。

#### 验证 1：中文输出

问它：

“以后默认用什么语言回复和写注释？”

如果配置生效，它应该偏向明确回答“简体中文”。

#### 验证 2：串行偏好

问它：

“你默认会不会主动并行拆任务或启用多代理？”

如果配置生效，它应该偏向回答“默认串行，不主动并行”。

#### 验证 3：代码风格

给它一段代码，要求修改，看它会不会把本来一段短逻辑硬拆成一堆小函数。

#### 验证 4：自定义 prompts 与 skills

检查两件事：

1. 在 prompt 菜单里能否看到这 4 个自定义 prompt
2. 新会话里请求“处理 Windows UTF-8 乱码”或“做紧凑型重构”时，Codex 是否能更自然地贴近新增 skill 的规则

## 7.3 一键复制时的注意事项

最容易出问题的是 `config.toml`，不是 `AGENTS.md`。

原因：

1. `AGENTS.md` 主要是规则文本
2. `config.toml` 里可能带有你当前机器绑定的 provider 信息、MCP 配置、插件状态
3. `prompts` 和 `skills` 才是后续效率提升最明显、也最容易被漏迁移的一层

所以：

- `AGENTS.md` 可以更大胆地复制
- `config.toml` 要看你新电脑是否沿用同一套服务配置
- `prompts` 和 `skills` 建议整套一起迁

## 8. 如果以后还想继续加强，有哪几个方向最值

这几个是下一步最值得做的，不用现在一起塞进去。

### 8.1 给常用项目加项目级 `AGENTS.md`

全局 `AGENTS.md` 解决的是“你这个人长期想怎么用 Codex”。

项目级 `AGENTS.md` 解决的是“这个仓库应该怎么写”。

比如你可以在具体项目里继续补：

- 这个项目注释密度要多高
- 变量命名偏英文还是偏拼音
- 是否统一 CRLF
- 哪些目录不能动
- 哪些接口不能擅自改

### 8.2 给常用项目补 `.editorconfig`

这是控制编码和换行最便宜、最有效的方式之一。

### 8.3 统一编辑器保存编码

如果编辑器本身就在乱切编码，那你光调 Codex 还是会受影响。

## 8.4 什么时候才建议启用多代理

默认结论仍然是：

- 平时不要开
- 只有在任务天然可拆、边界清晰、并行后总成本更低时才开

### 快速判断清单

下面 6 条里，如果同时满足 3 条以上，才值得认真考虑启用多代理：

1. 任务可以拆成 2 到 3 个相对独立的子任务
2. 子任务之间几乎不改同一批文件
3. 各子任务可以按照清晰接口或目录边界并行推进
4. 主线实现不依赖另一个子任务的即时结果
5. 最终合并时只需要轻量对齐，不需要大规模重构
6. 并行后节省的时间，明显大于沟通、合并、复核成本

### 适合启用的场景

1. 大型重构，但模块边界非常清楚
2. 一个代理改实现，另一个代理补测试或做独立验证
3. 一个代理查资料、梳理现状，另一个代理推进不依赖结果的实现部分
4. 多个独立页面、独立模块、独立目录同时改动，彼此基本不冲突

### 不适合启用的场景

1. 单文件修改
2. 单条主流程的小范围调整
3. 编码乱码、环境变量、路径、Windows 兼容性这类强依赖上下文的问题
4. 需要统一风格、统一抽象尺度、统一注释口径的细活
5. 你还没想清楚怎么拆，只是觉得“并行可能更快”

### 对你当前使用方式的建议

按你现在的习惯，建议长期保持：

```toml
[features]
multi_agent = false
```

原因很明确：

1. 你更在意稳定、清晰、少噪音，而不是表面上的推进速度
2. 你不希望代码被拆碎，多代理天然更容易放大这种问题
3. 中文注释、文档口径、编码处理这类事情，串行更容易控住一致性

### 一句话判断法

先问自己一句：

“这个任务拆开以后，最后的合并成本会不会比单线程直接做更低？”

如果答案不够明确，就不要开。

## 8.5 本次新增的 Prompts 与 Skills，后面怎么用

### prompts 的使用方式

放在：

`C:\Users\56547\.codex\prompts\`

当前新增：

1. `refactor-compact-zh`
2. `comment-doc-zh`
3. `debug-root-cause-zh`
4. `instruction-reflector-zh`

使用建议：

- 重构但不想拆碎时，用 `refactor-compact-zh`
- 要补中文注释和中文说明时，用 `comment-doc-zh`
- 排查问题时，用 `debug-root-cause-zh`
- 想继续优化规则时，用 `instruction-reflector-zh`

### skills 的使用方式

放在：

`C:\Users\56547\.codex\skills\`

当前新增：

1. `windows-utf8-guard`
2. `compact-refactor`
3. `zh-comment-doc`
4. `verification-before-completion-zh`

使用建议：

- Windows 编码、BOM、乱码排查：`windows-utf8-guard`
- 少拆方法的重构：`compact-refactor`
- 中文注释与中文文档：`zh-comment-doc`
- 改完后强制验证：`verification-before-completion-zh`

### 使用上的实话

这些 prompts 和 skills 的价值不在“替你思考”，而在于：

1. 减少你反复重复同一套要求
2. 把高频任务固化成稳定工作流
3. 让 Codex 更容易在你的长期偏好里保持一致

它们本质上是“把你的工作习惯变成可复用入口”。

## 9. 当前这份方案的核心价值

一句话总结：

不是让 Codex 变得“更会演”，而是让它更稳定地按你的习惯干活。

具体价值是：

1. 默认中文
2. 默认少拆小方法
3. 默认串行
4. 默认补必要中文注释
5. 默认更谨慎处理编码与换行
6. 默认在 Windows 语境下做正确决策
7. 常用工作流已经有了可复用的 prompts
8. 高频复杂任务已经有了可复用的 skills

## 10. 附录：当前推荐使用的 `AGENTS.md` 全文

下面这段就是当前已经写入你机器的全局规则。如果以后换新 Windows 电脑，最省事的做法就是直接复制这份内容。

```md
# 全局协作规范（Windows 开发）

> 本文件约束 Codex 的全局默认行为。若项目内存在更具体的 `AGENTS.md`、用户当次提出了明确要求，或系统安全规则另有规定，则以更高优先级的指令为准。

## 1. 语言与输出规范

- 所有可见回复、解释、计划、代码注释、技术文档、变更说明默认使用简体中文。
- 除非用户明确要求，否则不要输出英文注释、英文文档、英文操作说明。
- 代码中的标识符、框架 API、第三方库名称遵循项目既有风格，不为了“全中文”强行改动已有命名体系。

## 2. 基本工作原则

- 质量第一，代码正确性、可维护性和安全性优先于表面上的“快”。
- 复杂任务先分析再动手，小任务可以直接执行，但需要保留关键判断依据。
- 优先复用项目现有模式、现有工具链和成熟方案，不为了一点形式上的“优雅”发明新结构。
- 关键决策需要可追溯：为什么这样改、影响了什么、还有哪些边界条件。
- 以结果达成为导向。默认直接帮用户落地，不停留在泛泛建议层面。

## 3. 代码风格与抽象尺度

- 遵循 SOLID、DRY、关注点分离和 YAGNI，但不过度设计。
- 命名要清晰，控制流程要尽量线性、易读、易追踪。
- 不要为了“显得模块化”而拆出大量只被调用一次、只有一两行、没有复用价值的小方法。
- 只有在以下情况成立时才抽方法：
  - 逻辑会被复用
  - 抽出后能明显降低局部复杂度
  - 抽出后能提升可测试性
  - 需要隔离副作用、边界处理或错误处理
- 对于单一流程中的短逻辑，优先内联，保持阅读时一眼能看清主路径。
- 修改功能时，及时删除无用代码、旧分支和无意义的兼容层；除非用户明确要求，否则不要保留陈旧过渡代码。

## 4. 注释与文档规范

- 注释和文档默认使用简体中文。
- 关键流程、核心逻辑、业务约束、边界条件、性能取舍、非直觉实现必须补充简洁的中文注释。
- 不写噪音注释，不解释显而易见的赋值、判断或语法动作。
- 变更涉及使用方式、配置方法、迁移步骤时，默认补充中文说明文档；除非用户要求，否则不额外输出英文版。

## 5. 编码、换行与乱码防护

- 新建或编辑文本文件时，默认使用 UTF-8。
- 除非用户明确要求，否则不要主动改变已有文件的编码、BOM 头或换行风格（CRLF/LF）。
- 遇到中文乱码时，先区分问题来源：
  - 文件本身编码错误
  - 终端或控制台显示编码错误
  - 工具读取、重定向或脚本写入时破坏了编码
- 在未确认根因前，不要批量转码，不要整仓库统一改编码。
- 避免使用容易引入编码漂移的写法，例如 `>`、`>>`、`echo >` 这类 shell 重定向覆盖文件；优先使用稳定的编辑工具或原生文件编辑能力。
- 编辑文件时尽量保留原有字符集和换行风格，避免无关的格式噪音。

## 6. Windows 执行环境规范

- 默认开发环境为 Windows，优先使用 PowerShell 和 Windows 路径语义，不假设 Bash 或 Linux 工具链存在。
- 搜索文件或文本时，若 `rg`/`rg --files` 可用，优先使用；不可用时再使用 PowerShell 等价方案。
- 不依赖 `sed`、`awk`、`head`、`tail` 这类偏 Linux 的一行命令完成关键编辑；需要修改文件时，优先使用明确、可读、可控的编辑方式。
- 路径、编码、换行敏感的操作要尽量缩小影响范围，优先保留现状。

## 7. 执行模型

- 默认串行工作，不主动并行化，不主动启用子代理或多代理协作。
- 只有在用户明确要求，或任务规模足够大且并行收益非常明确时，才讨论并行方案。
- 默认先把主线做通，再考虑扩展优化，不为了“并发感”增加额外复杂度。

## 8. 危险操作确认机制

- 执行以下操作前必须获得用户明确确认：
  - 删除文件或目录
  - 递归移动、批量重命名、批量修改大量文件
  - 修改系统配置、环境变量、权限设置
  - 数据库删除、结构变更、批量更新
  - 调用生产环境 API 或发送敏感数据
  - 全局安装、卸载或升级核心依赖
  - 批量修改文件编码、BOM 或换行风格
- 确认前需要说明三件事：
  - 操作类型
  - 影响范围
  - 风险评估
- 等待用户明确回复“是”或“确认”后再执行。

## 9. 沟通风格

- 风格保持直接、务实、清醒，发现问题要明确指出，不做无效迎合。
- 当用户方案存在明显风险、过度设计、过时配置或错误前提时，要给出清楚理由和替代方案。
- 保持尊重、克制和可操作性，不使用嘲讽、羞辱或刻意冒犯的表达。
- 目标是帮助用户把事情做成，同时把后续维护成本控制住。
```

## 11. 参考依据

这次方案不是拍脑袋写的，主要参考了下面几类信息，并结合你的真实诉求做了筛选。

### 官方参考

1. OpenAI Codex `AGENTS.md` 指南  
   <https://developers.openai.com/codex/guides/agents-md>
2. OpenAI Codex 配置参考  
   <https://developers.openai.com/codex/config-reference>
3. OpenAI Codex Windows 说明  
   <https://developers.openai.com/codex/app/windows>

### 你提供的参考文章

1. Linux.do 文章  
   <https://linux.do/t/topic/1413296>
2. RexAI 文章  
   <https://rexai.top/ai/codex/codex-53-parallel-optimization/>

### 与编码问题相关的社区现象参考

1. OpenAI Codex GitHub issue：Windows / PowerShell 默认 ANSI 编码导致的非英文显示问题  
   <https://github.com/openai/codex/issues/4498>

## 12. 最后给你的实话

你这套需求其实一点都不离谱，反而挺成熟。

真正容易把配置搞坏的，往往不是要求太多，而是三种常见毛病：

1. 什么文章都抄一点，最后彼此打架
2. 人设写太重，模型开始演而不是干活
3. 想解决乱码，结果上来就全仓库批量转码

这次我给你做的，就是把最有价值的那层留下，把最容易惹事的那层拿掉。后面你再换一台 Windows 电脑，照这份文档迁过去就行。
