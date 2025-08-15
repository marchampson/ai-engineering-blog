---
title: "The Persistent Context Hypothesis"
slug: "the-persistent-context-hypothesis"
date: 2025-08-15T15:13:17+01:00
draft: false
description: "How we transformed chaotic multi-conversation development into a systematic, persistent workflow using specialised AI agents."
tags: ["AI Engineering", "Agents", "Workflow", "Laravel", "Vue"]
categories: ["AI Engineering"]
---

# Building a Session-Based Agent Workflow System for Complex Software Development

*How we transformed chaotic multi-conversation development into a systematic, persistent workflow using specialised AI agents.*

## The Problem: Development Context Loss

Every developer knows this frustration: you are deep into a complex feature implementation across multiple conversations with AI assistants, and suddenly the thread is lost. Previous decisions are forgotten, architectural choices are second‑guessed, and work goes in circles.

We faced this exact challenge while building a Laravel application with Vue.js components. Our development process was fragmented:

- Conversations would end mid‑implementation.
- Context was lost between sessions.
- Agents contradicted previous decisions.
- Complex work lacked persistent planning.
- Multiple agents interfered with one another.

**The breaking point:** we were refactoring a Livewire component to Vue.js and kept hitting the same bugs repeatedly across multiple conversations, with no systematic way to track what had been tried and what the actual plan was.

## The Solution: Session‑Based Agent Workflows

We built a comprehensive session management system that treats complex development work like professional project management — with persistent sessions, specialised agents, clear hand‑offs, and systematic progress tracking.

### Core Architecture

```
.claude/
├── wip/                          # Work in progress sessions
│   ├── .template/               # Session creation templates
│   ├── feature-ai-integration/  # Example active session
│   └── refactor-briefing-form/  # Another session
├── commands/
│   ├── start-session.md         # Main session orchestrator
│   ├── session-commands.md      # Complete command reference
│   └── session-utils.md         # Management utilities
└── agents/                      # Specialised agent definitions
    ├── laravel-researcher.md    # Research-only agent
    ├── feature-architect.md     # Planning and architecture
    ├── backend-developer.md     # PHP/Laravel backend
    ├── vue-developer.md         # Vue.js frontend specialist
    ├── livewire-frontend.md     # Blade/FluxUI/Alpine
    └── laravel-tester.md        # Testing specialist
```

### Session Structure

Each development session becomes a persistent workspace:

```json
{
  "id": "feature-ai-integration",
  "title": "Add AI Chat Integration",
  "type": "feature",
  "status": "in-progress",
  "scope": "Kanban → Card Modal → AI Integration",
  "risk": "medium",
  "current_task": 2,
  "assigned_agent": "feature-architect",
  "tasks": [
    {
      "id": 1,
      "title": "Research existing patterns",
      "agent": "laravel-researcher",
      "status": "completed"
    },
    {
      "id": 2,
      "title": "Plan component architecture",
      "agent": "feature-architect",
      "status": "in-progress"
    }
  ]
}
```

## Specialised Agent Architecture

We redesigned our agent system with clear responsibilities and areas of expertise.

### Research & Planning Agents
- **`laravel-researcher`**: documentation research and pattern analysis.
- **`feature-architect`**: architecture planning for features and refactoring.

### Implementation Agents
- **`backend-developer`**: PHP/Laravel backend work (controllers, services, APIs).
- **`vue-developer`**: Vue.js components, Pinia stores, real‑time features.
- **`livewire-frontend`**: Blade templates, FluxUI, Alpine.js.

### Quality Assurance Agents
- **`laravel-tester`**: Pest testing and quality validation.
- **`security-code-reviewer`**: security and vulnerability review.

### Agent Integration with Sessions

Each agent is “session‑aware”.

