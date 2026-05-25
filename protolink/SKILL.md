---
name: protolink
description: 在 IDE 里的 AI 助手中，帮助用户直接完成 ProtoLink 的项目预览、迭代预览、查看本地迭代、创建/更新/发布迭代、获取在线链接、发布配置诊断与安装引导；支持“打开当前项目预览”“看看有哪些迭代”“发布这个迭代并给我链接”“看看为什么还不能发布”“安装/更新 ProtoLink Skill”“还没装 protolink 怎么办”这类自然说法
---

# ProtoLink Skill

你负责把用户在 IDE 里的自然语言请求，转换成当前工作区中 ProtoLink 现有 CLI 的最短操作序列，并把结果整理成可直接点击的链接、简短列表或发布状态摘要。

## 这类说法通常应该命中
- 打开当前项目预览
- 给我当前项目的 ProtoLink 预览地址
- 看看现在有哪些本地迭代
- 帮我新建一个迭代
- 把这个页面加进某个迭代
- 把这个迭代再加几个页面
- 打开这个迭代的本地预览
- 帮我本地看一下这个迭代
- 发布这个迭代并把链接给我
- 这个迭代的在线地址是什么
- 这个迭代现在发布到哪了
- 看看后台发布状态
- 这个迭代现在能不能直接发布
- 看看为什么还不能发布
- 帮我检查一下当前发布配置
- 安装 / 更新 ProtoLink Skill
- 自动安装 ProtoLink CLI
- 本地还没装 protolink，怎么办
- 找不到 protolink 命令

## 基本原则
- 只操作当前工作区，不猜别的仓库。
- 面向 IDE 内的 AI 使用，不把交互入口、提示文案或能力边界写死到 Claude Code；只要当前 IDE 里的 AI 能调用 ProtoLink CLI，这套编排语义就成立。
- 先检查当前 IDE / AI 进程里是否能直接执行 `protolink`：优先按 `command -v protolink >/dev/null 2>&1 && protolink <command>` 这条路径理解。
- 只要 `protolink` 在 `PATH` 里可执行，就统一使用 `protolink <command>`；这包括“用户只做了全局安装、当前项目没有安装 protolink”的场景。
- 当前环境找不到 `protolink` 命令时，先判断当前工作区是否就是 ProtoLink 源码仓：只有当存在 `package.json` 且 package name 为 `protolink`，并且有可用的 `npm run cli -- <command>` 开发入口时，才回退到源码仓 CLI。
- 如果当前环境找不到 `protolink` 命令，且当前工作区不是 ProtoLink 源码仓或没有明确的本地 CLI 开发入口，不要继续猜测本地脚本；应主动尝试通过官方远程安装命令安装 ProtoLink CLI。
- 官方远程安装源固定为 `https://github.com/lurenyi2025/protolink-release/releases/latest/download/protolink-latest.json`；主动远程安装时必须先读取该 manifest，再按 manifest 下载和校验 tarball。
- 远程安装完成后，必须重新执行 `command -v protolink`，并优先运行 `protolink inspect-install` 核对安装位置和 CLI entry；确认可用后再继续执行用户原本的 ProtoLink 请求。
- 如果远程安装失败，不要引导用户去网站下载；应转述失败原因、说明卡在下载 / hash 校验 / npm 安装 / inspect-install 哪一步，并等待用户处理网络或 npm 权限问题后再重试。
- 不要因为当前目录里碰巧存在 ProtoLink 源码、`node_modules` 或可推测的本地脚本，就跳过对全局 `protolink` 的可执行性检查。
- 不要用 `protolink --version` 作为唯一可用性判断；当前 CLI 还未实现该参数时，应以 `command -v protolink` 和真实业务命令能否执行为准。
- 用户询问“如何安装 / 更新 ProtoLink Skill”时，可以按官方 GitHub / 文档提供的命令安装或更新；如果当前上下文没有明确的官方仓库地址或安装命令，不要编造 URL 或命令。
- 信息不完整时先补问，不猜迭代名、页面名或页面归属。
- 页面选择先用 `list-pages`，迭代选择先用 `list-local-iterations`。
- 不要用 `protolink <command> --help` 去试探参数；直接按下面的命令契约调用。现在 CLI 已支持子命令帮助，但它只是给人看的，不是编排前置条件。
- 用户要“项目预览地址”时，返回 `preview` 的结果，不要返回 direct page。
- 用户要“本地预览链接”时，返回 `preview-iteration` 的结果。
- 用户要“在线预览链接”时，先尝试 `show-iteration-link`；如果返回 queued / running / failed 相关提示，就明确说明链接可能已预生成但尚未真正进入已发布索引。
- 用户要“后台发布状态”时，优先调用 `publish-status` 查看 queued / running / succeeded / failed。
- 如果用户提到“预览”“发布”“链接”“配置”“检查一下能不能发”这类词，但上下文明显是在说当前工作区里的 ProtoLink 项目 / 迭代，也应按 ProtoLink 语义理解，不要求用户必须说出完整命令式表述。
- 如果用户说的是普通站点预览、应用部署或与 ProtoLink 无关的发布，不要误命中这个 Skill。

