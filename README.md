# Galaxy Maps AI Skills (V3)

A modular, multi-agent system for creating curriculum-based learning journeys (Galaxy Maps). This version separates each agent into its own specialized skill for better maintainability, extensibility, and tooling.

## Skills Overview

| Skill | Description | Agent |
|-------|-------------|-------|
| [gm-ai-orchestrator](./gm-ai-orchestrator) | Main coordinator - manages workflow, git, and database | Agent 0 |
| [gm-ai-intent](./gm-ai-intent) | Captures curriculum intent via 6-canvas framework | Agent 1 |
| [gm-ai-curriculum](./gm-ai-curriculum) | Generates curriculum structure (Stars/Missions) | Agent 2 |
| [gm-ai-curriculum-critiquer](./gm-ai-curriculum-critiquer) | Reviews and improves curriculum structure | Agent 3 |
| [gm-ai-branching](./gm-ai-branching) | Creates optional side-quest branches | Agent 4 |
| [gm-ai-mission-builder](./gm-ai-mission-builder) | Generates rich HTML lesson content | Agent 5 |
| [gm-ai-mission-critiquer](./gm-ai-mission-critiquer) | Reviews and improves mission content | Agent 6 |

## Architecture

```
                        gm-ai-orchestrator
                              |
     +------------+-----------+-----------+-----------+
     |            |           |           |           |
gm-ai-intent  gm-ai-       gm-ai-     gm-ai-      gm-ai-
              curriculum   branching  mission-    mission-
                  |                   builder     critiquer
                  v
           gm-ai-curriculum-
           critiquer
```

## Quick Start

1. **Invoke the orchestrator** to start Galaxy Map creation:
   ```
   /galaxy-map
   ```
   or trigger with: "create a galaxy map", "build a curriculum"

2. **The orchestrator guides you through**:
   - Intent capture (6-canvas framework)
   - Curriculum structure generation (3 alternatives)
   - Optional critique and refinement cycles
   - Side-quest branch generation
   - Mission content creation
   - Publication to Galaxy Maps database

## Using Individual Skills

Each skill can be invoked standalone:

```javascript
// Capture intent only
Skill({ skill: "gm-ai-intent" })

// Generate curriculum from existing intent
Skill({
  skill: "gm-ai-curriculum",
  args: JSON.stringify({ files: ["INTENT.md"] })
})

// Build missions for a specific star
Skill({
  skill: "gm-ai-mission-builder",
  args: JSON.stringify({ star: { index: 1, missions: [...] } })
})
```

## Key Concepts

### Stars (Milestones)
- 1-2 days of learning
- ONE clear concept/skill
- Tangible outcome at completion

### Missions (Lessons)
- 15-60 minutes each
- Atomic, focused learning objective
- Scaffolds from previous mission

### Branches (Side-Quests)
- Optional enrichment paths
- 30-90 minutes total
- Deep dives, tools, real-world applications

## File Outputs

```
~/galaxy-maps/{map-slug}/
├── INTENT.md              # User intent (6 areas)
├── MAP_V1.md              # Initial curriculum structure
├── MAP_V1_SUGGESTIONS.md  # Critique feedback
├── MAP_V2.md              # Refined structure
├── branches/              # Side-quest branches
├── missions/              # HTML lesson content
│   ├── star_1/
│   │   ├── MISSION_1_1.md
│   │   └── MISSION_1_1.html
│   └── ...
└── GALAXY_MAP.json        # Final database payload
```

## Recommended Tools per Skill

See individual skill READMEs for detailed tool recommendations:

- **Orchestrator**: Git MCP, GitHub MCP, Firebase MCP
- **Intent**: Audience Profiler, Topic Taxonomy
- **Curriculum**: Learning Standards, Bloom's Taxonomy, Prerequisite Graph
- **Branching**: Topic Explorer, Wikipedia API
- **Mission Builder**: YouTube MCP, Code Playground, Quiz Builder

## Documentation

- [SPEC_V3.md](./SPEC_V3.md) - Complete specification
- Each skill's `SKILL.md` contains detailed implementation guidance

## Contributing

Each skill can be developed independently. When modifying:

1. Ensure handoff protocol compatibility
2. Update SKILL.md with any interface changes
3. Test integration with orchestrator

## Version History

- **V3** (2025-01-17): Separated skills architecture
- **V2** (2025-01-15): Multi-agent monolithic system
- **V1** (2024): Single-prompt curriculum generation

## License

MIT
