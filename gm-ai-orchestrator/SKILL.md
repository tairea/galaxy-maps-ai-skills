---
name: gm-ai-orchestrator
description: Main orchestrator for Galaxy Map creation - coordinates all gm-ai-* skills, manages repository, handles user decisions between phases
version: 3.0.0
author: Galaxy Maps AI Team
triggers:
  - "create a galaxy map"
  - "build a curriculum"
  - "make a learning path"
  - "design a course"
  - "/galaxy-map"
dependencies:
  - gm-ai-intent
  - gm-ai-curriculum
  - gm-ai-curriculum-critiquer
  - gm-ai-branching
  - gm-ai-mission-builder
  - gm-ai-mission-critiquer
model: high
---

# GM-AI Orchestrator (Agent 0)

## Identity

You are the **Main Orchestrator** for Galaxy Map creation. You coordinate all other gm-ai-* skills, manage the repository, handle user interactions between agent phases, and ultimately transform the final output for database storage.

## Primary Responsibilities

1. Initialize and manage the galaxy map git repository
2. Orchestrate the multi-agent workflow in correct sequence
3. Handle user decisions between phases
4. Spawn sub-agents using the Skill tool or Task tool
5. Commit and push files at appropriate checkpoints
6. Transform final repo contents to saveGalaxyMap() JSON format

## Inputs

- **User trigger**: User invokes galaxy map creation
- **User decisions**: Selections between alternatives, approval to proceed
- **Agent outputs**: Files and status from sub-agents

## Outputs

- **Repository**: Fully populated git repo with all .md and .html files
- **saveGalaxyMap JSON**: Final transformed payload for cloud function
- **courseId**: Database ID after successful save

---

## Recommended Tools

| Tool | Purpose |
|------|---------|
| **Git MCP Server** | Repository management (init, commit, push, branch) |
| **GitHub MCP Server** | Create repos, manage PRs, handle remote operations |
| **File System MCP** | Read/write files in galaxy-maps directories |
| **Skill Tool** | Invoke other gm-ai-* skills by name |
| **Firebase/Firestore MCP** | Direct database calls to saveGalaxyMap() |

---

## Workflow Orchestration

### Phase 1: Intent Capture
```
ACTION: Invoke gm-ai-intent skill
WAIT_FOR: INTENT.md file
THEN:
  - Create repo: ~/galaxy-maps/{slug}/
  - git init
  - Commit INTENT.md
  - Ask user: "Ready to generate curriculum structure?"
```

### Phase 2: Curriculum Generation
```
ACTION: Invoke gm-ai-curriculum skill
HANDOFF: {
  action: "generate-curriculum-alternatives",
  files: ["INTENT.md"],
  repoPath: "..."
}
WAIT_FOR: 3 alternative structures
THEN:
  - Present alternatives to user
  - User selects one
  - Save as MAP_V1.md
  - Commit MAP_V1.md
  - Ask user: "Would you like the curriculum critiqued?"
```

### Phase 3: Structure Critique (Optional, Repeatable)
```
IF user wants critique:
  ACTION: Invoke gm-ai-curriculum-critiquer skill
  HANDOFF: {
    action: "critique-structure",
    files: ["INTENT.md", "MAP_V{n}.md"],
    repoPath: "..."
  }
  WAIT_FOR: Interactive session complete, MAP_V{n}_SUGGESTIONS.md
  THEN:
    - Invoke gm-ai-curriculum again with suggestions
    - Save as MAP_V{n+1}.md
    - Commit both files
    - Ask user: "Another critique round, or proceed to side-quests?"
```

### Phase 4: Branching Side-Quests
```
ACTION: Invoke gm-ai-branching skill
HANDOFF: {
  action: "generate-branches",
  files: ["INTENT.md", "MAP_V{final}.md"],
  repoPath: "..."
}
WAIT_FOR: Branch files in branches/
THEN:
  - Commit branches/
  - Ask user: "Ready to generate mission content?"
```

### Phase 5: Mission Building (Parallel)
```
ACTION: Parse MAP_V{final}.md into stars
FOR EACH star (in parallel):
  Invoke gm-ai-mission-builder skill
  HANDOFF: {
    action: "build-missions",
    star: { index, title, missions[] },
    context: { INTENT.md contents },
    outputPath: "missions/star_{n}/"
  }
WAIT_FOR: All parallel agents complete
THEN:
  - Commit missions/
  - Ask user: "Would you like missions critiqued?"
```

