# TourGuide JSON Schema Reference

**Technical specification for TourGuide JSON files**

---

## Schema Version

**Current Version:** 0.1.0  
**Format:** JSON (strict)  
**Encoding:** UTF-8

---

## Root Object

```typescript
{
  guide_name: string;           // REQUIRED
  version: string;              // REQUIRED (semver)
  description: string;          // REQUIRED
  ai_instructions: AiInstructions;  // REQUIRED
  navigation: Navigation;       // REQUIRED
  ignorance_zones?: IgnoranceZone[];  // OPTIONAL
  completion_criteria?: Record<string, string>;  // OPTIONAL
}
```

---

## AiInstructions

Defines the AI's role and behavior rules.

```typescript
{
  role: string;       // REQUIRED - AI's role in this exploration
  task: string;       // REQUIRED - What the AI should accomplish
  rules: string[];    // REQUIRED - Array of behavioral rules
}
```

**Example:**
```json
{
  "role": "You are exploring this codebase systematically",
  "task": "Build complete understanding through phased navigation",
  "rules": [
    "Read files in exact order specified",
    "Complete checkpoints before moving on",
    "Do NOT skip ahead or summarize until instructed"
  ]
}
```

**Best Practices:**
- `role`: Set expectations (e.g., "exploring", "learning", "auditing")
- `task`: Be specific about the goal
- `rules`: Include 3-7 clear, enforceable rules

---

## Navigation

Defines the exploration path through the project.

```typescript
{
  start_here: string;  // REQUIRED - Phase ID to start with
  phases: Phase[];     // REQUIRED - Array of navigation phases
}
```

---

## Phase

A logical group of sequential steps.

```typescript
{
  id: string;          // REQUIRED - Unique identifier
  name: string;        // REQUIRED - Human-readable name
  description: string; // REQUIRED - What this phase covers
  sequence: Step[];    // REQUIRED - Ordered steps in this phase
}
```

**Example:**
```json
{
  "id": "phase_1",
  "name": "Project Context",
  "description": "Understand the project's purpose and scope",
  "sequence": [
    { "step": 1, "action": "read_file", "path": "README.md" }
  ]
}
```

**Naming Conventions:**
- `id`: Lowercase, underscore-separated (e.g., `phase_1`, `auth_exploration`)
- `name`: Title case, descriptive (e.g., "Project Context", "API Layer")
- `description`: Sentence case, explains purpose

---

## Step

A single action in the navigation sequence.

### Common Fields

```typescript
{
  step: number;        // REQUIRED - Sequential step number
  action: ActionType;  // REQUIRED - Type of action
  notes?: string;      // OPTIONAL - Additional context
}
```

### ActionType: "read_file"

Read a specific file.

```typescript
{
  step: number;
  action: "read_file";
  path: string;        // REQUIRED - Relative path from project root
  focus?: string[];    // OPTIONAL - What to pay attention to
  notes?: string;
}
```

**Example:**
```json
{
  "step": 1,
  "action": "read_file",
  "path": "src/main.py",
  "focus": [
    "Entry point function",
    "Initialization sequence",
    "Main execution loop"
  ],
  "notes": "This is the application entry point"
}
```

### ActionType: "follow_guide"

Navigate to a sub-guide (another TourGuide JSON).

```typescript
{
  step: number;
  action: "follow_guide";
  path: string;        // REQUIRED - Path to sub-guide JSON
  notes?: string;
}
```

**Example:**
```json
{
  "step": 2,
  "action": "follow_guide",
  "path": "src/api/tourguide.json",
  "notes": "Deep dive into API layer"
}
```

### ActionType: "checkpoint"

Verify understanding before proceeding.

```typescript
{
  step: number;
  action: "checkpoint";
  verify: string[];    // REQUIRED - Questions to answer
  next: string;        // REQUIRED - Instructions for proceeding
}
```

**Example:**
```json
{
  "step": 3,
  "action": "checkpoint",
  "verify": [
    "What is the project's purpose?",
    "What are the main components?",
    "What technologies are used?"
  ],
  "next": "If you can answer all questions, proceed to phase_2"
}
```

### ActionType: "report"

Request a summary or synthesis.

```typescript
{
  step: number;
  action: "report";
  task: string;        // REQUIRED - What to report
  include: string[];   // REQUIRED - What to include in report
}
```

**Example:**
```json
{
  "step": 1,
  "action": "report",
  "task": "Summarize your understanding of the project",
  "include": [
    "Project purpose and scope",
    "Architecture and design patterns",
    "Key components and relationships"
  ]
}
```

### ActionType: "complete"

Mark exploration as finished.

```typescript
{
  step: number;
  action: "complete";
  message: string;     // REQUIRED - Completion message
  next_steps: string[]; // OPTIONAL - Suggested next actions
}
```

**Example:**
```json
{
  "step": 2,
  "action": "complete",
  "message": "✅ Project exploration complete",
  "next_steps": [
    "Ready for specific questions",
    "Can assist with targeted modifications"
  ]
}
```

---

## IgnoranceZone

Explicitly defines files/directories to **not** explore.

