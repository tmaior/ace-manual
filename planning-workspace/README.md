# Planning Workspace

This workspace serves as the documentation and planning hub for AI agents working on the ACE Manual project. It provides a structured approach to managing objectives, tasks, and documentation flow.

## Overview

The planning workspace establishes a systematic documentation workflow that guides AI agents through:
1. **Initial Planning** - Defining objectives and scope
2. **Architecture** - Documenting system design and technical decisions
3. **Engineering** - Planning implementation milestones
4. **Task Management** - Tracking and executing specific tasks

## Folder Structure

```
planning-workspace/
├── README.md                          # This file - entry point for AI agents
├── RULES.md                           # Rules and guidelines for working in this workspace
├── 00-default/                        # Template folder for new task/objective folders
│   ├── README.md                      # Template for task documentation
│   ├── objective.md                   # Template for objective definition
│   ├── notes.md                       # Template for working notes
│   ├── devlogs/                       # Development logs for tracking progress
│   │   ├── README.md                  # Devlogs purpose and usage guide
│   │   └── devlog-entry.md            # Template for new devlog entries
│   └── assets/                        # Folder for any supporting assets
├── 01-project-name/                   # Example task folder (increment number prefix)
│   ├── README.md
│   ├── objective.md
│   ├── notes.md
│   └── assets/
└── [NN-*-name/]                       # Additional task/objective folders
```

### Folder Naming Convention

Task/objective folders follow this pattern:
- **Prefix**: Two-digit number (starting from 00 for default/template)
- **Separator**: Hyphen (-)
- **Name**: Descriptive lowercase name with hyphens for spaces

Examples:
- `00-default` - Template folder (DO NOT MODIFY)
- `01-user-authentication` - First task
- `02-api-integration` - Second task
- `03-database-migration` - Third task

## Documentation Workflow

The planning workspace follows a structured documentation flow:

### 1. Initial Phase
- Define the overall objective/scope
- Identify key requirements
- Establish success criteria

**Documentation Required:**
- `objective.md` - What needs to be accomplished
- `README.md` - Overview and context

### 2. Architecture Phase
- Document system design
- Define component relationships
- Establish design patterns and technical decisions

**Documentation Required:**
- Architecture diagrams
- Component specifications
- Design decisions log

### 3. Engineering Phase
- Break down into milestones
- Define technical requirements
- Create implementation timeline

**Documentation Required:**
- `milestones.md` - Milestone definitions
- `notes.md` - Working notes and progress

### 4. Tasks Management Phase
- Create specific task folders
- Track individual task progress
- Document deliverables

**Documentation Required:**
- Task-specific README
- Task completion status
- Supporting documentation

## Creating a New Task/Objective Folder

### Step-by-Step Process

1. **Copy the template folder:**
   ```bash
   cp -r 00-default/ 01-your-task-name/
   ```

2. **Rename with incremented number:**
   - Use the next available two-digit number
   - Example: If `02-feature-x` exists, create `03-feature-y`

3. **Update the folder contents:**
   - Edit `README.md` with task-specific information
   - Update `objective.md` with the task objective
   - Clear `notes.md` for fresh notes

4. **Add task to tracking:**
   - Update the parent workspace README with the new task

## Documentation Types

### README.md
- **Purpose**: Entry point and overview for the task/objective folder
- **Contents**: 
  - Brief description of the task
  - Context and background
  - Links to related documents
  - Current status

### objective.md
- **Purpose**: Clear definition of what needs to be accomplished
- **Contents**:
  - Primary objective statement
  - Success criteria
  - Constraints and dependencies
  - Acceptance criteria

### notes.md
- **Purpose**: Working notes, decisions, and progress tracking
- **Contents**:
  - Meeting notes
  - Decision log
  - Progress updates
  - Open questions
  - Action items

### Assets Folder
- **Purpose**: Store supporting files (diagrams, images, etc.)
- **Contents**:
  - Architecture diagrams
  - Screenshots
  - Data files
  - Reference materials

## Quick Reference

| Action | Command/Step |
|--------|--------------|
| Create new task | Copy `00-default/` and rename with next number |
| Start working | Read `README.md` and `objective.md` first |
| Track progress | Update `notes.md` regularly |
| Complete task | Update status in `README.md` |
| Ask questions | Check existing docs, then consult parent workspace |

## Navigation

- **[../README.md](../README.md)** - Parent workspace documentation
- **[./RULES.md](./RULES.md)** - Rules and guidelines for this workspace
