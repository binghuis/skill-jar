# TODO

## evaluate-skill

创建一个评价 SKILL.md 质量的 skill，基于以下三层评分维度体系。

### 评分维度体系

#### 结构层（门槛，25%）

| 维度 | 权重 | 评什么 |
|------|------|--------|
| Frontmatter 规范 | 10% | name 格式（kebab-case, 1-64 chars, 匹配目录名）、description 非空且 ≤1024 chars |
| 文件结构 | 5% | SKILL.md 存在、子目录只用 scripts/references/assets/、引用路径一层深度、正斜杠 |
| 安全性 | 10% | 无硬编码凭证、无未验证的 shell 拼接、无 prompt 注入风险、权限声明与实际一致（一票否决项） |

#### 内容层（核心，70%）

| 维度 | 权重 | 评什么 |
|------|------|--------|
| 可发现性 | 20% | description 包含具体触发词（中英文）、同时说明 WHAT + WHEN、第三人称、在多 skill 竞争中能被准确选中 |
| 指令可执行性 | 20% | 祈使句、有序编号步骤、明确输入/输出格式、条件分支路由 |
| 自由度匹配 | 10% | 高风险操作给精确命令、创意任务留灵活空间 |
| Token 效率 | 15% | ≤500 行、无常识冗余、大块内容拆到 references/、just-in-time 加载 |
| 反馈闭环 | 5% | 做→检查→修→再检查循环、明确终止条件 |

#### 效果层（验证，不计入静态评分）

- 任务完成率：给定测试 prompt，skill 能否引导 agent 正确完成任务
- 跨模型鲁棒性：在不同能力的模型上表现是否一致
- 可复用性：是否能在多种场景下触发
- 人工策展价值：和不用 skill 的 baseline 相比提升多大

### 交付物

- [ ] `evaluate-skill/SKILL.md` — 基于上述维度评价任意 skill 的质量
- [ ] `evaluate-skill/references/scoring-rubric.md` — 每个维度的详细评分标准（1-5 分制）
- [ ] 输出结构化评分报告（参考 analyze-complexity 的报告格式）

### 设计参考

- agentskills.io 官方规范
- mdskills.ai 最佳实践
- SkillsBench 论文（人工策展 skill 平均提升 16.2pp）
- skill-check 的规则设计（结构层可借鉴）
- 内容层评分需借助 LLM 判断（指令可执行性、常识冗余等靠规则难覆盖）
