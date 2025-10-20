# Workflow Automation Prompt Files

> ℹ️ **Note**: This directory currently contains only this README. The actual prompt files (`.prompt.md`) are **created by you in Step 5-0** as part of the bootstrap process!

**In Step 5-0, you'll create** three reusable prompt files that enable autonomous agentic workflows:

- `/execute-step` - Autonomously execute GitHub Issue instructions
- `/validate-step` - Verify completion against success criteria
- `/commit-and-push` - Analyze changes and push to feature branch

This README explains **what you'll build** and **why it matters**.

## Architecture

These prompts follow a **shared knowledge pattern** with **two mode strategies**:

- **Prompt Files** (`.prompt.md`) = User-friendly entry points (slash commands)
- **Project Instructions** (`.github/copilot-instructions.md`) = Shared knowledge base with workflow utilities
- **Specialized Chat Modes** = Task-specific AI agents (`tdd-developer`, `code-reviewer`)

### Two Mode Patterns

#### Pattern 1: Mode-Agnostic (No Mode Specified)

When you invoke a prompt like `/commit-and-push`:

1. Collects input from you (branch name, etc.)
2. Uses the **currently active chat mode** (works from any context!)
3. Inherits workflow utilities (gh CLI, Git commands) from project instructions
4. Executes the task using that mode's tools and capabilities

**Why?** Commit tasks are universal utilities that work in any context (TDD, review, etc.)

#### Pattern 2: Mode-Specific (Auto-Switching)

When you invoke prompts like `/execute-step` or `/validate-step`:

1. **Automatically switches** to the specified mode (`tdd-developer` or `code-reviewer`)
2. Collects input from you (issue URL, step number, etc.)
3. Inherits workflow utilities from project instructions
4. Executes the task in the appropriate specialized context

**Why?**
- **Execution** is usually iterative development → `tdd-developer` mode
- **Validation** is always a review task → `code-reviewer` mode
- The prompts know the right context!

### Benefits

- ✅ Demonstrates both mode-agnostic AND mode-specific patterns
- ✅ Two prompts auto-switch (`/execute-step` → tdd-developer, `/validate-step` → code-reviewer)
- ✅ One prompt is universal (`/commit-and-push` works in any mode)
- ✅ Workflow knowledge (gh CLI, Git) available to ALL modes via project instructions
- ✅ Students learn when auto-switching vs. flexibility makes sense
- ✅ Transparent architecture - you can see which prompts auto-switch
- ✅ Memory system updates happen automatically through project instructions (no separate prompt needed)

## What Are Prompt Files?

Prompt files (`.prompt.md`) are reusable, on-demand prompts that can be invoked like slash commands in GitHub Copilot Chat. They:

- Provide user-friendly slash command interfaces
- Collect input variables from users
- Delegate execution to appropriate chat modes
- Enable consistent, repeatable workflows

## Available Prompts

### 📋 `/execute-step` - Autonomous Step Execution (AUTO MODE-SWITCHING!)

**Purpose**: Have Copilot read GitHub Issue instructions and execute them autonomously.

**Special Feature**: This prompt **automatically switches to `tdd-developer` mode** - you don't need to switch manually!

**How to use**:

1. Open Copilot Chat (in ANY mode - it will auto-switch!)
2. Type `/execute-step`
3. Provide issue URL when prompted
4. Watch it automatically switch to `tdd-developer` mode for execution

**What it does**:

- **Automatically switches to `tdd-developer` mode** (most step work involves iterative development!)
- Uses `gh` CLI commands (from project instructions) to fetch the GitHub Issue
- Parses `:keyboard: Activity:` sections from the issue
- Executes tasks systematically using TDD workflow patterns
- Reports completion status
- **Does NOT commit** - that's the job of `/commit-and-push`

**Why auto-switch?** Most step instructions involve implementing features, fixing tests, or iterative development - all activities that benefit from the TDD mindset. By specifying `mode: tdd-developer` in the prompt file, we ensure execution happens in the right context.

**Example workflow**:

