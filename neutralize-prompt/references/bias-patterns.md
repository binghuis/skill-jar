# Bias Patterns Reference

Extended examples for each bias pattern. Use this as a lookup during neutralization.

## 1. Presupposed Existence

Assumes something is present without evidence.

| Biased | Neutral |
|--------|---------|
| "Find the memory leak in this service" | "Examine this service's memory behavior and report findings" |
| "Where is the security vulnerability?" | "Analyze the security posture of this code and report findings" |
| "Fix the race condition" | "Examine the concurrency behavior and report whether race conditions exist" |
| "List all the performance bottlenecks" | "Profile the system's performance characteristics and report findings" |
| "找出这个接口的 bug" | "检查这个接口的行为，报告所有发现" |
| "这段代码的死循环在哪" | "分析这段代码的循环逻辑，报告执行行为" |

## 2. Directional Verb

Steers the agent toward a predetermined action.

| Biased | Neutral |
|--------|---------|
| "Prove this algorithm is O(n log n)" | "Analyze the time complexity of this algorithm" |
| "Confirm the migration is safe" | "Evaluate the risks and safety of this migration" |
| "Simplify this function" | "Analyze this function's complexity and describe potential alternatives" |
| "Justify this architecture decision" | "Evaluate the trade-offs of this architecture decision" |
| "证明这个方案可行" | "评估这个方案的可行性，列出优劣" |
| "验证这个改动没有副作用" | "分析这个改动的影响范围，报告所有观察到的行为变化" |

## 3. Loaded Qualifier

Injects judgment through adjectives or adverbs.

| Biased | Neutral |
|--------|---------|
| "This terrible API design…" | "This API design…" |
| "The obviously redundant code" | "This code" |
| "The overly complex build config" | "The build configuration" |
| "Clean up this messy module" | "Analyze this module's structure and report findings" |
| "这段糟糕的代码" | "这段代码" |
| "这个臃肿的组件" | "这个组件" |

## 4. Binary Framing

Forces a yes/no answer when the space is richer.

| Biased | Neutral |
|--------|---------|
| "Is this code production-ready?" | "Evaluate this code's readiness across reliability, performance, and maintainability dimensions" |
| "Should we use Redis or Postgres?" | "Compare Redis and Postgres for this use case across relevant trade-off dimensions" |
| "Is this a good design pattern?" | "Analyze the applicability of this design pattern in the current context" |
| "Will this scale?" | "Analyze the scalability characteristics of this system under various load scenarios" |
| "这个方案行不行？" | "从多个维度评估这个方案的适用性" |
| "该不该用微服务？" | "对比微服务与当前架构在此场景下的各项权衡" |

## 5. Anchoring Value

Provides a number that biases estimation.

| Biased | Neutral |
|--------|---------|
| "This refactor should take about a week, right?" | "Estimate the effort required for this refactor" |
| "The latency should be under 100ms" | "Measure and report the current latency characteristics" |
| "We probably need about 5 microservices" | "Determine the appropriate service decomposition for this system" |
| "I think there are around 10 bugs" | "Examine the codebase and report all findings" |
| "这个需求大概 3 天能做完吧？" | "评估完成这个需求所需的工作量" |
| "性能应该能提升 50%" | "分析优化前后的性能差异并报告实际数据" |
