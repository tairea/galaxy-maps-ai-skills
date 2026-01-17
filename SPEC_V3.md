# Galaxy Maps AI V3 - Separated Skills Specification

## Overview

Galaxy Maps AI V3 transforms the monolithic multi-agent system into a collection of specialized, independent skills. Each skill can be invoked standalone or orchestrated together for complete Galaxy Map creation.

### Key Improvements Over V2
- **Modular Architecture**: Each agent is now a separate skill with its own file
- **Specialized Tooling**: Each skill can have its own MCP servers and tools
- **Independent Evolution**: Skills can be versioned and updated independently
- **Reusability**: Skills can be invoked by other workflows
- **Clear Interfaces**: Standardized handoff protocols between skills
- **Parallel Development**: Multiple contributors can work on different skills

---

## Skill Overview

| Skill | Purpose | Model | Standalone |
|-------|---------|-------|------------|
| gm-ai-orchestrator | Coordinates workflow, manages git, transforms output | High | No |
| gm-ai-intent | Captures user intent through 6-canvas framework | Low | Yes |
| gm-ai-curriculum | Generates curriculum structure (Stars/Missions) | Ultra | Yes |
| gm-ai-curriculum-critiquer | Reviews and suggests structure improvements | Ultra | Yes |
| gm-ai-branching | Creates optional side-quest branches | Ultra | Yes |
| gm-ai-mission-builder | Generates HTML lesson content | High | Yes |
| gm-ai-mission-critiquer | Reviews and suggests content improvements | High | Yes |

---

## System Architecture

### Skill Relationships

```
                           +------------------------+
                           |   gm-ai-orchestrator   |
                           |       (Agent 0)        |
                           +------------------------+
                                      |
            +--------------------------+--------------------------+
            |            |             |             |            |
            v            v             v             v            v
    +-------------+ +-------------+ +-------------+ +-------------+ +-------------+
    | gm-ai-      | | gm-ai-      | | gm-ai-      | | gm-ai-      | | gm-ai-      |
    | intent      | | curriculum  | | branching   | | mission-    | | mission-    |
    | (Agent 1)   | | (Agent 2)   | | (Agent 4)   | | builder     | | critiquer   |
    +-------------+ +-------------+ +-------------+ | (Agent 5)   | | (Agent 6)   |
                          |                         +-------------+ +-------------+
                          v
                   +-------------+
                   | gm-ai-      |
                   | curriculum- |
                   | critiquer   |
                   | (Agent 3)   |
                   +-------------+
```

### Communication Model
- Hub-and-spoke: All communication flows through orchestrator
- Standardized handoff payloads with files[], action, context{}
- Skills never communicate directly with each other

---

## Skill Specifications

### gm-ai-orchestrator (Agent 0)

**Location**: `gm-ai-v3/gm-ai-orchestrator/SKILL.md`

**Purpose**: Main coordinator that manages the multi-agent workflow

**Responsibilities**:
1. Initialize and manage git repository
2. Orchestrate workflow in correct sequence
3. Handle user decisions between phases
4. Spawn sub-skills using Skill/Task tools
5. Commit and push files at checkpoints
6. Transform output to saveGalaxyMap() JSON

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Git MCP Server | Repository operations |
| GitHub MCP Server | Remote repo management |
| File System MCP | Read/write files |
| Firebase MCP | Database operations |

**Triggers**:
- "create a galaxy map"
- "build a curriculum"
- "/galaxy-map"

---

### gm-ai-intent (Agent 1)

**Location**: `gm-ai-v3/gm-ai-intent/SKILL.md`

**Purpose**: Capture curriculum intent through 6-canvas framework

**The 6 Areas**:
1. Audience (Who) - target learners
2. Topic/Domain (What) - subject matter
3. Unique Ideas - differentiators
4. Outcomes - success criteria
5. Timing - duration and pacing
6. Assessment - mastery measurement

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Audience Profiler MCP | Learner personas |
| Topic Taxonomy MCP | Subject standards |
| Duration Estimator | Timing calculations |

**Output**: `INTENT.md`

---

### gm-ai-curriculum (Agent 2)

**Location**: `gm-ai-v3/gm-ai-curriculum/SKILL.md`

**Purpose**: Generate curriculum structure with Stars and Missions

**Core Principles**:
- Stars: 1-2 days, ONE clear milestone, tangible outcome
- Missions: 15-60 minutes, atomic, scaffolded LOs

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Learning Standards MCP | Educational standards |
| Bloom's Taxonomy Tool | Cognitive levels |
| Prerequisite Graph | Dependency analysis |
| Parallel Model Invocation | Multi-LLM alternatives |

**Outputs**:
- `alternatives.md` (initial generation)
- `MAP_V{n}.md`

---

### gm-ai-curriculum-critiquer (Agent 3)

