# Improvement Changelog

This document details the evolution of Scope Defender, highlighting the failure modes encountered in local LLMs (Ollama / Llama3) and the architectural decisions within N8N to mitigate them.

### v1.0 — Manual Baseline

* **What was attempted:** Direct and manual response to demand/pressure emails without data ingestion.
* **Result:** Generic and argumentative responses, incapable of defending deadlines under pressure with concrete data.
* **Decision:** Need to automate scope extraction and cross-reference it with the team's task JSON.

### v1.1 — Automatic Capacity Analysis (The Template Problem)

* **What was attempted:** Analyst Agent evaluating the JSON with the prompt requesting the output in the `"VIAVEL | PARCIAL | INVIAVEL"` format.
* **Failure Mode:** The model literally copied the entire template in the output instead of processing the logical decision, breaking the routing.
* **Solution:** Output Formatting refinement. The prompt was altered to prohibit additional text and force the output of only one of the three keywords on the first line.

### v1.2 — Assignee Identification (Aggregation Blindness)

* **What was attempted:** Forcing the LLM to read the raw task array (18+ items) and calculate the workload for all members.
* **Failure Mode:** The model multiplied the 4 members by 40h, ignoring the already allocated tasks and generating a false positive for availability (160h free).
* **Solution:** Explicit targeting. Prompt rules were modified so the model ignores the loose array and reads only the pre-calculated `capacity_summary` object.

### v1.3 — Deterministic Routing (Math vs. AI)

* **What was attempted:** Using the LLM to mathematically classify whether the final balance (-60h) meant "PARCIAL" or "INVIAVEL", relying on it as a flow router.
* **Failure Mode:** LLM limitation with boolean operators on negative numbers. It classified an extreme negative balance as "PARCIAL".
* **Solution (Key Insight):** LLMs should not be logical routers. The prompt was changed to output only a strict structured object (`{"disponivel": -40, "exigido": 20, "saldo": -60}`). N8N (Switch Node) took over the responsibility of routing the decision based on mathematical expressions (`< 0`), eliminating the margin of error.

### v1.4 — Diplomatic Generation (Context Collapse)

* **What was attempted:** Injecting the raw JSON output from the Analyst directly into the Communicator Agent's prompt to ground the final email.
* **Failure Mode:** The LLM suffered a semantic collapse, generated severe literal translation errors (text looping, swapping "prazo" [deadline] for "prato" [plate]) and lost cohesion.
* **Solution:** Clean variable injection via N8N. The numerical properties were extracted from the JSON using expressions (`{{ JSON.parse($json.conteúdo).saldo }}`) before being passed to the prompt, delivering only clean and readable text to the model. Result: Flawless and irrefutable email.

### v1.5 — The End of Mathematical Hallucinations (Code Node)

* **What was attempted:** Keeping the calculation of balance = available - required within the Analyst LLM's interpretation.
* **Failure Mode:** The model frequently failed at subtraction or got confused by sign rules, generating incorrect balances that broke the flow.
* **Solution (Critical Insight):** LLMs should not perform mathematical calculations. The arithmetic was moved to a native Code node (JavaScript) in N8N. The engine calculates the balance with 100% precision and delivers the processed data to the AI solely for drafting, eliminating mathematical hallucinations.

### v1.6 — Prompt Isolation

* **What was attempted:** Using a single Communicator AI node at the end of the flow, injecting a status variable (VIAVEL or INVIAVEL) into a generic prompt.
* **Failure Mode:** The model tried to mix corporate tones, generating confusing emails that rejected the deadline but expressed thanks in a submissive manner, or accepted the task while creating unreal problems.
* **Solution**: Implementation of Prompt Isolation. The flow was branched into three independent Ollama nodes after the Switch. Each node has a restricted prompt specifically designed for its scenario:
Unfeasible: defensive communication grounded in capacity.
Partial: communication with an explicit risk warning.
Feasible: clean and objective acceptance.