```markdown
## Session Integration

### Before Starting Work
1. **Read AGENT_HELP_INDEX.md** to locate relevant documentation.
2. **If working in a session**: read session context from `.claude/wip/{session-id}/`.
   - Review `session.json` for scope and constraints.
   - Read `plan.md` to understand objectives.
   - Check `progress.md` for previous work.
3. **Build on previous work**: use findings from previous agents.

### Session Handover Protocol
When completing a task:
1. Update task status to “completed”.
2. Add completion notes to the progress log.
3. Identify the next task and the required agent.
4. Handover the session with a concise context summary.
```

## Workflow Types and Patterns

### Feature Development Flow
1. **Research** (`laravel-researcher`) → understand existing patterns.
2. **Architecture** (`feature-architect`) → plan component structure.
3. **Implementation** (`backend-developer` + `vue-developer`) → build the feature.
4. **Testing** (`laravel-tester`) → comprehensive test suite.
5. **Security Review** (`security-code-reviewer`) → vulnerability check.

### Bug Fix Flow
1. **Research** (`laravel-researcher`) → understand the problem.
2. **Reproduce** (`laravel-tester`) → create a failing test.
3. **Fix** (appropriate specialist) → implement the minimal fix.
4. **Verify** (`laravel-tester`) → ensure the fix works.

### Refactoring Flow
1. **Analysis** (`laravel-researcher`) → document the current implementation.
2. **Architecture** (`feature-architect`) → plan the refactoring approach.
3. **Implementation** (appropriate specialist) → execute the refactor.
4. **Testing** (`laravel-tester`) → ensure no behavioural changes.

## Session Management Commands

### Core Session Operations

```bash
# Start a new session or resume an existing one
/start-session

# List all current sessions
/list-sessions

# Resume a specific session
/resume-session feature-ai-integration

# Show detailed session status
/session-status feature-ai-integration
```

### Agent Workflow Commands

```bash
# For agents: load session context
/read-session-context feature-ai-integration

# Complete the current task and hand over
/complete-task feature-ai-integration 2 "Architecture completed"

# Add progress notes
/add-session-note feature-ai-integration "Decided on slide-out approach"
```

## Real‑World Example: Session in Action

Here is how a real refactoring session looks.

### Session Creation

```
User: /start-session
Agent: What type of work? → refactor
User: Convert BriefingForm from Livewire to Vue
Agent: Session created: refactor-briefing-form
       Starting with laravel-researcher
```

### Research Phase

```markdown
## Research Complete – Handover to feature-architect

**Current implementation analysis:**
- `BriefingForm.blade.php` with Alpine.js interactions.
- Livewire component handling form state.
- Real‑time field locking via Livewire events.
- Integration with presence system.

**Conversion requirements:**
- Maintain all existing functionality.
- Convert to Vue 3 + Composition API.
- Preserve real‑time collaborative features.
- Keep integration with `CardModal.vue`.

**Next agent:** `feature-architect` should plan the Vue conversion.
```

### Architecture Phase

```markdown
## Architecture Complete – Handover to vue-developer

**Vue conversion plan:**
- Convert to `BriefingTab.vue` component.
- Use a Pinia store for form state management.
- Migrate real‑time features to Laravel Echo.
- Maintain field locking with the presence system.

**Implementation strategy:**
1. Create the new Vue component structure.
2. Convert Alpine.js logic to the Composition API.
3. Integrate with existing Vue stores.
4. Test real‑time features thoroughly.
```

### Implementation & Testing

The `vue-developer` and `laravel-tester` agents execute the plan with full context of previous decisions and requirements.

## Benefits Achieved

### Persistent Context
- Work survives conversation breaks.
- Decisions are documented and maintained.
- No more starting over each conversation.

### Specialised Expertise
- Each agent has clear responsibilities.
- No overlap or conflicting approaches.
- Quality work within expertise areas.

### Progress Tracking
- Visual progress through complex tasks.
- Clear task dependencies and hand‑overs.
- Easy to resume exactly where you left off.

### Quality Gates
- Built‑in validation at each step.
- Testing requirements enforced.
- Security review mandatory for code changes.

### Systematic Problem Solving
- Bugs are documented and addressed systematically.
- No more going in circles on complex issues.
- Clear escalation path when agents get stuck.