**Location**: `gm-ai-v3/gm-ai-curriculum-critiquer/SKILL.md`

**Purpose**: Quality control for curriculum structure

**Critique Framework**:
- Level 1: Overall (completeness, alignment, sequencing, engagement)
- Level 2: Star (size, focus, outcome, position)
- Level 3: Mission (atomicity, scaffolding, clarity, necessity)

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Pedagogical Analyzer | Best practices check |
| Scope Validator | Coverage verification |
| Transition Analyzer | Gap detection |

**Output**: `MAP_V{n}_SUGGESTIONS.md`

---

### gm-ai-branching (Agent 4)

**Location**: `gm-ai-v3/gm-ai-branching/SKILL.md`

**Purpose**: Create optional side-quest branches

**Branch Types**:
1. Deep Dive - deeper on touched topics
2. Adjacent Topic - related distinct areas
3. Tool/Technique - alternative approaches
4. Real-World Application - industry context
5. History/Context - background information

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Topic Explorer MCP | Knowledge graph traversal |
| Wikipedia/Wikidata API | Research |
| Industry Trends MCP | Real-world applications |

**Output**: `branches/STAR_{n}_BRANCH_{m}.md`

---

### gm-ai-mission-builder (Agent 5)

**Location**: `gm-ai-v3/gm-ai-mission-builder/SKILL.md`

**Purpose**: Generate rich HTML lesson content

**Content Structure**:
1. Hook - engaging opening
2. Objective - clear statement
3. Content - core teaching
4. Practice - hands-on application
5. Check - understanding verification
6. Bridge - next mission connection

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| YouTube MCP | Educational videos |
| D3/Visualization Generator | Diagrams |
| Code Playground Embedder | Runnable examples |
| Quiz Builder | Assessment questions |
| Code Validator | Syntax checking |

**Outputs**:
- `MISSION_{n}_{m}.md`
- `MISSION_{n}_{m}.html`

---

### gm-ai-mission-critiquer (Agent 6)

**Location**: `gm-ai-v3/gm-ai-mission-critiquer/SKILL.md`

**Purpose**: Quality control for mission content

**Critique Dimensions**:
1. LO Alignment
2. Audience Appropriateness
3. Content Quality
4. Code Accuracy
5. Engagement
6. Scaffolding
7. Completeness

**Recommended Tools**:
| Tool | Purpose |
|------|---------|
| Readability Analyzer | Language level |
| Code Linter | Syntax validation |
| HTML Validator | Accessibility |
| LO Coverage Checker | Objective alignment |

**Output**: `MISSION_{n}_{m}_SUGGESTIONS.md`

---

## Complete Workflow

```
PHASE 1: INTENT CAPTURE
------------------------
User -> gm-ai-orchestrator -> gm-ai-intent
                                   |
                                   v
                            [Interactive session]
                            [6 areas of intent]
                                   |
                                   v
                            gm-ai-intent -> gm-ai-orchestrator
                            files: [INTENT.md]
                                   |
                                   v
                            git init, commit INTENT.md


PHASE 2: CURRICULUM GENERATION
------------------------------
gm-ai-orchestrator -> gm-ai-curriculum
                           |
                           v
                     [Generate 3 alternatives]
                           |
                           v
                     User selects preferred structure
                           |
                           v
                     commit MAP_V1.md


PHASE 3: STRUCTURE CRITIQUE (Optional, Repeatable)
--------------------------------------------------
gm-ai-orchestrator -> gm-ai-curriculum-critiquer
                           |
                           v
                     [Interactive critique session]
                     [User approves/declines]
                           |
                           v
                     gm-ai-curriculum regenerates with suggestions
                           |
                           v
                     commit MAP_V{n+1}.md


PHASE 4: BRANCHING SIDE-QUESTS
------------------------------
gm-ai-orchestrator -> gm-ai-branching
                           |
                           v
                     [Generate branch mini-curricula]
                           |
                           v
                     commit branches/*


PHASE 5: MISSION BUILDING (Parallel)
------------------------------------
gm-ai-orchestrator -> gm-ai-mission-builder (x N parallel)
                           |
                           +-> Star 1 missions
                           +-> Star 2 missions
                           +-> Star N missions
                           |
                           v (parallel completion)
                     commit missions/*


PHASE 6: MISSION CRITIQUE (Optional, Parallel)
----------------------------------------------
gm-ai-orchestrator -> gm-ai-mission-critiquer (x N parallel)
                           |
                           v
                     [Interactive sessions]
                     [Regenerate as needed]


PHASE 7: FINALIZE & PUBLISH
---------------------------
gm-ai-orchestrator:
  1. Transform repo files -> saveGalaxyMap JSON
  2. Call saveGalaxyMap() cloud function
  3. Commit GALAXY_MAP.json (archive)
  4. Push to remote
  5. Return courseId to user
```

---

## File Schemas

