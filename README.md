The Math That Kills AI Agents in Production

96% step accuracy sounds safe. Run 20 steps in a chain, and your system reliability drops to 44%. This is not a model problem. This is an infrastructure problem.
You built an agent. It passed every test. The demo worked. The LLM reasoned. The tools fired. The output looked correct.
Then it hit production. And it broke. Not with a crash. Not with an error code. It broke with a plausible, confident, completely wrong answer that got committed to your database and propagated through every downstream step.
No alarm fired. No retry triggered. The run finished with a success status.
This is the number one killer of agent systems in production. And no amount of prompt engineering fixes it.
Let me show you the math.
 The Arithmetic Nobody Talks About

Every step in an agent chain has an accuracy rate. Call it 96%. That sounds high. That sounds production-ready.
But agents run in series. Step 1 feeds step 2. Step 2 feeds step 3. Each step multiplies the reliability of the last.
The formula is simple. It comes from reliability engineering. Lusser's Law.
System Reliability Formula:
R(system) = R(step)^n
Plug in the numbers:
After 10 steps: 0.96 to the 10th power = 66% success rate.
After 20 steps: 0.96 to the 20th power = 44% success rate. You are below a coin flip.
After 30 steps: 0.96 to the 30th power = 29% success rate. Your agent is wrong 7 times out of 10.

This is not a hallucination problem. This is a serial dependency problem. The model did its job at each step. The system had no mechanism to catch the drift.
"Step Succeeded" Is Not "Step Is Correct"

Here is the part that breaks most teams.
An agent calls an API. The API returns HTTP 200. The response parses. The schema validates. The framework marks the step as complete and moves on.
But the data inside the response is wrong. A stale cache. A corrupted field. A hallucinated ID that the tool accepted because it matched the format.
The step succeeded. The output is garbage.
Current agent frameworks treat this as a success. The graph advances. The next step builds on bad data. By step 14, the entire artifact is confidently wrong. And nothing in the pipeline stopped it.
This is not a bug in your prompt. This is a missing layer in your infrastructure.
What the Current Stack Handles (and What It Does Not)

I will be direct about how we think about this.
When we built the system, we looked at the existing tools. LangGraph handles graph traversal and state management. We use it for that. Temporal handles durable execution and retries. We use it for that. LiteLLM handles model routing. We use it for that.
These are solid primitives. They solve real problems. We did not rebuild them.
But none of them solve the problem I just described.
LangGraph does not inspect whether the output of step 7 is factually grounded before passing it to step 8. Temporal does not trigger a compensation transaction when step 14 invalidates the work done in steps 1 through 13. LiteLLM does not enforce a spend cap or a PII guard at the runtime level.
These tools handle execution mechanics. They do not handle execution governance.
That gap is where production systems die.
What Actually Needs to Exist

After working through this problem, we identified five mechanisms that must exist in the infrastructure layer. Not in the prompt. Not in the model. In the runtime.
1. Policy-Gated Validation
Every step output passes through a schema checkpoint before the next step begins. If the shape deviates, execution halts. Not retries. Halts. A human or a compensation routine takes over. This stops hallucination cascades at the point of origin. Step 7 produces a malformed field. The checkpoint catches it. Step 8 never sees it.
2. Sagas with Compensations
Long-running workflows span multiple systems. A database write. An API call. A file upload. If step 14 fails, you cannot just retry step 14. You need to undo steps 1 through 13 in reverse order. This is the Saga pattern from distributed systems. No major agent framework implements it natively for LLM workflows. It must live in the infrastructure.
3. Evidence-Led Audit
Logs are telemetry. They tell you what happened. They do not tell you why a decision was made, from what input, under what policy. When a regulator asks why your agent denied a claim, "check the logs" is not an answer. You need an immutable evidence registry. Every artifact. Every state transition. Every policy evaluation. Tied to a specific step and a specific run.
4. Runtime Enforcement, Not Prompt Guardrails
Guardrails written into a prompt live inside the context window. The model can ignore them. Context bloat pushes them out. A prompt injection overrides them. Policies must live outside the model. In the runtime. The model never sees the policy. It cannot bypass what it cannot read. The runtime intercepts the action, checks it against the policy, and blocks or allows it.
5. Self-Healing Execution
Agents get stuck. A tool returns stale data. The model retries the same call with the same prompt. It loops. For hours. You find out from the invoice. The infrastructure needs hard iteration caps. Timeouts. Budget ceilings. When a step exceeds its retry budget, the runtime kills the loop and routes to a fallback. No human intervention. No overnight billing spike.
The Category

These five mechanisms do not belong in a framework. They do not belong in a prompt. They belong in a dedicated infrastructure layer.
We call this layer the Execution Plane.
Nondeterministic input goes in. Deterministic output comes out.
The model reasons. The execution plane owns the outcome.
Why This Matters for High-Stakes Environments

If you are building a chatbot that summarizes blog posts, none of this matters. Use whatever works.
If you are building an agent that processes insurance claims, reconciles financial transactions, or reviews legal documents, this is not optional.
A silent skip on a compliance check is a regulatory violation. A hallucinated value in a medical record is a liability. An unauthorized write to a production database is a security incident.
These are not edge cases. They are the default behavior of ungoverned agent systems running at scale.
The infrastructure must enforce correctness. Not suggest it. Not prompt for it. Enforce it.
What Comes Next

This post covered the problem and the category. The math. The missing layer. The five mechanisms that must exist in the runtime.
In the next posts, I will go deeper into each mechanism. How we implement Sagas with compensations for LLM workflows. How the evidence registry works. How policy gating replaces prompt guardrails. How the self-healing engine decides when to retry, when to fall back, and when to halt.
If you are building long-running, mission-critical agentic workflows and you are hitting these walls, I want to hear from you.

Escepine-an execution plane for agentic systems. We are onboarding our first design partners now.
https://escepine.vercel.app