```text
You: /execute-step
Copilot: [Auto-switches to tdd-developer mode]
Prompt: [Asks for issue URL]
You: [Paste issue URL]
Copilot: [Uses gh CLI, fetches issue, parses activities, executes in tdd-developer mode]
```

**Key Principle**: This prompt auto-switches to ensure consistent execution context across all steps.

---

### 🚀 `/commit-and-push` - Smart Git Workflow

**Purpose**: Analyze changes, generate meaningful commits, and push to the correct feature branch.

**How to use**:

1. Complete work on a step
2. Type `/commit-and-push` in Copilot Chat
3. Provide branch name when prompted (e.g., "feature/agentic-workflow")

**What it does**:

- Runs `git status` and `git diff` to analyze changes
- Generates conventional commit message (feat:, fix:, chore:, etc.)
- Creates branch if it doesn't exist, or switches if it does
- Stages all changes with `git add .`
- Commits with the generated message
- Pushes to GitHub (triggering automation for next step)

**Example workflow**:

```text
You: /commit-and-push
Copilot: Which branch should I push to?
You: feature/agentic-workflow
Copilot: [Analyzes changes]
Copilot: "I see you've added Copilot customization files. I'll commit with message:
         'feat: add copilot customizations for tdd and code review'
         and push to feature/agentic-workflow. Proceed?"
You: Yes
Copilot: [Creates/switches branch, stages, commits, pushes]
```

---

### ✅ `/validate-step` - Completion Verification (AUTO MODE-SWITCHING!)

**Purpose**: Check if all success criteria for a step have been met.

**Special Feature**: This prompt **automatically switches to `code-reviewer` mode** - you don't need to switch manually!

**How to use**:

1. When you think you've completed a step
2. Type `/validate-step` (in ANY mode - it will auto-switch!)
3. Provide the step number (e.g., "5-0", "5-1")
4. Watch it automatically switch to `code-reviewer` mode for validation

**What it does**:

- **Automatically switches to `code-reviewer` mode** (validation is always a review task!)
- Uses `gh` CLI (from project instructions) to fetch the GitHub Issue
- Finds the specified step in the issue comments
- Extracts success criteria from that step
- Validates each criterion against your current workspace
- Provides ✅/❌ report with specific guidance

**Why auto-switch?** Validation inherently requires a review mindset - checking criteria, analyzing state, providing feedback. By specifying `mode: code-reviewer` in the prompt file, we ensure validation always happens in the right context.

**Example workflow**:

```text
You: /validate-step
Copilot: Which step would you like to validate?
You: 5-0
Copilot: [Uses gh CLI to fetch issue, finds Step 5-0, checks criteria]

## Step 5-0 Validation Results

✅ Created .github/copilot-instructions.md
✅ Created .github/chatmodes/tdd-developer.chatmode.md
✅ Created .github/chatmodes/code-reviewer.chatmode.md
❌ Branch feature/agentic-workflow not found
✅ Files contain proper YAML frontmatter

4 of 5 criteria met. Create the branch with: git checkout -b feature/agentic-workflow
```

---

## The Agentic Workflow Pattern

These three prompts enable a **fully autonomous workflow**:

```text
1. /execute-step → AI auto-switches to tdd-developer and executes instructions
2. /validate-step → AI auto-switches to code-reviewer and verifies completion
3. /commit-and-push → AI commits and pushes (works in any mode)
4. [GitHub Actions automatically posts next step]
```

**Notice:** You don't need to manually switch modes! The prompts handle context automatically.

**Memory Updates**: The AI automatically documents patterns to `.github/memory/patterns-discovered.md` as it works through each step. The Memory System section in your project instructions tells AI to do this naturally - no separate prompt needed!

This demonstrates **real-world agentic development** where AI:

- Automatically selects the right context for each task
- Follows documented procedures autonomously
- Validates its own work
- Manages version control
- Integrates with CI/CD pipelines

## Best Practices

### Architecture: Shared Knowledge Pattern

