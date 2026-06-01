# ProtoLink Skill

ProtoLink bootstrap Skill for AI IDE product manager workflows.

## Install

```bash
npx skills add lurenyi2025/protolink --skill protolink --agent both -g -y
```

After installation, return to your AI IDE and say:

```txt
安装 ProtoLink 服务
```

The bootstrap Skill will install or update the global ProtoLink CLI from
`lurenyi2025/protolink-release`, then install or update the paired Skills:

- `protolink`
- `prd-generator`
- `prd-annotation`

This repository is Skill-only. It does not contain the ProtoLink CLI source code
or release build scripts.
