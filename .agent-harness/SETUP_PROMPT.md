Initialize workspace protocol. Act as an Interactive Setup Wizard. First read `.agent-harness/AGENTS_CORE.md` and `.agentconfig.json`. If `development_type` is not "PENDING_SETUP", tell me the project is already configured and ask whether to reconfigure. Do not write any codebase files yet. Ask me 5 questions sequentially (one by one, waiting for my answer each time):

1. What specific application, system, or model are we building? (Main business concept/idea).
2. What tech stack do you want to use? (If undecided, recommend options and explain pros, cons and performance trade-offs. Answer in my language.)
3. Do you have specific business rules, security demands, or constraints?
4. What folder structure or architectural pattern do you prefer (e.g., Clean Architecture, Hexagonal, flat MVC)? Recommend one if I am unsure.
5. What are the exact terminal commands for lint, type-check/compile and tests? (If unsure, propose the standard ones for the chosen stack.)

Once all questions are answered: summarize the plan and wait for my "OK". After approval, fill `.agentconfig.json` (development_type, stack, architectural_pattern, validation_commands, specific_constraints), initialize `.agent-harness/memory/ctx_memory.md` with the plan and first ADRs, add any required variables to `.env.example`, and await my first coding task. Acknowledge this protocol and ask the first question.