This system uses a **shared knowledge architecture with smart mode selection**:

```text
┌─────────────────────────────────────────┐
│  User invokes slash command             │
│  /execute-step, /validate-step, etc.    │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Prompt File (.prompt.md)               │
│  - Auto-switches to appropriate mode    │
│    OR uses current mode                 │
│  - Collects user input                  │
│  - Provides clear instructions          │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  Specialized Mode + Project Instructions│
│  - Mode provides tools & capabilities   │
│    * tdd-developer for execution        │
│    * code-reviewer for validation       │
│    * Any mode for commits               │
│  - Project instructions provide:        │
│    * Workflow utilities (gh CLI)        │
│    * Git workflow knowledge             │
│    * Documentation references           │
└─────────────────────────────────────────┘
```

### When to Use Prompts vs Chat Modes

- **Prompts** (`/execute-step`, `/validate-step`, `/commit-and-push`): Quick, single-command execution with structured workflow
  - Most prompts **auto-switch** to the right mode for you
  - Just invoke and provide input - the prompt handles context
- **Chat Modes** (`tdd-developer`, `code-reviewer`): Ongoing conversation and iterative work on specific tasks
  - Use for exploratory work, debugging, or multi-turn conversations
  - Prompts can invoke these modes automatically when needed

### When to Specify Mode in Prompts

**Specify `mode:` when:**

- The task ALWAYS needs the same context
  - Example: `/execute-step` → `tdd-developer` (execution = iterative development)
  - Example: `/validate-step` → `code-reviewer` (validation = review task)
- You want to reduce cognitive load (no manual mode switching required)
- The mode choice is obvious and unchanging for that workflow

**Don't specify `mode:` when:**

- The task is context-independent
  - Example: `/commit-and-push` (works in any mode)
- You want flexibility to use the prompt in different contexts

**Leave mode unspecified when:**

- The task context depends on what the user is doing
- Flexibility is more valuable than consistency
- Multiple modes might be appropriate for the same task

**Example from this session:**

- `/validate-step` → `mode: code-reviewer` (validation is ALWAYS review)
- `/execute-step` → NO MODE (execution context varies by step)
- `/commit-and-push` → NO MODE (git operations are universal)

### Mode Selection Pattern

**For auto-switching prompts** (`/execute-step`, `/validate-step`):

No need to switch modes - the prompts handle context automatically!

```text
Execution  → any mode → /execute-step (auto-switches to tdd-developer)
Validation → any mode → /validate-step (auto-switches to code-reviewer)
```

**For universal utility prompts** (`/commit-and-push`):

Works in any mode - no switching needed:

```text
Committing work → any mode → /commit-and-push (uses current mode)
```

**For chat modes used directly** (no prompts):

When you need ongoing conversation or exploration:

```text
Creating Customizations → copilot-customization → (use chat directly)
Iterative TDD work     → tdd-developer → (use chat directly)
Code quality discussion → code-reviewer → (use chat directly)
```

### Human-in-the-Loop

While these prompts enable autonomous execution:

- **Review** what the AI plans to do
- **Confirm** before major actions (commits, pushes)
- **Learn** from observing the AI's approach
- **Override** if needed

## Technical Details

### Prompt File Format

```markdown
---
description: "Short description shown in autocomplete"
mode: "agent"  # or "ask", "edit"
model: "Claude Sonnet 4"  # optional
tools: ['codebase', 'runCommands', ...]  # available tools
---

# Prompt instructions in Markdown
Use ${input:variable-name} for user input
Reference files with [link](../path/to/file.md)
```

### Variables

Prompts can request input:

- `${input:issue-url}` - Prompts for "issue-url"
- `${input:step-number}` - Prompts for "step-number"
- `${input:branch-name}` - Prompts for "branch-name"

### Tools Available

Common tools used in these prompts:

- `codebase` - Search and understand code
- `runCommands` - Execute terminal commands (including `gh` CLI!)
- `getTerminalOutput` - Retrieve command output
- `editFiles` - Modify existing files
- `createFile` - Create new files
- `problems` - View ESLint/compile errors

