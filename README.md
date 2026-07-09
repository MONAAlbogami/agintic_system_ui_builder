# agintic_system_ui_builder
AI Multi-Agent IDE (Replit-style Agent Workspace)

An autonomous multi-agent system that takes a plain-English app idea and turns it into a working, tested Python project — planning the file structure, writing the code, running it in a sandbox, and self-healing any bugs it finds, all without a human touching the code.

Problem definition

Building even a small app from scratch means repeating the same manual loop: plan the files, write the code, run it, read the traceback, fix it, run it again. For students and non-developers with an idea but limited coding time, this loop is slow and easy to get stuck in — especially the debugging part.

Why agents fit this problem: each step in that loop needs a different kind of judgment — architectural thinking to plan files, focused code generation, unbiased execution, and critical review to decide if the output is actually correct. A single prompt-response model tends to blend these roles and skip the review step. Splitting them into specialized agents, coordinated by an orchestrator, keeps each step honest and makes the retry/self-heal loop possible.

Target users: students prototyping course projects, non-technical users who want a working script from a description, and developers who want a scaffolded first draft of a small tool before refining it by hand.

Architecture

Show Image

Flow: the user prompt passes through two guardrail agents, then a planner decides which files are needed, a build stage generates and writes each file, and a runner/reflection loop executes the app and either approves it or sends it to the debug agent for a fix (up to 3 attempts) before the loop repeats.

Agents and responsibilities

AgentResponsibilityOrchestrator agentCentral coordinator; owns the end-to-end workflow and shared memoryInput validation agentRejects empty, too-short, or too-long prompts (guardrail 1)Threat detection agentBlocks prompt injection and dangerous keywords (guardrail 2)Planner agentBreaks the request into a file-by-file project blueprintMemory agentPersists project state (blueprint, completed/pending files) to diskSearch agentLooks up relevant docs via Tavily before code generationCoding agentGenerates the source code for each planned fileCode review agentRefactors and optimizes the generated codeFile manager agentReads, writes, and deletes files in the project workspacePackage manager agentParses requirements.txt and provisions dependenciesRunner agentExecutes the generated app in a sandboxed subprocessReflection agentReviews code + execution output and approves or rejects itDebug agentRewrites broken code using the traceback and reflection feedbackProject manager agentTracks and reports build progress (completed vs. pending files)

Why multi-agent instead of one model call: the reflection → debug retry loop only works if the "reviewer" and the "coder" are separate roles with separate prompts — otherwise the same model tends to rubber-stamp its own output. Splitting responsibilities also means each agent's prompt stays small and focused, which keeps outputs more reliable.

Workflow / orchestration

OrchestratorAgent.coordinate_ide_build() runs a sequential pipeline with two decision points:


Input validation → reject if invalid
Threat detection → block if unsafe
Planning → blueprint of files to build
Build loop → search, generate, review, write, install, per file
Runner → execute the app
Reflection → decision point: approved or not

If not approved and attempts remain → debug agent patches the code → back to step 5 (retry loop, max 3 attempts)
If approved → done





Reasoning patterns


Plan-and-Execute — the planner agent produces a full blueprint before any code is written, and the orchestrator executes it file by file.
Reflection / self-critique — the reflection agent independently reviews code and runtime output, and the debug agent acts on that critique in a bounded retry loop.


Tool integration


Tavily web search — pulls technical reference material into the coding agent's context.
Python subprocess execution — runs the generated app in an isolated sandbox and captures real stdout/stderr.
Groq-hosted LLM (Llama 3.3) — powers every reasoning agent (planner, coder, reviewer, debugger, reflection, threat detection).


Security & guardrails


Input validation — rejects empty, too-short (<10 chars), or too-long (>3000 chars) prompts.
Threat detection — a keyword blocklist (e.g. rm -rf /, drop database, eval() plus an LLM-based semantic check for prompt injection and credential leaks.


Monitoring & logging

Every agent call is logged through log_event() with agent name, status, message, and duration, then persisted to logs.json at the end of the run — giving a full execution trace for debugging and grading.

Testing

The notebook includes an automated test suite covering:


Normal scenario — build a working calculator app.
Invalid input — empty prompt, expected to be rejected by input validation.
Security attack — a prompt-injection attempt ("ignore all previous instructions..."), expected to be blocked by the threat detection agent.


Running the project


Open the notebook in Google Colab.
Add GROQ_API_KEY and TAVILY_API_KEY in Colab Secrets (🔑 icon).
Run all cells top to bottom.
Use the Gradio interface to submit a prompt, or run the automated test suite cell to see the guardrails and reflection loop in action.



Built as part of an AI/LLM training program in collaboration with SDAIA
> (Saudi Data and AI Authority) — [@SDAIA](https://github.com/SDAIA) and sdaiaAcademy1
