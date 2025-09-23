# Immediate Next Steps: What to Do RIGHT NOW
*v2 Discovery - 2025-06-23*

## Today (Next 2 Hours)

### 1. Create Context Extraction Script
```bash
#!/bin/bash
# Save as: ~/____PKM/scripts/extract-cursor-contexts.sh

SPECSTORY_DIRS=$(find /Users/user/__Repositories -name ".specstory" -type d)
OUTPUT_DIR="/Users/user/____Sandruk/___PKM/__SecondBrain/Dailies_Outputs/llm-contexts"
TODAY=$(date +%Y%m%d)

mkdir -p "$OUTPUT_DIR"

for dir in $SPECSTORY_DIRS; do
    PROJECT=$(basename $(dirname "$dir"))
    echo "Processing $PROJECT..."
    
    # Extract just prompts from today
    find "$dir" -name "*.md" -mtime -1 -exec grep -A2 "^User:" {} \; >> "$OUTPUT_DIR/$TODAY-$PROJECT-prompts.md"
done

# Run claude-code-exporter
claude-code-exporter --output "$OUTPUT_DIR/$TODAY-claude-prompts.md"

echo "Contexts extracted to $OUTPUT_DIR"
```

### 2. Fix tg2prompt Contact Issue
```python
# Quick patch for contact lookup
# Add to your existing script around line 324:

def get_contact_by_id(contact_id):
    # Existing lookup code...
    
    if not contact_found:
        # Add pending contact
        with open('pending_contacts.csv', 'a') as f:
            f.write(f"{contact_id},{username},PENDING\n")
        
        # Return temporary entry
        return {
            'id': contact_id,
            'name': f'Unknown-{contact_id}',
            'username': username or 'unknown'
        }
```

### 3. Set Up GitHub Project
```bash
# Using GitHub CLI
gh project create "Digital Twin Development" --owner @me --public

# Create initial structure
gh issue create --title "Context Extraction Pipeline" \
  --body "Extract prompts from Cursor, Claude Code, etc." \
  --label "epic"

gh issue create --title "Fix tg2prompt contact lookup" \
  --body "Handle new contacts gracefully" \
  --label "bug"

gh issue create --title "Voice entity extraction" \
  --body "Add NLP layer to voice pipeline" \
  --label "enhancement"
```

## Tomorrow

### Morning Ritual Setup
```bash
# Add to crontab
0 5 * * * /Users/user/____PKM/scripts/morning-briefing.sh

# Create the script:
#!/bin/bash
# morning-briefing.sh

DAILY_NOTE="/Users/user/____Sandruk/___PKM/__SecondBrain/Dailies_Notes/$(date +%Y-%m-%d-%a).md"

# Get yesterday's unfinished tasks
YESTERDAY="/Users/user/____Sandruk/___PKM/__SecondBrain/Dailies_Notes/$(date -v-1d +%Y-%m-%d)*.md"
UNFINISHED=$(grep -h "^\- \[ \]" $YESTERDAY 2>/dev/null || echo "None")

# Create briefing
cat > "$DAILY_NOTE" << EOF
# $(date +%Y-%m-%d) - Morning Briefing

## 📋 Unfinished from Yesterday
$UNFINISHED

## 🎯 Focus for Today
- Review Koryun's deployment
- Career course exercise
- Process pending Telegrams

## ⚡ Energy Prediction
Based on sleep: $(check_garmin_sleep)
Suggested peak: 05:30-08:00

---
EOF

# Send notification
osascript -e 'display notification "Morning briefing ready" with title "Digital Twin"'
```

## This Week

### 1. Organize Your Scripts
```bash
# Move to atomic repos
cd /Users/user/__Repositories

# Create organized structure
mkdir -p digital-twin-core/{scripts,agents,configs}
mkdir -p llm-context-extractor
mkdir -p tg-prompt-enhanced
mkdir -p voice-pipeline-nlp

# Move existing scripts
cp ~/____PKM/scripts/* digital-twin-core/scripts/
cp ~/.../messages-tg2prompt-*.py tg-prompt-enhanced/
```

### 2. Simple Entity Extraction
```python
# Add to voice pipeline
import spacy
nlp = spacy.load("en_core_web_sm")

def extract_entities(transcript):
    doc = nlp(transcript)
    
    entities = {
        'people': [ent.text for ent in doc.ents if ent.label_ == "PERSON"],
        'tasks': extract_tasks(doc),  # "need to", "should", "will"
        'projects': extract_projects(doc),  # Your project names
        'energy': detect_energy_level(doc)  # "tired", "focused", etc.
    }
    
    return entities
```

### 3. Toggl Automation Prototype
```python
# Simple context-aware tracker
import toggl
from datetime import datetime

class SmartToggler:
    def __init__(self):
        self.client = toggl.TogglClient(API_TOKEN)
        self.current_context = None
    
    def detect_context(self):
        # Check active window
        # Check recent files
        # Check calendar
        # Return best guess of current project
        
    def auto_track(self):
        context = self.detect_context()
        if context != self.current_context:
            self.client.stop_current()
            self.client.start(
                description=f"Auto: {context}",
                project_id=PROJECT_MAP[context]
            )
            self.current_context = context
```

## This Weekend

### Deploy First Agent
```yaml
# docker-compose.yml for context extractor
version: '3.8'
services:
  context-extractor:
    build: .
    volumes:
      - /Users/user/__Repositories:/repos:ro
      - /Users/user/____Sandruk/___PKM:/pkm
    environment:
      - SCHEDULE=0 * * * *  # Every hour
    command: python extract_contexts.py
```

### Create Atomic Scripts Library
```
digital-twin-core/
├── extractors/
│   ├── cursor_prompts.py
│   ├── claude_code.py
│   ├── telegram_messages.py
│   └── voice_transcripts.py
├── processors/
│   ├── entity_extractor.py
│   ├── star_case_builder.py
│   └── pattern_detector.py
├── outputs/
│   ├── obsidian_writer.py
│   ├── github_issues.py
│   └── toggl_tracker.py
└── orchestrator.py
```

## Key Principles

### YAGNI (You Aren't Gonna Need It)
- Don't build a vector database yet
- Don't create complex agent systems
- Don't redesign working pipelines

### KISS (Keep It Simple, Stupid)
- Cron jobs > Kubernetes
- Scripts > Microservices
- Local files > Cloud storage

### DRY (Don't Repeat Yourself)
- Reuse existing scripts
- Extract common functions
- Build on what works

### SSOT (Single Source of Truth)
- PKM is the source
- Everything else is a view
- No duplicate data stores

### JTBD (Jobs To Be Done)
- Extract contexts → Build STAR cases
- Track patterns → Prevent crashes
- Preserve context → Enable flow
- Manage relationships → Reduce overhead

## Success Checklist

### Today ✓
- Context extraction script created
- tg2prompt patched for new contacts
- GitHub Project initialized

### Tomorrow ✓
- Morning briefing automated
- First entities extracted
- Scripts organized

### This Week ✓
- All scripts in atomic repos
- Basic entity extraction working
- Toggl automation prototype
- First agent deployed

### This Month ✓
- Full context pipeline operational
- STAR cases building automatically
- Pattern detection preventing crashes
- Digital twin learning your patterns

Remember: Every step should provide immediate value. No building for future maybe-needs.

## Next: [[05-digital-garden-synthesis]]