## 意图到命令
- 项目预览地址 → `preview`
- 当前有哪些迭代 → `list-local-iterations`
- 创建迭代 → `list-pages` → `create-iteration` → `preview-iteration`
- 更新迭代 → `list-local-iterations` → `list-pages`（按需）→ `update-iteration` → `preview-iteration`
- 本地预览链接 → `preview-iteration`
- 发布迭代 → `publish-iteration`
- 在线预览链接 → `show-iteration-link`
- 后台发布状态 → `publish-status`
- 检查当前项目能不能发布 / 缺什么配置 → `check-remote-config`

## 官方远程安装 CLI
- 当普通用户项目缺少 `protolink` CLI 时，按以下流程主动安装，不要要求用户手动下载：

```bash
manifest_url="https://github.com/lurenyi2025/protolink-release/releases/latest/download/protolink-latest.json"
tmpdir="$(mktemp -d)"
curl --http1.1 --retry 2 --connect-timeout 10 --max-time 60 -fsSL "$manifest_url" -o "$tmpdir/protolink-latest.json"
node -e "const fs=require('fs'); const m=JSON.parse(fs.readFileSync(process.argv[1],'utf8')); for (const k of ['version','tarballUrl','tarballSha256','cliEntrySha256']) { if (!m[k]) throw new Error('missing '+k); } console.log(m.tarballUrl)" "$tmpdir/protolink-latest.json" > "$tmpdir/tarball-url"
tarball_url="$(cat "$tmpdir/tarball-url")"
curl --http1.1 --retry 2 --connect-timeout 10 --max-time 120 -fL "$tarball_url" -o "$tmpdir/protolink.tgz"
expected_sha="$(node -e "const fs=require('fs'); console.log(JSON.parse(fs.readFileSync(process.argv[1],'utf8')).tarballSha256)" "$tmpdir/protolink-latest.json")"
actual_sha="$(shasum -a 256 "$tmpdir/protolink.tgz" | awk '{print $1}')"
test "$actual_sha" = "$expected_sha"
npm install -g --force "$tmpdir/protolink.tgz"
prefix_bin="$(npm config get prefix)/bin"
export PATH="$prefix_bin:$PATH"
profile_file=""
case "${SHELL:-}" in
  */zsh) profile_file="$HOME/.zshrc" ;;
  */bash) profile_file="$HOME/.bashrc" ;;
esac
if [ -n "$profile_file" ]; then
  touch "$profile_file"
  if ! grep -Fq "# ProtoLink CLI PATH" "$profile_file"; then
    {
      echo ""
      echo "# ProtoLink CLI PATH"
      echo "export PATH=\"$prefix_bin:\\$PATH\""
    } >> "$profile_file"
  elif ! grep -Fq "$prefix_bin" "$profile_file"; then
    {
      echo ""
      echo "# ProtoLink CLI PATH"
      echo "export PATH=\"$prefix_bin:\\$PATH\""
    } >> "$profile_file"
  fi
fi
hash -r 2>/dev/null || true
command -v protolink
protolink inspect-install > "$tmpdir/inspect-install.json"
expected_cli_sha="$(node -e "const fs=require('fs'); console.log(JSON.parse(fs.readFileSync(process.argv[1],'utf8')).cliEntrySha256)" "$tmpdir/protolink-latest.json")"
actual_cli_sha="$(node -e "const fs=require('fs'); console.log(JSON.parse(fs.readFileSync(process.argv[1],'utf8')).cliEntrySha256)" "$tmpdir/inspect-install.json")"
test "$actual_cli_sha" = "$expected_cli_sha"
cat "$tmpdir/inspect-install.json"
```

- 如果 `test "$actual_sha" = "$expected_sha"` 失败，必须停止安装并提示 tarball hash mismatch。
- `npm install -g` 可能因权限或 npm prefix 失败；失败时转述 npm 的关键报错，不要换用未确认的安装路径。
- 安装后必须把 `npm config get prefix` 对应的 `bin` 加入当前会话 `PATH`，并幂等写入当前 shell 的 profile（zsh 写 `~/.zshrc`，bash 写 `~/.bashrc`）；如果用户已有终端仍提示 command not found，提示执行 `source ~/.zshrc` / `source ~/.bashrc` 或重开终端。
- `protolink inspect-install` 的 `cliEntrySha256` 应与 manifest 中的 `cliEntrySha256` 对齐；不一致时说明 PATH 可能命中了旧 CLI 或全局安装未覆盖成功。

