# OCSP · Original Character Skill Producer

> 把你的角色创意变成可运行的行为程序。不是角色卡，不是设定集，是可执行的 agent 行为包。

OCSP 是 [CSP](https://github.com/MiFann/csp) 的姊妹工具。CSP 从互联网已有角色蒸馏行为，OCSP 从你的描述**创造**原创角色并蒸馏行为。

## 和 CSP 的关系

| | CSP | OCSP |
|------|-----|------|
| 角色来源 | 已有作品的已有角色 | 用户描述 + AI 共创 |
| 资料获取 | 互联网检索 + 用户材料 | AI 从描述推演 + 用户打磨 |
| 输出格式 | SKILL.md + manifest + references | 相同格式，完全兼容 |

## 工作流程

7 个阶段，5 个共创检查点：

```
描述输入 → 设定共创 → 性格共创 → 表达共创 → 关系与场景 → 行为蒸馏 → 组装验证
              🔍          🔍          🔍           🔍           🔍
```

每个 🔍 标记处会停下来让你确认或调整，确保最终产出符合你的预期。

## 产出

每个角色生成一个完整的 Skill 目录：

```
<角色名>/
├── SKILL.md              # 可运行的角色行为 Skill
├── manifest.json         # 元数据
└── references/
    ├── sources.json      # 创作依据记录
    ├── distillation.md   # 行为蒸馏过程
    ├── quality-report.json
    └── research/
        ├── 01-setting.md
        ├── 02-personality.md
        ├── 03-expression.md
        ├── 04-relationships.md
        └── 05-key-scenes.md
```

## 安装

将整个仓库作为 Skill 安装到你的 AI 助手中。

生成的原创角色 Skill 和 CSP 产出格式完全兼容，可互换部署使用。

## 目录结构

```
Original-Character-Skill-Producer/
├── SKILL.md                          # OCSP Skill 定义
├── skill-template.md                 # 生成角色 SKILL.md 的模板
├── references/
│   ├── manifest-template.json        # 角色 manifest 模板
│   └── sources-entry-template.json   # 创作依据条目模板
└── docs/
    └── superpowers/
        ├── specs/                    # 设计规格
        └── plans/                    # 实现计划
```

## License

MIT