**Why `gh` CLI instead of `fetch` tool?**

These prompts use `gh` CLI (GitHub CLI) instead of the `fetch` tool because:

- ✅ **Pre-authenticated** in GitHub Codespaces - no token management needed
- ✅ **Works with private repos** - uses your Codespace's authentication
- ✅ **More reliable** - direct API access with proper error handling
- ✅ **Rich features** - can query, filter, and format issue data easily
- ✅ **Professional pattern** - real teams use `gh` CLI in CI/CD pipelines

## Learning Objectives

By using these prompts, you'll learn:

1. **Agentic Development**: AI autonomously following documented procedures
2. **Workflow Automation**: How AI integrates with Git, CI/CD, and issue tracking
3. **Meta-Development**: Using AI to manage AI-assisted development
4. **Professional Patterns**: Real-world automation approaches used by teams

## What You'll Create in Step 5-0

You'll use the `copilot-customization` chat mode to create these three prompt files:

### 1. `execute-step.prompt.md`

A slash command that reads GitHub Issues and executes instructions autonomously.

**Mode Strategy**: AUTO-SWITCHES to `tdd-developer` mode (execution = iterative development!)

### 2. `commit-and-push.prompt.md`

A slash command that analyzes changes and pushes to the feature branch you specify.

**Mode Strategy**: NO MODE specified - works in any mode (git is universal!)

### 3. `validate-step.prompt.md`

A slash command that validates completion against success criteria from the GitHub Issue.

**Mode Strategy**: AUTO-SWITCHES to `code-reviewer` mode (validation = review task!)

**🎓 Learning Objective**: By creating these three prompts, you'll see BOTH patterns in action:
- **Auto-switching prompts** (#1 `/execute-step` & #2 `/validate-step`): Intelligent context selection for specific workflows
- **Mode-agnostic prompts** (#3 `/commit-and-push`): Universal utilities that work anywhere

This teaches you when auto-switching adds value vs. when flexibility is better!

**Plus** you'll create:

- Project instructions (`.github/copilot-instructions.md`) with workflow utilities (gh CLI, Git)
- Memory system (`.github/memory/`) for capturing discoveries
- `tdd-developer` chat mode for test-driven development
- `code-reviewer` chat mode for code quality work

## Why You're Building This

1. **Understand the architecture**: You'll see how prompts work with modes and shared knowledge
2. **Learn by doing**: Creating these teaches you more than using pre-built ones
3. **Real-world skill**: Professional teams create custom prompts for their workflows
4. **Meta-development**: Using AI (`copilot-customization`) to build AI tools (prompts, modes, instructions)

## Try It After Step 5-0

Once you've created these files:

1. **Test auto-switching execution:**
   - Open Copilot Chat (in ANY mode)
   - Type `/execute-step`
   - Provide issue URL when prompted
   - **Watch it automatically switch to `tdd-developer` mode!** 🎉
   - Observe how it executes using TDD workflow patterns

2. **Test universal utility:**
   - Type `/commit-and-push`
   - Provide branch name when prompted
   - Notice it works in whatever mode you're in (no switching needed!)

3. **Test auto-switching validation:**
   - Type `/validate-step`
   - Provide step number when prompted
   - **Watch it automatically switch to `code-reviewer` mode!** 🎉
   - Observe how it validates using review patterns

4. **Observe the patterns:**
   - `/execute-step` → Auto-switches to `tdd-developer` (execution context)
   - `/validate-step` → Auto-switches to `code-reviewer` (review context)
   - `/commit-and-push` → Uses current mode (universal utility)

5. **Advanced**: Complete an entire step using only slash commands!
   ```text
   /execute-step → /validate-step → /commit-and-push
   ```

This is the **future of software development** - AI agents working alongside humans with shared knowledge, following documented procedures, maintaining quality standards, and **intelligently selecting the right context** for each task automatically! 🚀

---

**Example implementations** are available in the `examples/` directory if you want to see reference solutions after creating your own.