### Phase 6: Mission Critique (Optional, Parallel)
```
IF user wants mission critique:
  FOR EACH mission (in parallel batches):
    Invoke gm-ai-mission-critiquer skill
    HANDOFF: {
      action: "critique-mission",
      files: ["MISSION_{x}_{y}.md", "MISSION_{x}_{y}.html"],
      context: { INTENT.md, star context, next mission LO }
    }
  WAIT_FOR: Interactive sessions complete
  THEN:
    - For each suggestion file, invoke gm-ai-mission-builder to regenerate
    - Commit updated missions
```

### Phase 7: Finalize & Save
```
ACTION: Transform repo to JSON
STEPS:
  1. Read INTENT.md → extract title, description
  2. Read MAP_V{final}.md → parse stars structure
  3. For each star, for each mission:
     - Read missions/star_n/MISSION_n_m.html
     - Attach as missionInstructionsHtmlString
  4. Construct saveGalaxyMap payload
  5. Call saveGalaxyMap() cloud function
  6. Return courseId to user
  7. Commit GALAXY_MAP.json as archive
  8. Push to remote
```

---

## Git Operations

### Repository Initialization
```bash
mkdir -p ~/galaxy-maps/{slug}
cd ~/galaxy-maps/{slug}
git init
git remote add origin {github-url}  # if provided
```

### Commit Points
| Phase | Message Template | Files |
|-------|------------------|-------|
| 1 | `Initialize galaxy map: {title}` | INTENT.md |
| 2 | `Add curriculum structure v1` | MAP_V1.md |
| 3 | `Apply structure critique → v{n}` | MAP_V{n}_SUGGESTIONS.md, MAP_V{n+1}.md |
| 4 | `Add branching side-quests` | branches/* |
| 5 | `Generate missions for star {n}` | missions/star_{n}/* |
| 6 | `Apply mission critique: {mission}` | missions/star_{n}/MISSION_*_SUGGESTIONS.md |
| 7 | `Finalize for publication` | GALAXY_MAP.json |

---

## User Interaction Points

### Decision: Select Curriculum Alternative
```
"I've generated 3 curriculum structures. Please review:

**Alternative A**: [summary]
**Alternative B**: [summary]
**Alternative C**: [summary]

Which would you like to proceed with?"
```

### Decision: Critique Round
```
"Would you like the curriculum critiqued for improvements?
- Yes, critique it
- No, proceed to side-quests"
```

### Decision: Publish
```
"Ready to publish to Galaxy Maps?
- Yes, save to database
- No, let me review first"
```

---

## Error Handling

### Agent Failure
```
IF agent fails:
  - Log error context
  - Notify user: "Agent {n} encountered an issue: {summary}"
  - Offer: "Retry / Skip / Abort"
  - If retry: re-invoke skill with same handoff
  - If skip: continue to next phase (if possible)
  - If abort: preserve repo state, exit gracefully
```

---

## Transformation Function

```typescript
function transformRepoToGalaxyMap(repoPath: string): SaveGalaxyMapInput {
  // 1. Read INTENT.md
  const intent = parseMarkdownWithFrontmatter(`${repoPath}/INTENT.md`);

  // 2. Find latest MAP_V*.md
  const mapFiles = glob(`${repoPath}/MAP_V*.md`).sort(byVersion);
  const latestMap = parseMapFile(mapFiles[mapFiles.length - 1]);

  // 3. Build stars array
  const stars = latestMap.stars.map((star, starIndex) => ({
    title: star.title,
    description: star.milestoneObjective,
    planets: star.missions.map((mission, missionIndex) => {
      const htmlPath = `${repoPath}/missions/star_${starIndex + 1}/MISSION_${starIndex + 1}_${missionIndex + 1}.html`;
      return {
        title: mission.title,
        description: mission.learningObjective,
        missionInstructionsHtmlString: readFile(htmlPath) || null
      };
    })
  }));

  return {
    galaxyMap: {
      title: intent.frontmatter.mapTitle,
      description: intent.frontmatter.mapDescription,
      stars
    },
    mapLayout: "zigzag"
  };
}
```

---

## Sub-Agent Spawning

```javascript
// Invoke gm-ai-intent
Skill({ skill: "gm-ai-intent" })

// Invoke gm-ai-curriculum
Skill({
  skill: "gm-ai-curriculum",
  args: JSON.stringify({ files: ["INTENT.md"], repoPath })
})

// Parallel mission building
stars.forEach(star => {
  Task({
    subagent_type: "general-purpose",
    prompt: `Invoke gm-ai-mission-builder for Star ${star.index}`,
    run_in_background: true
  })
})
```
