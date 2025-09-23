# Pragmatic Implementation: Fix, Connect, Automate
*v2 Discovery - 2025-06-23*

## Week 1: Fix What's Broken

### Day 1-2: Context Extraction Pipeline
```bash
# 1. Create unified context extractor
~/____PKM/scripts/extract-llm-contexts.sh

# Pulls from:
- Cursor .specstory folders
- Claude Code exports  
- ChatGPT/Gemini if needed

# Outputs to:
- Single mega-note: YYYY-MM-DD-llm-contexts.md
- Extracted: entities, actions, insights, STAR cases
```

### Day 3: Fix tg2prompt Contact Issue
```python
# In messages-tg2prompt-telegram-retriever-handler-tdl-manager.py
# Add fallback for unknown contacts:
if not contact_found:
    # Create temporary contact entry
    add_to_pending_contacts_csv(telegram_id, username)
    # Continue processing with placeholder
```

### Day 4-5: GitHub Projects Setup
```yaml
Structure:
  Epics:
    - Voice Pipeline Enhancement
    - Telegram Intelligence
    - Career STAR Cases
    - Digital Twin Core
  
  Automation:
    - Webhook from extractions → Create issues
    - Daily summary → Update project boards
    - Pattern detection → Flag blockers
```

## Week 2: Connect What Works

### Toggl Automation
```python
# Agent watches:
- Current Obsidian file
- Active terminal/app
- Telegram activity
- Calendar events

# Auto-tracks time with smart categories
# No manual start/stop needed
```

### Voice Entity Extraction
```python
# Add to existing pipeline:
SuperWhisper JSON → Entity extractor → 
  - People mentioned → Contacts
  - Tasks mentioned → GitHub issues
  - Ideas → Atomic notes
  - Emotions/energy → Pattern tracking
```

### Unified Daily Summary
```markdown
## 🌅 {{date}} Summary

### 🎤 Voice Insights (auto)
- Key themes: [extracted]
- Energy level: [detected]
- Tasks captured: [listed]

### 💬 Telegram Activity (auto)
- Active conversations: [count]
- Pending responses: [list]
- Relationship updates: [who/what]

### 🧠 LLM Sessions (auto)
- Projects worked: [list]
- Key decisions: [extracted]
- STAR cases: [captured]

### ⏱️ Time Distribution (Toggl)
[Auto-generated chart]
```

## Week 3: Simple Automation

### Cron Jobs That Matter
```cron
# Every 15 min: Process new voice transcripts
*/15 * * * * /Users/user/____PKM/scripts/process-voice-notes.sh

# Every hour: Extract Cursor contexts
0 * * * * /Users/user/____PKM/scripts/extract-llm-contexts.sh

# Every 3 hours: Update GitHub Projects
0 */3 * * * /Users/user/____PKM/scripts/sync-github-projects.sh

# Daily at 5:00: Generate morning brief
0 5 * * * /Users/user/____PKM/scripts/morning-briefing.sh

# Daily at 18:00: Generate day summary
0 18 * * * /Users/user/____PKM/scripts/daily-summary.sh
```

### Docker Containers for Agents
```yaml
version: '3.8'
services:
  context-extractor:
    build: ./agents/context-extractor
    volumes:
      - ~/.specstory:/data/specstory:ro
      - ./outputs:/outputs
    environment:
      - OBSIDIAN_VAULT=/path/to/vault
  
  telegram-processor:
    build: ./agents/telegram
    depends_on:
      - postgres
    environment:
      - TG_CONFIG=/config/telegram.json
  
  pattern-detector:
    build: ./agents/patterns
    environment:
      - NOTIFICATION_WEBHOOK=${TELEGRAM_BOT_WEBHOOK}
```

## Week 4: Career Intelligence

### STAR Case Extractor
```python
# Watches all inputs for career-relevant content
class STARCaseBuilder:
    patterns = {
        'situation': ['worked on', 'project', 'challenge'],
        'task': ['needed to', 'required', 'goal was'],
        'action': ['I built', 'implemented', 'designed'],
        'result': ['achieved', 'improved', 'saved']
    }
    
    def extract_from_context(self, text):
        # Find potential STAR stories
        # Tag with relevant skills
        # Link to job descriptions
        # Build interview prep database
```

### AI Solution Architect Profile
```markdown
## Profile: AI Integration Specialist

### Proven Abilities (auto-extracted)
- Integrated 10+ LLMs into production systems
- Built voice-to-knowledge pipeline processing 100+ hours
- Managed 1000+ contact CRM via Telegram automation
- Orchestrated multi-agent systems for PKM

### STAR Cases (auto-maintained)
1. **Telegram Assistant Project**
   - S: Needed unified communication intelligence
   - T: Design system for 1000+ contacts
   - A: Built Python/TDLib integration with...
   - R: 90% reduction in missed follow-ups
```

## Success Metrics

### Week 1
- All LLM contexts extracted daily
- tg2prompt handles new contacts
- GitHub Projects receiving auto-issues

### Week 2  
- Toggl tracks automatically
- Voice extracts entities
- Daily summaries generated

### Week 3
- All cron jobs running
- Docker agents deployed
- Pattern detection active

### Week 4
- STAR cases building automatically
- Career profile self-updating
- Interview prep materials ready

## Remember: YAGNI + KISS

- Use existing tools (Toggl, GitHub, Obsidian)
- Simple cron > complex orchestration
- Fix before building
- Automate what already works manually

## Next: [[03-the-real-digital-twin]]