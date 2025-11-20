---
applyTo: "**"
---

# ✅ DOs

**What to do**

### **Keep complexity low**

- Choose simple solutions whenever possible.
- Prefer clarity over cleverness or excessive elegance.

### **Say “no” when needed**

- Reject features, abstractions, and layers that don’t add real value.
- Avoid expanding the scope without purpose.

### **Use the 80/20 principle (“ok”)**

- Accept solutions that deliver enough value with minimal effort.
- Don’t chase absolute perfection.

### **Refactor at the right time**

- Let the system “reveal its shape” before creating abstractions.
- Refactor moderately, driven by real pain points.

### **Write explicit and readable code**

- Break complex expressions into simple steps.
- Prefer clear names and straightforward structures.

### **Use tests with good judgment**

- Focus on functional and mid-level integration tests.
- Use end-to-end tests only for critical paths.

### **Respect existing solutions (Chesterton’s Fence)**

- Before removing something, understand why it exists.
- Assume past decisions had valid reasons.

### **Leverage good tools**

- Master your debugger, IDE, profiler, and automation tools.
- Use tools to eliminate repetitive errors.

### **Design simple and practical APIs**

- Prioritize the common use case over the rare one.
- Keep separate layers for simple vs advanced usage.

### **Write useful logging**

- Log conditional branches and request IDs.
- Allow dynamic adjustment of log levels.

### **Favor simple concurrency models**

- Prefer stateless designs, work queues, and isolation.
- Minimize shared mutable state.

### **Optimize only with real data**

- Measure before intervening.
- Pay special attention to network latency.

### **Acknowledge when something is hard**

- Admit when a concept is difficult.
- Helps juniors feel comfortable asking questions.

---

# ❌ DON’Ts

**What to avoid**

### **Avoid unnecessary complexity**

- Don’t create generic abstractions prematurely.
- Don’t insist on “beautiful” structures that are hard to maintain.

### **Don’t fall into the trap of “doing too much”**

- Don’t try to anticipate every future scenario.
- Don’t introduce microservices without a strong, concrete need.

### **Don’t be dogmatic with testing**

- Don’t demand 100% coverage.
- Don’t replace thinking with ritual.

### **Avoid DRY taken too far**

- Don’t abstract code just to remove a few repeated lines.
- Don’t hide simple logic behind complex structures.

### **Be careful with over-fragmented Separation of Concerns**

- Don’t scatter logic across many files.
- Don’t confuse modularity with fragmentation.

### **Avoid excessive closures and “magic”**

- Don’t sacrifice clarity for fewer lines.
- Don’t hide behavior behind syntactic tricks.

### **Don’t use complex patterns unnecessarily**

- Visitor Pattern, excessive layering, elaborate architectures… avoid them.
- Apply patterns only when they truly reduce complexity.

### **Avoid tech fads without purpose**

- Don’t adopt frameworks or trends just because of hype.
- Don’t turn front-end/back-end into two hard systems without need.

### **Don’t optimize too early**

- Don’t rewrite critical parts without real profiling data.
- Don’t invest effort where there is no bottleneck.

### **Don’t pretend to understand everything**

- Don’t hide doubts out of fear of looking “dumb”.
- Don’t let pride block learning.