### INTENT.md
```yaml
---
intentId: <uuid>
mapTitle: <string>
mapDescription: <string>
createdAt: <ISO8601>
createdBy: <string>
status: draft | complete
---

# 1. Audience (Who)
# 2. Topic/Domain (What)
# 3. Unique Ideas
# 4. Outcomes
# 5. Timing
# 6. Assessment/Mastery
```

### MAP_V{n}.md
```yaml
---
mapId: <uuid>
intentId: <reference>
version: <n>
name: <slug>
title: <string>
description: <string>
tags: [<tags>]
estimatedDuration: <time>
totalStars: <count>
totalMissions: <count>
createdAt: <ISO8601>
status: draft | review | approved
---

- Star 1 - <Title> - <Milestone Objective>
  - Mission 1.1 - <Title> - <Learning Objective>
  - Mission 1.2 - <Title> - <Learning Objective>
```

### Handoff Protocol
```json
{
  "from": "gm-ai-{skill}",
  "to": "gm-ai-orchestrator",
  "status": "complete | needs-input | error",
  "files": ["file1.md", "file2.html"],
  "stats": { ... },
  "message": "Human-readable status message"
}
```

---

## Repository Structure

```
~/galaxy-maps/{map-slug}/
├── .git/
├── INTENT.md
├── MAP_V1.md
├── MAP_V1_SUGGESTIONS.md
├── MAP_V2.md
├── branches/
│   ├── STAR_1_BRANCH_1.md
│   ├── STAR_2_BRANCH_1.md
│   └── STAR_2_BRANCH_2.md
├── missions/
│   ├── star_1/
│   │   ├── MISSION_1_1.md
│   │   ├── MISSION_1_1.html
│   │   └── ...
│   └── star_2/
│       └── ...
└── GALAXY_MAP.json  (final archive)
```

---

## Skill Directory Structure

```
gm-ai-v3/
├── gm-ai-orchestrator/
│   ├── SKILL.md
│   ├── schemas/
│   │   ├── intent.schema.json
│   │   ├── map.schema.json
│   │   └── handoff.schema.json
│   └── templates/
│       └── workflow-state.yaml
├── gm-ai-intent/
│   ├── SKILL.md
│   ├── prompts/
│   │   └── elicitation-hints.md
│   └── templates/
│       └── INTENT.template.md
├── gm-ai-curriculum/
│   ├── SKILL.md
│   ├── rules/
│   │   └── design-principles.md
│   └── templates/
│       └── MAP.template.md
├── gm-ai-curriculum-critiquer/
│   ├── SKILL.md
│   └── frameworks/
│       └── critique-checklist.md
├── gm-ai-branching/
│   ├── SKILL.md
│   └── templates/
│       └── BRANCH.template.md
├── gm-ai-mission-builder/
│   ├── SKILL.md
│   ├── templates/
│   │   ├── mission.template.md
│   │   └── mission.template.html
│   └── styles/
│       └── mission.css
└── gm-ai-mission-critiquer/
    ├── SKILL.md
    └── frameworks/
        └── quality-rubric.md
```

---

## Benefits of Separation

1. **Independent Testing**: Each skill can be unit tested in isolation
2. **Version Control**: Skills can evolve at different rates
3. **Specialization**: Tools can be tailored precisely to each skill's needs
4. **Reusability**: Skills can be used by other educational workflows
5. **Parallel Development**: Multiple contributors can work simultaneously
6. **Cost Optimization**: Use appropriate model tiers per skill
7. **Debugging**: Issues isolated to specific skills
8. **Documentation**: Each skill has focused, clear documentation

---

## Implementation Priority

| Priority | Skill | Rationale |
|----------|-------|-----------|
| 1 | gm-ai-orchestrator | Core coordination, must exist first |
| 2 | gm-ai-intent | Entry point, user-facing |
| 3 | gm-ai-curriculum | Critical path, generates structure |
| 4 | gm-ai-mission-builder | Content generation, high value |
| 5 | gm-ai-curriculum-critiquer | Quality improvement |
| 6 | gm-ai-mission-critiquer | Content quality |
| 7 | gm-ai-branching | Enhancement, not critical path |

---

## Future Considerations

### gm-ai-realtime-tutor (Agent 7 - Post-MVP)
- Assists learners during mission completion
- Provides hints and explanations
- Assesses mastery before progression

### Branch Content Generation
- Currently branches are structure-only
- Future: gm-ai-mission-builder generates HTML for branch missions

### Shared Infrastructure
- gm-ai-schema-validator: Validates all .md file formats
- gm-ai-storage: Unified file I/O
- gm-ai-handoff: Standardized inter-skill communication

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 3.0 | 2025-01-17 | Separated skills architecture |
| 2.0 | 2025-01-15 | Multi-agent monolithic system |
| 1.0 | 2024-XX-XX | Original single-prompt system |
