# Coderift

**AI code review that understands your entire codebase — not just the diff.**

Most code review tools operate on diffs. Coderift operates on dependency graphs. It parses your repository into an AST-derived call graph, giving GPT-4o full cross-function context before surfacing any finding. A second auditor agent then filters the reviewer's output, eliminating contradictory and low-signal findings before they reach you.

The result: logic bugs that span multiple functions — invisible to line-level review — get caught before they reach production.

---

## The Problem With Standard Code Review

Line-level review has a structural blind spot. A bug that exists across three interdependent functions looks clean when you inspect any one of them in isolation. Static analysis tools like Pylint and Radon flag syntax and style — they do not model cross-function logic. Human reviewers face the same constraint: they see the diff, not the dependency graph.

Coderift was built to fix this.

---

## Architecture

### 1. Graph-RAG over AST-Derived Call Graphs

When code is submitted for review, Coderift parses the repository into an abstract syntax tree and derives a call graph capturing function-level dependencies. Rather than passing a raw diff to the LLM, it retrieves:

- **Dependency context** (`hops=2`): functions the target code calls, and the functions those call
- **Impact context** (`hops=1`): functions that call into the target code and would be affected by changes

This gives the model repository-wide context that no diff-based tool can provide.

### 2. Dual-Agent Reviewer-Auditor Pipeline

Code review runs through two sequential GPT-4o agents:

**Agent 1 — Reviewer:** Receives the target code, static analysis output (Pylint + Radon), and the full graph context. Returns a structured JSON draft of findings including logic bugs, redundant paths, and null guard issues.

**Agent 2 — Auditor:** Receives everything Agent 1 saw, plus Agent 1's draft. Its sole responsibility is to validate each finding, eliminate false positives, resolve contradictions, and produce a final structured verdict.

This pipeline reduced false positives by **34%** compared to static analysis alone, measured across logic bugs, redundant paths, and null guard findings.

### 3. Static Analysis Cross-Validation

LLM findings are cross-validated against Pylint and Radon output using structured prompt constraints. Findings that contradict static analysis are flagged and filtered by the auditor, preventing contradictory entries from reaching the developer.

---

## Delivery

| Surface | Implementation |
|---|---|
| **VS Code Extension** | TypeScript extension surfacing findings as native diagnostics with one-click Quick Fix suggestions |
| **GitHub PR Agent** | Webhook-driven FastAPI service that automatically posts inline AI-generated review comments on pull requests |
| **Backend** | Python + FastAPI, containerized with Docker, deployed on Railway |
| **CI/CD** | GitHub Actions pipeline for automated builds and VS Code Marketplace releases |

---

## Stack

| Layer | Technology |
|---|---|
| Extension | TypeScript, VS Code Extension API |
| Backend | Python, FastAPI |
| AI Engine | GPT-4o (dual-agent), structured JSON outputs |
| Graph Analysis | AST parsing, call graph traversal |
| Static Analysis | Pylint, Radon |
| DevOps | Docker, GitHub Actions, Railway |

---

## Getting Started

1. Install from the [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=AdityaMurali.coderift)
2. Add your OpenAI API key under **Settings → Coderift**
3. Open any Python file and run `Coderift: Review Current File` (`Cmd+Shift+P`)
4. Hover over any diagnostic → click **Quick Fix** to apply the suggestion

---

## License

MIT — see `LICENSE` for details.

---

Built by [Aditya Murali](https://github.com/adityamurali99)
