# slash-criticalthink

English | [日本語](README_ja.md)

## Overview

Modern LLMs (Large Language Models) excel at generating confident, plausible-sounding responses. However, these responses often ignore real-world constraints or contain logical flaws.

The `criticalthink` command is a custom command that embeds healthy skepticism into the dialogue process itself as a countermeasure against AI's "confirmation bias" and humans' "authority bias" of blindly trusting AI responses. By having the AI critically analyze its own previous response, it reveals hidden assumptions and overlooked risks.

### Target Audience

- Developers who routinely use coding agents (Claude Code / Codex CLI, etc.)
- Engineers who want to critically verify AI suggestions rather than accepting them at face value

## Setup

1. Place [`criticalthink.md`](criticalthink.md) in the appropriate directory for your tool:

   - **Claude Code**: `.claude/commands/` (in project root or home directory)
   - **Codex CLI**: `~/.codex/prompts/`

2. Create directory and copy file:

   For Claude Code:

   ```bash
   mkdir -p .claude/commands
   cp criticalthink.md .claude/commands/
   ```

   For Codex CLI:

   ```bash
   mkdir -p ~/.codex/prompts
   cp criticalthink.md ~/.codex/prompts/
   ```

3. Restart your tool (or start a new conversation)

4. Verify it works:
   ```
   User: What is 2 + 2?
   AI: 4
   User: /criticalthink
   ```
   If critical analysis begins, it's working

## Usage

### Basic Usage

Type `/criticalthink` immediately after the AI generates a response. The AI will analyze its previous response from the following perspectives:

- Validity of assumptions
- Logical consistency
- Overlooked risks
- AI-specific failure patterns (problem avoidance, happy path bias, over-engineering, hallucination, etc.)

### Usage Examples

#### Case 1: When receiving a complex architecture proposal

```
User: I want to implement microservices, how should I design it?
Claude: I propose a configuration with Kubernetes + Istio + gRPC as follows...
       [Detailed design proposal]
User: /criticalthink
Claude: ### 1. Core Thesis & Confidence Score (Initial)
       - **1-1. Core Thesis:** A Kubernetes-based full-stack solution
         is optimal for microservices architecture
       - **1-2. Initial Confidence:** 8/10

       ### 2. Foundational Analysis: Assumptions & Context
       - **2-1. High-Impact Assumptions:**
         1. Team has Kubernetes operational experience
         2. Traffic scale justifies orchestration adoption
         3. Initial investment and learning costs are acceptable
       ...
```

#### Case 2: When receiving a response claiming a "perfect solution"

```
User: How should I implement rate limiting for this API?
Claude: You can implement it perfectly with Redis + Sliding Window algorithm.
       With the following code...
User: /criticalthink
Claude: ### 3. AI-Specific Pitfall Analysis
       - **3-2. "Happy Path" Bias:** Fail
         Reason: No consideration of fallback strategy during Redis failure
       - **4-1. Overlooked Risks:**
         1. Redis becomes a SPOF (Single Point of Failure)
         2. Network latency affects all requests
         3. Redis memory usage estimation not performed
       ...
```

#### Case 3: When proposed major refactoring of existing code

```
User: I want to modernize this legacy code
Claude: Let's rewrite everything in TypeScript + React Hooks...
User: /criticalthink
Claude: ### 3. AI-Specific Pitfall Analysis
       - **3-3. Over-Engineering:** Fail
         Reason: Proposing complete rewrite instead of gradual migration
       - **4-2. Alternative Scenarios:**
         Gradual migration strategy: First add type definition files (.d.ts),
         then gradually TypeScript-ify starting from critical paths.
         This approach reduces business risk and costs.
       ...
```

#### Case 4: When receiving informational summaries without verification

```
User: Review the README.md
Claude: This README describes slash-criticalthink, a custom command for AI coding assistants...
       [Summary of contents]
User: /criticalthink
Claude: ### 2. Foundational Analysis: Assumptions & Context
       - **2-1. High-Impact Assumptions:**
         1. The user wanted a summary rather than specific information extraction
         2. The criticalthink.md file actually exists in this repository
         3. The user has context about what slash commands are
       - **2-2. Contextual Integrity:** I assumed the user's intent without clarification:
         "Review" could mean summarize, analyze quality, check for errors, verify
         completeness, or prepare to use/modify.

       ### 3. AI-Specific Pitfall Analysis
       - **3-2. "Happy Path" Bias:** Fail
         Reason: Didn't verify whether criticalthink.md actually exists or check
         if .claude/commands/ directory is present

       ### 5. Synthesis & Revised Recommendation
       - **5-3. Actionable Next Step:** First, clarify what aspect of the README
         you want reviewed: (a) content summary (already done), (b) verify the
         slash command is installed and working, (c) critique the documentation
         quality, or (d) help you set it up/use it.
```

