# TourGuide User Guide

**How to Create and Use TourGuide Files**

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Creating Your First Guide](#creating-your-first-guide)
3. [Guide Structure](#guide-structure)
4. [Best Practices](#best-practices)
5. [Common Patterns](#common-patterns)
6. [Troubleshooting](#troubleshooting)

---

## Quick Start

**5-Minute Setup:**

1. Copy `templates/minimal.json` to your project root
2. Rename it to `tourguide.json`
3. Update file paths to match your project
4. Show it to your AI assistant

**Example:**
```json
{
  "guide_name": "My Project Guide",
  "version": "0.0.1",
  "description": "Navigate my awesome project",
  "ai_instructions": {
    "role": "You are exploring this project",
    "task": "Read the specified files in order",
    "rules": ["Follow the sequence exactly"]
  },
  "navigation": {
    "start_here": "phase_1",
    "phases": [
      {
        "id": "phase_1",
        "name": "Overview",
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

## Creating Your First Guide

### Step 1: Choose a Template

- **Minimal**: Single-phase guide (2-5 files)
- **Standard**: Multi-phase guide (5-15 files)
- **Codebase**: Comprehensive guide (15+ files, sub-guides)

**When to use which:**
- Small project / quick intro → **Minimal**
- Medium project / typical codebase → **Standard**
- Large/legacy project / deep exploration → **Codebase**

### Step 2: Map Your Project

Before writing the guide, list:
1. **Essential files** (must-read for understanding)
2. **Nice-to-have files** (useful but not critical)
3. **Ignorable files** (build artifacts, vendor code)

**Example mapping:**
```
Essential:
- README.md (overview)
- docs/ARCHITECTURE.md (design)
- src/main.py (entry point)

Nice-to-have:
- CHANGELOG.md (history)
- docs/API.md (reference)

Ignore:
- node_modules/
- __pycache__/
```

### Step 3: Define Phases

Group files into logical phases:
1. **Context** (what is this project?)
2. **Structure** (how is it organized?)
3. **Implementation** (how does it work?)
4. **Configuration** (how to set it up?)

**Example phases:**
```json
"phases": [
  {
    "id": "phase_1",
    "name": "Project Context",
    "description": "Understand purpose and scope"
  },
  {
    "id": "phase_2",
    "name": "Core Implementation",
    "description": "Learn how it works"
  }
]
```

### Step 4: Add Checkpoints

After each phase, add verification questions:
```json
{
  "action": "checkpoint",
  "verify": [
    "What does this project do?",
    "What are the main components?"
  ],
  "next": "If understood, proceed to phase_2"
}
```

### Step 5: Test Your Guide

Show your guide to an AI and observe:
- Does it follow the sequence?
- Does it skip steps or files?
- Does it complete checkpoints?
- Does it understand the project afterward?

**Iterate based on results.**

---

## Guide Structure

### Required Fields

```json
{
  "guide_name": "string (required)",
  "version": "semver (required)",
  "description": "string (required)",
  "ai_instructions": {
    "role": "string (required)",
    "task": "string (required)",
    "rules": ["array (required)"]
  },
  "navigation": {
    "start_here": "phase_id (required)",
    "phases": ["array (required)"]
  }
}
```

### Optional Fields

```json
{
  "ignorance_zones": [
    {
      "path": "string",
      "reason": "why to ignore"
    }
  ],
  "completion_criteria": {
    "key": "what AI should understand"
  }
}
```

### Phase Structure

```json
{
  "id": "unique_phase_id",
  "name": "Human-readable phase name",
  "description": "What this phase covers",
  "sequence": [
    {
      "step": 1,
      "action": "read_file | follow_guide | checkpoint",
      "path": "file/path (for read_file, follow_guide)",
      "focus": ["what to pay attention to"],
      "verify": ["questions (for checkpoint)"],
      "next": "instructions for next step"
    }
  ]
}
```

---

## Best Practices

### 1. Start Simple

**❌ Don't:**
```json
{
  "phases": [
    { "name": "Phase 1", "sequence": [ /* 20 steps */ ] },
    { "name": "Phase 2", "sequence": [ /* 15 steps */ ] },
    { "name": "Phase 3", "sequence": [ /* 25 steps */ ] }
  ]
}
```

**✅ Do:**
```json
{
  "phases": [
    { "name": "Overview", "sequence": [ /* 3 steps */ ] },
    { "name": "Deep Dive", "sequence": [ /* 5 steps */ ] }
  ]
}
```

Start with 2-3 phases, expand later.

### 2. Use Checkpoints

**❌ Don't:**
```json
{
  "sequence": [
    { "step": 1, "action": "read_file", "path": "file1.py" },
    { "step": 2, "action": "read_file", "path": "file2.py" },
    { "step": 3, "action": "read_file", "path": "file3.py" }
  ]
}
```

**✅ Do:**
```json
{
  "sequence": [
    { "step": 1, "action": "read_file", "path": "file1.py" },
    { "step": 2, "action": "checkpoint", "verify": ["Understood file1?"] },
    { "step": 3, "action": "read_file", "path": "file2.py" }
  ]
}
```

Checkpoint every 3-5 files.

### 3. Be Specific with Focus

**❌ Don't:**
```json
{
  "action": "read_file",
  "path": "src/main.py"
}
```

**✅ Do:**
```json
{
  "action": "read_file",
  "path": "src/main.py",
  "focus": [
    "Entry point function",
    "Initialization sequence",
    "Main execution loop"
  ]
}
```

Tell the AI **what** to look for.

### 4. Use Ignorance Zones

**❌ Don't:**
```json
{
  "sequence": [
    { "action": "read_file", "path": "README.md" },
    // No mention of what to skip
  ]
}
```

**✅ Do:**
```json
{
  "ignorance_zones": [
    { "path": "node_modules/", "reason": "External deps" },
    { "path": "__pycache__/", "reason": "Build artifacts" }
  ]
}
```

Explicitly list what **not** to read.

### 5. Use Sub-Guides for Deep Dives

**❌ Don't:**
```json
{
  "sequence": [
    { "step": 1, "action": "read_file", "path": "src/module1/file1.py" },
    { "step": 2, "action": "read_file", "path": "src/module1/file2.py" },
    { "step": 3, "action": "read_file", "path": "src/module1/file3.py" },
    // 20 more files from module1...
  ]
}
```

**✅ Do:**
```json
{
  "sequence": [
    {
      "step": 1,
      "action": "follow_guide",
      "path": "src/module1/tourguide.json",
      "notes": "Deep dive into module1"
    }
  ]
}
```

Create sub-guides for complex modules.

---

## Common Patterns

### Pattern 1: README → Docs → Code

**Use case:** New contributor onboarding

```json
{
  "phases": [
    {
      "id": "context",
      "name": "Project Context",
      "sequence": [
        { "action": "read_file", "path": "README.md" },
        { "action": "read_file", "path": "docs/ARCHITECTURE.md" }
      ]
    },
    {
      "id": "implementation",
      "name": "Code Exploration",
      "sequence": [
        { "action": "read_file", "path": "src/main.py" },
        { "action": "follow_guide", "path": "src/tourguide.json" }
      ]
    }
  ]
}
```

### Pattern 2: Layer-by-Layer

**Use case:** Understanding architecture layers

```json
{
  "phases": [
    { "id": "api_layer", "name": "API Layer" },
    { "id": "service_layer", "name": "Service Layer" },
    { "id": "data_layer", "name": "Data Layer" }
  ]
}
```

### Pattern 3: Feature-by-Feature

**Use case:** Understanding specific features

```json
{
  "phases": [
    { "id": "auth", "name": "Authentication System" },
    { "id": "payments", "name": "Payment Processing" },
    { "id": "notifications", "name": "Notification System" }
  ]
}
```

### Pattern 4: Problem → Solution

**Use case:** Understanding design decisions

```json
{
  "sequence": [
    {
      "action": "read_file",
      "path": "docs/PROBLEM.md",
      "focus": ["What problem are we solving?"]
    },
    {
      "action": "read_file",
      "path": "docs/SOLUTION.md",
      "focus": ["How did we solve it?"]
    },
    {
      "action": "read_file",
      "path": "src/implementation.py",
      "focus": ["See the solution in code"]
    }
  ]
}
```

---

## Troubleshooting

### AI Skips Steps

**Symptom:** AI jumps ahead or reads files out of order

**Fix:**
1. Make rules more explicit:
   ```json
   "rules": [
     "Read files in EXACT order specified",
     "Do NOT skip ahead",
     "Do NOT summarize until instructed"
   ]
   ```
2. Add more checkpoints to enforce sequence

### AI Gets Overwhelmed

**Symptom:** AI says "too much information" or gives shallow answers

**Fix:**
1. Break into smaller phases
2. Use sub-guides for deep dives
3. Add more ignorance zones
4. Reduce files per phase (max 5-7)

### AI Ignores Checkpoints

**Symptom:** AI doesn't verify understanding before moving on

**Fix:**
1. Make checkpoint questions more specific:
   ```json
   "verify": [
     "What are the 3 main components?",
     "How does authentication work?",
     "What database is used?"
   ]
   ```
2. Add "next" instruction explicitly:
   ```json
   "next": "Do NOT proceed until you can answer all questions"
   ```

### AI Reads Ignored Files

**Symptom:** AI explores files in ignorance zones

**Fix:**
1. Make ignorance zones more explicit
2. Repeat rules in ai_instructions:
   ```json
   "rules": [
     "Respect ignorance zones - do NOT read excluded files"
   ]
   ```

---

## Advanced Topics

### Sub-Guides

Create a guide hierarchy:

```
project/
├── tourguide.json (root guide)
├── src/
│   ├── tourguide.json (code guide)
│   ├── api/
│   │   └── tourguide.json (API guide)
│   └── services/
│       └── tourguide.json (services guide)
```

**Root guide references sub-guides:**
```json
{
  "sequence": [
    {
      "action": "follow_guide",
      "path": "src/tourguide.json"
    }
  ]
}
```

### Conditional Navigation

Use "next" field for branching:

```json
{
  "action": "checkpoint",
  "verify": ["Do you need to understand API details?"],
  "next": "If YES, follow src/api/tourguide.json. If NO, proceed to phase_3"
}
```

### Completion Criteria

Define what "complete understanding" means:

```json
{
  "completion_criteria": {
    "architecture": "AI can draw system diagram",
    "workflows": "AI can explain main user flows",
    "dependencies": "AI knows all external services",
    "ready": "AI can implement features independently"
  }
}
```

---

## Examples

See `examples/` directory for real-world guides:
- **python-app**: Standard Python application
- **docs-site**: Documentation site navigation

---

**Need more help?** Check the [JSON Schema Reference](SCHEMA.md) for technical details.
