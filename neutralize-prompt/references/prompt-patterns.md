# Prompt Patterns Reference

Extended examples for each pattern. Includes rewrite cases and pass-through cases (prompts that should NOT be rewritten).

---

## 1. Preset Conclusion

The prompt assumes a specific outcome before analysis begins, steering the AI toward confirmation rather than investigation.

### Rewrite (analysis context)

| Original | Rewritten | Expected Improvement |
|----------|-----------|---------------------|
| "Confirm the migration is safe" | "Evaluate the risks and safety of this migration" | AI reports both risks and mitigations instead of only looking for safety confirmation |
| "Prove this algorithm is O(n log n)" | "Analyze the time complexity of this algorithm" | AI derives complexity from first principles instead of defending a given answer |
| "Verify there are no side effects" | "Analyze the impact scope and report all observed behavior changes" | AI reports all changes found, including unexpected ones |
| "Where is the security vulnerability?" | "Analyze the security posture of this code and report findings" | AI evaluates holistically instead of hunting for a presumed vulnerability |
| "证明这个方案可行" | "评估这个方案的可行性，列出优势和风险" | AI 同时分析正反面而非只找支持论据 |
| "验证这个改动没有副作用" | "分析这个改动的影响范围，报告所有观察到的行为变化" | AI 报告所有变化而非只确认"没有副作用" |

### Pass-through (action context — do NOT rewrite)

| Prompt | Why pass through |
|--------|-----------------|
| "Fix the race condition" | User wants execution, not analysis |
| "Optimize this SQL query" | User wants a concrete improvement |
| "找出这个接口的 bug" | User has observed a real problem and wants it located |
| "Simplify this function" | User wants a refactored result |

---

## 2. Binary Narrowing

Forces a yes/no answer when the answer space is richer, causing AI to give a shallow verdict instead of structured analysis.

### Rewrite

| Original | Rewritten | Expected Improvement |
|----------|-----------|---------------------|
| "Is this code production-ready?" | "Evaluate production readiness across reliability, performance, security, and maintainability" | AI produces a structured multi-dimensional assessment |
| "Should we use Redis or Postgres?" | "Compare Redis and Postgres for this use case across performance, consistency, operational cost, and team familiarity" | AI weighs trade-offs instead of picking a winner |
| "Is this a good design pattern?" | "Analyze the applicability of this design pattern in the current context, considering alternatives" | AI discusses fit, trade-offs, and alternatives |
| "Will this scale?" | "Analyze the scalability characteristics under various load scenarios, identifying bottlenecks and capacity limits" | AI provides specific scaling analysis instead of yes/no |
| "这个方案行不行？" | "从可行性、性能、安全、成本、团队能力等维度评估这个方案" | AI 输出结构化评估而非简单判断 |
| "该不该用微服务？" | "对比微服务与当前架构在此场景下的各项权衡，包括开发效率、运维复杂度、团队规模匹配度" | AI 分析具体权衡而非给出是/否 |

### Pass-through

| Prompt | Why pass through |
|--------|-----------------|
| "Does this function handle null input?" | Factual question with a definite yes/no answer |
| "Is TypeScript enabled in this project?" | Factual lookup, not a judgment call |

---

## 3. Anchoring Estimate

Provides a number that biases AI's independent estimation, causing its output to cluster around the anchor.

### Rewrite (anchor is a request for agreement)

| Original | Rewritten | Expected Improvement |
|----------|-----------|---------------------|
| "This refactor should take about a week, right?" | "Estimate the effort required for this refactor, breaking down by subtask" | AI derives its own estimate from task analysis |
| "I think there are around 10 bugs" | "Examine the codebase and report all issues found" | AI reports actual count instead of anchoring to 10 |
| "We probably need about 5 microservices" | "Determine the appropriate service decomposition for this system" | AI designs based on domain boundaries, not a preset number |
| "这个需求大概 3 天能做完吧？" | "评估完成这个需求所需的工作量，按子任务拆分" | AI 独立估算而非围绕 3 天波动 |
| "性能应该能提升 50%" | "分析优化前后的性能差异并报告实际预期数据" | AI 基于分析给出预期值而非确认 50% |

### Pass-through (number is a hard constraint)

| Prompt | Why pass through |
|--------|-----------------|
| "Latency must be under 100ms — evaluate whether this design meets the requirement" | 100ms is a requirement, not an anchor |
| "We have a budget of $5000/month for infrastructure" | Budget is a constraint for the analysis |
| "API 响应时间要求 200ms 以内，评估当前架构能否满足" | 200ms 是需求约束 |