### Combined Use with Checkpoint Feature (Recommended)

Since the `/criticalthink` command makes the AI self-criticize, **erroneous concerns or overly negative analysis may remain in the dialogue history and distort the AI's subsequent thinking** (context contamination).

To avoid this, the following workflow is recommended:

1. Receive a proposal from the AI
2. **Create a checkpoint** (Claude Code: `/checkpoint`, Codex CLI: save session)
3. Run `/criticalthink` for critical analysis
4. Review the analysis results and note only useful points
5. **Return to the checkpoint** (discard critical analysis history)
6. Request improvements from the AI based on noted points

This allows you to benefit from critical thinking while avoiding context contamination.

## Design Philosophy

### General Framework

Based on a general critical thinking framework applicable to any claim.

- **Identify the claim:** Identify the key points of the AI's own response
- **Extract assumptions:** Expose implicit assumptions behind the response
- **Verify logical fallacies:** Look for logical leaps or contradictions
- **Present alternatives:** Offer alternative perspectives or solutions not considered

### AI-Specific Failure Pattern Analysis

The general framework alone risked falling into "formal and superficial criticism." To overcome this, we incorporated failure patterns specific to LLMs and coding agents into the analysis items.

- **Problem avoidance:** Is it pretending to solve core problems with mocks or placeholders?
- **Happy path fixation:** Is it ignoring error handling or edge cases?
- **Over-engineering:** Is it proposing unnecessary libraries or complex solutions?
- **Hallucination:** Is it fabricating non-existent functions or APIs?
- **Context ignorance:** Is it ignoring constraints from previous dialogue?

For these items, the command asks the AI for explicit Pass/Fail self-evaluation.

## Caveats and Limitations

### Use as Training Wheels for Thinking

This command is a guideline for finding gaps in your own thinking. Use AI's critical analysis not as the "correct answer" but as a tool to gain new perspectives.

### Responsibility

The final judgment is always made by humans. AI's criticism is also just one generated artifact that should be critically examined. Pay special attention to:

- AI cannot fully recognize errors in its own responses
- Critical analysis itself may introduce new biases
- May include overly conservative warnings (alerts for areas that are actually fine)

### Spirit of Mutual Criticism

Users doubt AI responses while simultaneously using AI (and commands) to doubt their own assumptions and biases. This mutual criticism process leads to more robust decision-making.

### Performance Impact

Critical analysis consumes additional tokens and increases response time. Rather than applying it to all responses, selective use is recommended before important decisions such as:

- Architecture decisions
- Large-scale refactoring
- Security or performance-related implementations
- Introduction of external dependencies

## Related Work: Comparison with CQoT

The approach of introducing a self-critical process to AI to improve its reasoning capabilities has also attracted attention in academic research. Particularly notable is the paper "Critical-Questions-of-Thought (CQoT)" by Federico Castagna et al.

### Shared Purpose

CQoT and `/criticalthink` share the common purpose of "improving the quality and reliability of output by having AI perform metacognitive self-evaluation." Both introduce a step to verify the reasoning process behind responses rather than simply generating a single answer and stopping.

### Fundamental Differences in Approach

While sharing a common purpose, they differ in their objectives and methodologies.

| Comparison    | **CQoT (Academic Paper)**                                                          | **`/criticalthink` (This Tool)**                                                                |
| :------------ | :--------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------------------- |
| **Main Goal** | **Pursuit of Logical Soundness**                                                   | **Verification of Practical Viability**                                                         |
| **Domain**    | Math problems, logic puzzles, etc., **domains with clear correct answers**         | Software design, refactoring, etc., **domains with trade-offs**                                 |
| **Timing**    | **Integrated, automated pipeline before** answer generation                        | **Post-hoc analysis tool after** answer generation                                              |
| **Criteria**  | Universal logical principles<br>(e.g., Are premises clear? Is conclusion derived?) | AI and development-specific failure patterns<br>(e.g., happy path bias, over-engineering, SPOF) |

### Uniqueness of `/criticalthink`

In summary, while CQoT aims to make AI a better **"logician,"** `/criticalthink` aims to make AI a wiser **"design review partner"** for developers. This tool specializes in uncovering real-world risks, costs, and trade-offs—including alternative approaches—that cannot be measured by theoretical correctness alone, which developers face in practice. While CQoT is a closed-loop system that automatically corrects flaws in the reasoning process, `/criticalthink` is an open-loop tool that provides insights for humans to make final judgments.