## 典型编排规则
- 创建迭代时：先从 `list-pages` 里按页面名匹配 page ids，再调用 `create-iteration`；成功后立刻调用 `preview-iteration` 返回本地链接。
- 更新迭代时：先用 `list-local-iterations` 找到目标迭代；如果用户是在“加入页面”，先保留原有 `pageIds`，再把新匹配到的页面 id 合并后传给 `update-iteration`；如果只是改名或改描述，不要无意覆盖页面集合。
- 获取在线预览链接时：先调用 `show-iteration-link`；如果返回未发布错误，明确告诉用户还没有在线预览链接，并可继续帮他发布；如果返回 queued / running，说明已有预生成链接但远端仍在后台同步；如果返回 failed，说明最近一次远端发布失败并转向 `publish-status`。
- 获取后台发布状态时：先调用 `publish-status`；如果没有记录，说明这次还没有生成后台状态。
- 如果用户直接说“发布并给我链接”，在目标明确时直接执行 `publish-iteration`；SSH quick publish 返回后可以给出顶层 `previewUrl`，但必须说明这是预生成链接，远端同步状态以 `backgroundPublish.state` / `publish-status` 为准。
- 如果用户说“看看能不能发”“缺什么配置”“为什么发布不了”，优先调用 `check-remote-config`，并把输出里的关键结论和“下一步”整理给用户，而不是只贴原始终端输出。
- 如果发布时缺 server config，优先给出当前工作区的配置诊断或继续引导用户补齐，不要只返回原始报错。
- 如果用户提到“这个迭代”但上下文里没有唯一目标，先展示 `list-local-iterations` 的结果再确认。
- 如果多个页面都可能匹配同一句话，先列出候选页面名再确认，不要私自挑一个。
- 如果多个迭代名字相近，先澄清目标，再继续做预览、发布或查链接。

## 失败与容错
- `command -v protolink` 失败且当前不是 ProtoLink 源码仓时：不要执行 `npm run cli`；优先按官方远程安装命令安装 ProtoLink CLI，安装后重新探测并继续原请求。
- 自动远程安装失败时：转述失败原因和最后一条关键错误，不要把用户引导到下载网站；优先让用户处理网络、GitHub 可达性、npm 权限或 PATH 命中问题。
- `show-iteration-link` 报未发布时：明确说“这个迭代还没有在线预览链接”，并给出两个继续方向：先本地预览，或直接发布。
- `show-iteration-link` 报 queued / running 时：明确说“在线链接已预生成，但远端仍在后台同步”，并建议继续查看 `publish-status` 或使用 `publish-iteration <name> --wait` 严格等待。
- `show-iteration-link` 报 failed 时：说明最近一次远端发布失败，转述错误原因和日志路径，并建议先修复失败原因后重新发布。
- `publish-status` 没有记录时：明确告诉用户这次还没有后台发布状态，通常表示还没走 SSH 快速返回流程或还没开始发布。
- `publish-iteration` 因配置缺失失败时：优先转到 `check-remote-config`，把缺少的字段和下一步动作解释清楚。
- `preview-iteration` 或 `publish-iteration` 找不到迭代时：先调用 `list-local-iterations` 帮用户对齐当前可选迭代名。
- 页面匹配不到时：先重新调用 `list-pages`，向用户展示最接近的候选，而不是捏造 page id。

## 命令契约
- `create-iteration` 的参数顺序固定为：`<name> [description] [pageIdsCommaSeparated]`
- `update-iteration` 的参数顺序固定为：`<name> [nextName] [description] [pageIdsCommaSeparated]`
- `publish-iteration` 在 SSH 发布配置下会先返回 `previewUrl` 并把实际远端发布交给后台 worker；不要额外等待后台完成后才回复用户
- `publish-iteration` 之后，优先读取返回 JSON 顶层的 `previewUrl`；SSH quick publish 下 `publishedPreview` 可能是 `null`，不要把 `previewUrl` 解释成远端同步已完成
- 只有当用户明确要求“等发布彻底结束 / 同步发布 / 严格确认远端完成”时，才使用 `publish-iteration <name> --wait`
- 查询后台发布状态时，优先读取 `publish-status` 的 JSON；如果状态是 `queued` 或 `running`，要把它解释成后台仍在同步，不要伪装成已完成。

## 返回要求
- 创建/更新迭代后，优先把本地 preview URL 放在最前面。
- 发布成功后，优先把在线预览 URL 放在最前面。
- 如果返回的是列表，只保留用户当前决策需要的信息，不展开底层实现。
- 返回给用户时尽量只给结果，不要展开 CLI 过程，除非用户要求。