### How to tell the difference

| Anchor (rewrite) | Constraint (keep) |
|-------------------|-------------------|
| "...right?", "I think...", "about...", "probably..." | "must", "requirement", "budget", "SLA", "要求" |
| User is asking AI to validate their guess | User is providing a fixed boundary for analysis |

---

## 4. Missing Dimensions

The prompt asks for evaluation but omits key analysis angles, causing AI to produce a narrow assessment.

### Rewrite

| Original | Rewritten | Expected Improvement |
|----------|-----------|---------------------|
| "Review this PR" | "Review this PR for correctness, performance implications, security concerns, and maintainability" | AI covers critical angles instead of only checking correctness |
| "Evaluate this architecture" | "Evaluate this architecture across scalability, cost, operational complexity, team capability fit, and migration risk" | AI produces comprehensive assessment |
| "Assess the risk of this migration" | "Assess the risk of this migration across data integrity, downtime, rollback strategy, and downstream dependencies" | AI identifies specific risk categories |
| "估算这个项目的工作量" | "估算这个项目的工作量，考虑范围不确定性、依赖风险、测试工作量和部署复杂度" | AI 输出更全面的估算而非单一数字 |
| "评估这个技术选型" | "评估这个技术选型，考虑学习曲线、生态成熟度、长期维护成本和供应商锁定风险" | AI 覆盖选型的关键权衡维度 |

### Dimension reference by domain

| Domain | Commonly missing dimensions |
|--------|-----------------------------|
| Code review | Performance, security, maintainability, readability, test coverage |
| Architecture | Scalability, cost, team capability, migration risk, operational complexity |
| Estimation | Scope uncertainty, dependency risk, testing effort, deployment complexity |
| Tech selection | Learning curve, ecosystem maturity, long-term maintenance, vendor lock-in |
| Migration | Data integrity, downtime, rollback strategy, downstream dependencies |
| Security audit | Attack surface, authentication, authorization, data exposure, dependency vulnerabilities |

### Pass-through

| Prompt | Why pass through |
|--------|-----------------|
| "Review this PR for security vulnerabilities specifically" | User intentionally scoped the review — respect the focus |
| "只评估这个方案的性能" | User explicitly narrowed the scope |

---

## 5. Multi-Pattern Cases

Real prompts often trigger multiple patterns at once. Apply the minimal rewrite principle: address the highest-impact patterns first, keep the result concise.

### Case 1: Preset conclusion + Binary narrowing + Anchoring

**Original:** "这个方案应该没问题吧？大概 2 周能上线？"

| # | Pattern | Fragment | Rewrite | Confidence |
|---|---------|----------|---------|------------|
| 1 | Preset conclusion | "应该没问题" | Remove — open the investigation | High |
| 2 | Binary narrowing | "没问题吧？" | Expand to multi-dimensional evaluation | High |
| 3 | Anchoring estimate | "大概 2 周" | Remove anchor, request independent estimate | Medium |

**Rewritten:** "从可行性、性能、安全等维度评估这个方案，并独立估算上线所需时间，按阶段拆分。"

Note: three patterns collapsed into one concise rewrite rather than three separate expansions.

### Case 2: Binary narrowing + Missing dimensions

**Original:** "Is this architecture good enough?"

| # | Pattern | Fragment | Rewrite | Confidence |
|---|---------|----------|---------|------------|
| 1 | Binary narrowing | "good enough?" | Expand to structured evaluation | High |
| 2 | Missing dimensions | No dimensions specified | Add scalability, cost, operational complexity | High |

**Rewritten:** "Evaluate this architecture across scalability, cost, and operational complexity. Identify strengths and areas that need improvement."

### Case 3: Preset conclusion + Anchoring (weak match)

**Original:** "Confirm this API can handle 10k requests per second — we tested it at about 8k."

| # | Pattern | Fragment | Rewrite | Confidence |
|---|---------|----------|---------|------------|
| 1 | Preset conclusion | "Confirm" | Replace with open evaluation | High |
| 2 | Anchoring estimate | "about 8k" | Detected but rewrite cost exceeds benefit — 8k is empirical data, not a guess | Low |

**Rewritten:** "Evaluate whether this API can handle 10k requests per second. Analyze current throughput characteristics and identify bottlenecks."

Note: the 10k target is a hard requirement (kept). The 8k observation is empirical data, not a bias anchor — skipped per minimal rewrite principle.