```typescript
{
  path: string;        // REQUIRED - Path pattern (exact or glob)
  reason: string;      // REQUIRED - Why to ignore this path
}
```

**Example:**
```json
{
  "ignorance_zones": [
    {
      "path": "node_modules/",
      "reason": "External dependencies, not project code"
    },
    {
      "path": "__pycache__/",
      "reason": "Build artifacts, not source code"
    },
    {
      "path": "tests/",
      "reason": "Test code not needed for initial understanding"
    }
  ]
}
```

**Path Patterns:**
- Exact match: `"LICENSE"`, `"README.md"`
- Directory: `"node_modules/"`, `"dist/"`
- Glob-style: `"*.log"`, `"temp_*"`

---

## CompletionCriteria

Defines what "complete understanding" means.

```typescript
Record<string, string>
```

**Example:**
```json
{
  "completion_criteria": {
    "context": "AI understands project purpose and scope",
    "structure": "AI knows architecture and main components",
    "dependencies": "AI is aware of external requirements",
    "ready": "AI can answer questions or assist with tasks"
  }
}
```

**Use Cases:**
- Verification: Check if AI actually understood
- Documentation: Explain what guide achieves
- Testing: Validate guide effectiveness

---

## Validation Rules

### File Paths

- MUST be relative to project root
- MUST use forward slashes (`/`) on all platforms
- SHOULD NOT start with `./`
- SHOULD NOT use `..` for parent directories

**✅ Valid:**
```json
"path": "src/main.py"
"path": "docs/api/README.md"
```

**❌ Invalid:**
```json
"path": "./src/main.py"
"path": "../other-project/file.py"
"path": "C:\\absolute\\path\\file.py"
```

### Phase IDs

- MUST be unique within a guide
- SHOULD use snake_case
- SHOULD be descriptive

**✅ Valid:**
```json
"id": "phase_1"
"id": "auth_exploration"
"id": "database_schema"
```

**❌ Invalid:**
```json
"id": "Phase 1"        // No spaces
"id": "phase-1"        // Use underscore
"id": "p1"             // Not descriptive
```

### Step Numbers

- MUST be sequential within a phase
- SHOULD start at 1
- SHOULD increment by 1

**✅ Valid:**
```json
{ "step": 1, ... },
{ "step": 2, ... },
{ "step": 3, ... }
```

**❌ Invalid:**
```json
{ "step": 0, ... },    // Start at 1
{ "step": 1, ... },
{ "step": 3, ... }     // Skipped 2
```

---

## Example: Minimal Valid Guide

```json
{
  "guide_name": "Minimal Project Guide",
  "version": "0.0.1",
  "description": "The simplest possible TourGuide",
  "ai_instructions": {
    "role": "You are exploring this project",
    "task": "Read the specified files",
    "rules": ["Follow the sequence"]
  },
  "navigation": {
    "start_here": "phase_1",
    "phases": [
      {
        "id": "phase_1",
        "name": "Overview",
        "description": "Read the README",
        "sequence": [
          {
            "step": 1,
            "action": "read_file",
            "path": "README.md"
          }
        ]
      }
    ]
  }
}
```

---

## Example: Complete Guide

```json
{
  "guide_name": "Complete Project Guide",
  "version": "0.0.1",
  "description": "Comprehensive exploration with all features",
  "ai_instructions": {
    "role": "You are conducting a thorough exploration",
    "task": "Build complete understanding",
    "rules": [
      "Read files in exact order",
      "Complete checkpoints before proceeding",
      "Respect ignorance zones"
    ]
  },
  "navigation": {
    "start_here": "phase_1",
    "phases": [
      {
        "id": "phase_1",
        "name": "Context",
        "description": "Understand purpose",
        "sequence": [
          {
            "step": 1,
            "action": "read_file",
            "path": "README.md",
            "focus": ["Purpose", "Features"]
          },
          {
            "step": 2,
            "action": "checkpoint",
            "verify": ["What does this project do?"],
            "next": "Proceed to phase_2"
          }
        ]
      },
      {
        "id": "phase_2",
        "name": "Implementation",
        "description": "Explore code",
        "sequence": [
          {
            "step": 1,
            "action": "follow_guide",
            "path": "src/tourguide.json"
          },
          {
            "step": 2,
            "action": "report",
            "task": "Summarize understanding",
            "include": ["Architecture", "Key components"]
          },
          {
            "step": 3,
            "action": "complete",
            "message": "Exploration complete"
          }
        ]
      }
    ]
  },
  "ignorance_zones": [
    {
      "path": "node_modules/",
      "reason": "External dependencies"
    }
  ],
  "completion_criteria": {
    "understanding": "AI knows project purpose and architecture"
  }
}
```

---

## Version History

### 0.1.0 (Current)
- Initial schema definition
- Core action types: `read_file`, `follow_guide`, `checkpoint`, `report`, `complete`
- Support for ignorance zones and completion criteria

---

## Future Considerations

Potential additions in future versions:
- **Conditional navigation**: If/then paths based on checkpoint answers
- **Variables**: Parameterized paths or content
- **Metadata**: Author, created date, last updated
- **Dependencies**: Required tools or permissions

---

**For usage examples, see [User Guide](GUIDE.md)**