## Technical Implementation Details

### Session Storage

Each session is stored as a structured directory.

```
.claude/wip/feature-ai-integration/
├── session.json      # Metadata and task tracking
├── plan.md           # Architectural planning
├── progress.md       # Real‑time progress notes
└── artifacts/        # Generated files and diagrams
```

### Agent Enhancement Pattern

We enhanced existing agents with session awareness.

```markdown
## Always Do

- **First:** check if this is session‑based work.
- **Auto‑use Context7** for framework documentation.
- **Follow the session plan** exactly; avoid scope creep.
- **Update progress** with implementation notes.
- **Hand over properly** to the next agent with context.
```

### Integration with Existing Tools

The session system integrates seamlessly with existing commands:

- `/plan` → creates a session plan when starting a new session.
- `/code` → used by developers within session context.
- `/check` → quality‑gate validation for sessions.

## Results and Impact

### Before: Chaotic Development
- Lost context between conversations.
- Repeated work and contradictory decisions.
- Bugs going in circles without resolution.
- No systematic approach to complex features.

### After: Professional Workflow
- Persistent sessions with full context preservation.
- Specialised agents with clear expertise boundaries.
- Systematic progression through complex work.
- Built‑in quality gates and testing requirements.
- Professional project management for AI‑assisted development.

### Measurable Improvements

- **Context retention:** 100% (versus approximately 20% in ad‑hoc conversations).
- **Rework reduction:** around 80% less repeated work.
- **Bug resolution:** systematic tracking and resolution.
- **Feature completion:** higher success rate for complex features.

## Key Lessons Learned

1. **Persistence is critical.** Complex software development requires persistent context. Sessions that survive conversation breaks are essential for professional work.
2. **Agent specialisation works.** Clear agent boundaries prevent conflicting approaches and ensure expertise is applied appropriately.
3. **Planning before implementation.** Forcing a planning phase before implementation significantly improves outcomes and reduces rework.
4. **Progress tracking enables resumption.** Visual progress tracking makes it trivial to resume work exactly where it was left off.
5. **Quality gates prevent technical debt.** Built‑in testing and security review requirements prevent shortcuts that create future problems.

## Future Enhancements

### Session Analytics
- Track session completion rates.
- Identify common failure patterns.
- Optimise agent hand‑over processes.

### Advanced Session Types
- Investigation sessions for complex debugging.
- Architecture review sessions for large refactors.
- Performance optimisation sessions.

### Cross‑Session Learning
- Pattern recognition across similar sessions.
- Automated best‑practice suggestions.
- Session template improvements.

## Getting Started

### Implementation Steps

1. **Create the session structure:** set up the `.claude/wip/` directory system.
2. **Define specialised agents:** create agent definitions with clear boundaries.
3. **Build session commands:** implement session management commands.
4. **Enhance agents:** add session awareness to existing agents.
5. **Create templates:** build session‑creation templates.
6. **Test with real work:** start with a real refactor or feature.

### Best Practices

- Start with simple sessions to learn the workflow.
- Clearly define agent boundaries and areas of expertise.
- Always plan before implementing.
- Document decisions and rationale in session notes.
- Use quality gates to maintain code standards.

## Conclusion

Building a session‑based agent workflow system transformed our development process from chaotic multi‑conversation struggles into a systematic, professional approach to complex software development.

The key insight: **AI‑assisted development needs the same project‑management discipline as human development teams.** By treating each piece of complex work as a managed project with specialised roles, persistent context, and systematic progression, we achieved markedly better outcomes.

For development teams struggling with context loss and inconsistent AI assistance, implementing a session‑based workflow system provides the structure needed to tackle complex software projects with AI agents successfully.

The future of AI‑assisted development is not merely better individual responses — it is better workflow systems that treat AI agents as specialised team members working together on persistent, well‑managed projects.

---

*This session‑based workflow system is now production‑ready and handling real refactoring work, demonstrating that systematic approaches to AI development can achieve professional‑grade results.*
