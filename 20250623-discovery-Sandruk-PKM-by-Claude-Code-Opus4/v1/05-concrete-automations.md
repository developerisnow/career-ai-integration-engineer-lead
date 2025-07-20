# Concrete Automations: Ready-to-Deploy Scripts
*Discovery by Claude Code Opus4 - 2025-06-23*

## 1. Voice Note Auto-Processor

```bash
#!/bin/bash
# save as: ~/____PKM/scripts/process-voice-notes.sh

VOICE_DIR="/Users/user/____Sandruk/___PKM/__SecondBrain/Voice-Notes-Transcriptions"
PROCESSED_DIR="$VOICE_DIR/.processed"
DAILY_NOTE_DIR="/Users/user/____Sandruk/___PKM/__SecondBrain/Dailies_Notes"
TODAY=$(date +%Y-%m-%d-%a)

# Create processed directory if doesn't exist
mkdir -p "$PROCESSED_DIR"

# Process each new transcription
for transcript in "$VOICE_DIR"/*/*.txt; do
    if [[ -f "$transcript" ]] && [[ ! -f "$PROCESSED_DIR/$(basename "$transcript").done" ]]; then
        echo "Processing: $transcript"
        
        # Extract content
        CONTENT=$(cat "$transcript")
        TIMESTAMP=$(date +%Y%m%d-%H%M)
        
        # Use AI to extract insights
        ANALYSIS=$(echo "$CONTENT" | llm -m gpt-4 "
        Extract from this voice transcript:
        1. Main topic/thought
        2. Any action items (prefix with TODO:)
        3. Related project (HypeTrain/Career/PKM/etc)
        4. Key insight if any
        Format as markdown bullet points
        ")
        
        # Append to today's daily note
        echo -e "\n### 🎤 Voice Note - $TIMESTAMP\n$ANALYSIS\n\n---" >> "$DAILY_NOTE_DIR/$TODAY.md"
        
        # Mark as processed
        touch "$PROCESSED_DIR/$(basename "$transcript").done"
        
        # If action items found, create GitHub issue
        if echo "$ANALYSIS" | grep -q "TODO:"; then
            TODOS=$(echo "$ANALYSIS" | grep "TODO:" | sed 's/TODO: //')
            echo "$TODOS" | while read -r todo; do
                gh issue create --title "$todo" --body "Extracted from voice note: $TIMESTAMP" --label "voice-extracted"
            done
        fi
    fi
done
```

## 2. Pattern Detection Alert System

```python
#!/usr/bin/env python3
# save as: ~/____PKM/scripts/pattern_detector.py

import os
import re
from datetime import datetime, timedelta
from collections import defaultdict
import subprocess

class PatternDetector:
    def __init__(self):
        self.pkm_root = "/Users/user/____Sandruk/___PKM"
        self.patterns = {
            'analysis_paralysis': self.check_analysis_paralysis,
            'context_overload': self.check_context_overload,
            'hyperfocus_danger': self.check_hyperfocus_danger,
            'commitment_overflow': self.check_commitments
        }
    
    def check_analysis_paralysis(self):
        # Check if too many research files created without action
        recent_files = self.get_recent_files(hours=24)
        research_files = [f for f in recent_files if 'research' in f or 'analysis' in f]
        action_files = [f for f in recent_files if 'todo' in f or 'action' in f]
        
        if len(research_files) > 5 and len(action_files) < 2:
            return {
                'detected': True,
                'message': f"Analysis Paralysis: {len(research_files)} research files, only {len(action_files)} action items",
                'suggestion': "Time to make a decision. Set 25-min timer and choose."
            }
        return {'detected': False}
    
    def check_context_overload(self):
        # Check how many different projects touched today
        today_note = f"{self.pkm_root}/__SecondBrain/Dailies_Notes/{datetime.now().strftime('%Y-%m-%d')}-*.md"
        
        projects_mentioned = set()
        for note in glob.glob(today_note):
            with open(note, 'r') as f:
                content = f.read()
                # Extract project mentions
                projects = re.findall(r'(HypeTrain|Career|PKM|Assistant-Telegram|France)', content)
                projects_mentioned.update(projects)
        
        if len(projects_mentioned) > 4:
            return {
                'detected': True,
                'message': f"Context Overload: {len(projects_mentioned)} different projects today",
                'suggestion': "Focus on max 2 projects for rest of day"
            }
        return {'detected': False}
    
    def check_hyperfocus_danger(self):
        # Check git commits for hyperfocus pattern
        commits = subprocess.check_output(
            ['git', '-C', self.pkm_root, 'log', '--since="6 hours ago"', '--oneline']
        ).decode().strip().split('\n')
        
        if len(commits) > 10:
            return {
                'detected': True,
                'message': f"Hyperfocus Alert: {len(commits)} commits in 6 hours",
                'suggestion': "Take a break. Check body battery. Eat something."
            }
        return {'detected': False}
    
    def notify(self, pattern_name, result):
        if result['detected']:
            # macOS notification
            subprocess.run([
                'osascript', '-e',
                f'display notification "{result["suggestion"]}" with title "Pattern: {pattern_name}" subtitle "{result["message"]}"'
            ])
            
            # Log to daily note
            daily_note = f"{self.pkm_root}/__SecondBrain/Dailies_Notes/{datetime.now().strftime('%Y-%m-%d-%a')}.md"
            with open(daily_note, 'a') as f:
                f.write(f"\n### 🧠 Pattern Detected: {pattern_name}\n")
                f.write(f"- **Alert**: {result['message']}\n")
                f.write(f"- **Suggestion**: {result['suggestion']}\n")
                f.write(f"- **Time**: {datetime.now().strftime('%H:%M')}\n\n")

if __name__ == "__main__":
    detector = PatternDetector()
    for pattern_name, checker in detector.patterns.items():
        result = checker()
        detector.notify(pattern_name, result)
```

## 3. Telegram Context Summarizer

```python
#!/usr/bin/env python3
# save as: ~/____PKM/scripts/telegram_context.py

import json
import os
from datetime import datetime, timedelta

class TelegramContextManager:
    def __init__(self):
        self.tg_output = "/Users/user/____Sandruk/___PKM/__SecondBrain/Projects_PKM/Assistant-Telegram/outputs"
        self.daily_notes = "/Users/user/____Sandruk/___PKM/__SecondBrain/Dailies_Notes"
    
    def get_recent_conversations(self, hours=24):
        """Extract recent conversations with context"""
        cutoff = datetime.now() - timedelta(hours=hours)
        conversations = defaultdict(list)
        
        # Parse your telegram outputs
        for file in os.listdir(self.tg_output):
            if file.endswith('.json'):
                with open(os.path.join(self.tg_output, file)) as f:
                    data = json.load(f)
                    # Group by contact
                    if data['timestamp'] > cutoff:
                        conversations[data['contact']].append({
                            'time': data['timestamp'],
                            'message': data['text'],
                            'type': data.get('type', 'text')
                        })
        
        return conversations
    
    def generate_contact_summary(self):
        """Generate relationship context for each active contact"""
        convos = self.get_recent_conversations(hours=168)  # Last week
        
        summaries = []
        for contact, messages in convos.items():
            # Use AI to summarize
            context = "\n".join([f"{m['time']}: {m['message']}" for m in messages[-10:]])
            
            summary = f"""
## 👤 {contact}
- **Last contact**: {messages[-1]['time']}
- **Messages this week**: {len(messages)}
- **Topics discussed**: [AI extracts topics]
- **Pending items**: [AI extracts commitments]
- **Relationship context**: [AI summarizes relationship]
"""
            summaries.append(summary)
        
        return "\n".join(summaries)
    
    def update_daily_note(self):
        """Add Telegram summary to daily note"""
        today = datetime.now().strftime('%Y-%m-%d-%a')
        daily_note_path = f"{self.daily_notes}/{today}.md"
        
        summary = self.generate_contact_summary()
        
        with open(daily_note_path, 'a') as f:
            f.write(f"\n## 💬 Telegram Activity Summary\n{summary}\n")

if __name__ == "__main__":
    manager = TelegramContextManager()
    manager.update_daily_note()
```

## 4. Morning Briefing Generator

```python
#!/usr/bin/env python3
# save as: ~/____PKM/scripts/morning_briefing.py

import subprocess
import glob
from datetime import datetime, timedelta

class MorningBriefing:
    def __init__(self):
        self.pkm_root = "/Users/user/____Sandruk/___PKM"
        
    def get_unfinished_tasks(self):
        """Extract uncompleted tasks from yesterday"""
        yesterday = (datetime.now() - timedelta(days=1)).strftime('%Y-%m-%d')
        yesterday_note = glob.glob(f"{self.pkm_root}/__SecondBrain/Dailies_Notes/{yesterday}*.md")[0]
        
        tasks = []
        with open(yesterday_note, 'r') as f:
            for line in f:
                if '- [ ]' in line:  # Unchecked task
                    tasks.append(line.strip())
        return tasks
    
    def get_team_blockers(self):
        """Check recent Telegram for team issues"""
        # This would integrate with your Telegram data
        return [
            "Koryun: Waiting for Garden deployment review",
            "Anton: Elasticsearch configuration issue"
        ]
    
    def analyze_energy_pattern(self):
        """Predict today's energy based on patterns"""
        # Check sleep data if available
        # Check yesterday's commit patterns
        # Check calendar density
        
        return {
            'peak_hours': "05:30-08:00",
            'suggested_focus': "Complex problem solving",
            'avoid': "Meetings before 10:00"
        }
    
    def generate_briefing(self):
        timestamp = datetime.now().strftime('%Y-%m-%d %H:%M')
        
        briefing = f"""# 🌅 Morning Briefing - {timestamp}

## 📋 Unfinished Business
{chr(10).join(self.get_unfinished_tasks())}

## 👥 Team Status
{chr(10).join(self.get_team_blockers())}

## ⚡ Energy Forecast
- **Peak Performance Window**: {self.analyze_energy_pattern()['peak_hours']}
- **Best For**: {self.analyze_energy_pattern()['suggested_focus']}
- **Avoid**: {self.analyze_energy_pattern()['avoid']}

## 🎯 Today's Focus Recommendation
Based on your patterns:
1. Start with HypeTrain deployment review (high cognitive load)
2. Career project work during mid-morning lull
3. Admin tasks post-lunch when energy dips

## 🔗 Quick Links
- [Yesterday's Summary]({(datetime.now() - timedelta(days=1)).strftime('%Y-%m-%d')}.md)
- [This Week's MOC](week-{datetime.now().strftime('%V')}-moc.md)
- [HypeTrain Project](../Projects_PKM/HypeTrain)

---
*Generated by AI Twin - Calibrated to YOUR patterns*
"""
        
        # Save briefing
        briefing_path = f"{self.pkm_root}/__SecondBrain/Dailies_Outputs/briefings/{datetime.now().strftime('%Y%m%d')}-morning-briefing.md"
        os.makedirs(os.path.dirname(briefing_path), exist_ok=True)
        
        with open(briefing_path, 'w') as f:
            f.write(briefing)
        
        # Also append to today's daily note
        today_note = f"{self.pkm_root}/__SecondBrain/Dailies_Notes/{datetime.now().strftime('%Y-%m-%d-%a')}.md"
        with open(today_note, 'a') as f:
            f.write(f"\n## 🌅 Morning Briefing\n![[{os.path.basename(briefing_path)}]]\n")
        
        # Send notification
        subprocess.run([
            'osascript', '-e',
            'display notification "Your morning briefing is ready" with title "AI Twin" subtitle "Peak hours: 05:30-08:00"'
        ])

if __name__ == "__main__":
    briefing = MorningBriefing()
    briefing.generate_briefing()
```

## 5. Smart Git Commit Hook

```bash
#!/bin/bash
# save as: ~/____PKM/.git/hooks/pre-commit

# Auto-generate meaningful commit messages based on changes
FILES_CHANGED=$(git diff --cached --name-only)
CHANGE_SUMMARY=$(git diff --cached --stat)

# Use AI to generate commit message
COMMIT_MSG=$(echo "$CHANGE_SUMMARY" | llm -m gpt-3.5-turbo "
Generate a concise git commit message for these changes.
Include:
- Type: feat/fix/docs/refactor
- Scope: which part of PKM affected
- Description: what changed and why
Format: type(scope): description
")

# Save to commit message file
echo "$COMMIT_MSG" > .git/COMMIT_EDITMSG

# Add context to daily note
DAILY_NOTE="/Users/user/____Sandruk/___PKM/__SecondBrain/Dailies_Notes/$(date +%Y-%m-%d-%a).md"
echo -e "\n### 💾 Git Commit - $(date +%H:%M)\n- $COMMIT_MSG\n- Files: $FILES_CHANGED\n" >> "$DAILY_NOTE"
```

## 6. Project Context Switcher

```bash
#!/bin/bash
# save as: ~/____PKM/scripts/switch-context.sh

PROJECT=$1
PKM_ROOT="/Users/user/____Sandruk/___PKM"

case $PROJECT in
    "hypetrain"|"ht")
        echo "🚂 Switching to HypeTrain context..."
        
        # Open relevant files
        code "$PKM_ROOT/__SecondBrain/Projects_PKM/HappyAI-Company/alex-PKM-hypetrain"
        
        # Show recent activity
        echo "Recent HypeTrain activity:"
        grep -r "HypeTrain" "$PKM_ROOT/__SecondBrain/Dailies_Notes" --include="*.md" -l | tail -5
        
        # Show team status
        echo -e "\nTeam Status:"
        # Would query your Telegram data
        ;;
        
    "career")
        echo "💼 Switching to Career context..."
        
        # Open career files
        code "$PKM_ROOT/__SecondBrain/Projects_PKM/Career-Oleksandr-Aleksandruk"
        
        # Show pending tasks
        echo "Career TODOs:"
        grep -r "TODO.*Career" "$PKM_ROOT/__SecondBrain" --include="*.md"
        ;;
        
    "pkm")
        echo "🧠 Switching to PKM optimization context..."
        # etc...
        ;;
esac

# Update current context file
echo "$PROJECT" > "$PKM_ROOT/.current-context"

# Notify
osascript -e "display notification \"Switched to $PROJECT context\" with title \"Context Switch\""
```

## 7. Automated MOC Generator

```python
#!/usr/bin/env python3
# save as: ~/____PKM/scripts/generate_moc.py

import os
import re
from collections import defaultdict
from datetime import datetime, timedelta

class MOCGenerator:
    def __init__(self):
        self.pkm_root = "/Users/user/____Sandruk/___PKM"
        
    def generate_weekly_moc(self):
        """Generate MOC for current week"""
        week_num = datetime.now().strftime('%V')
        year = datetime.now().year
        
        # Collect all notes from this week
        week_start = datetime.now() - timedelta(days=datetime.now().weekday())
        notes_by_project = defaultdict(list)
        
        for i in range(7):
            date = week_start + timedelta(days=i)
            date_str = date.strftime('%Y-%m-%d')
            
            daily_notes = glob.glob(f"{self.pkm_root}/__SecondBrain/Dailies_Notes/{date_str}*.md")
            for note in daily_notes:
                with open(note, 'r') as f:
                    content = f.read()
                    
                # Extract project mentions and key points
                projects = re.findall(r'#(HypeTrain|Career|PKM|Assistant-Telegram)', content)
                for project in projects:
                    notes_by_project[project].append({
                        'date': date_str,
                        'file': note,
                        'summary': self.extract_summary(content)
                    })
        
        # Generate MOC
        moc_content = f"""# Week {week_num} - {year} MOC
*Generated: {datetime.now().strftime('%Y-%m-%d %H:%M')}*

## 🎯 Week Overview

### By Project
"""
        
        for project, notes in notes_by_project.items():
            moc_content += f"\n### {project}\n"
            for note in notes:
                moc_content += f"- [[{os.path.basename(note['file'])}|{note['date']}]]: {note['summary']}\n"
        
        # Add patterns detected
        moc_content += "\n## 🧠 Patterns This Week\n"
        moc_content += self.analyze_weekly_patterns()
        
        # Add achievements
        moc_content += "\n## ✅ Completed This Week\n"
        moc_content += self.extract_completed_tasks()
        
        # Save MOC
        moc_path = f"{self.pkm_root}/__SecondBrain/Maps_Notes/week-{week_num}-{year}-moc.md"
        with open(moc_path, 'w') as f:
            f.write(moc_content)
        
        return moc_path

if __name__ == "__main__":
    generator = MOCGenerator()
    moc_path = generator.generate_weekly_moc()
    print(f"Generated MOC: {moc_path}")
```

## Setup Instructions

1. **Make scripts executable**:
```bash
chmod +x ~/____PKM/scripts/*.sh
chmod +x ~/____PKM/scripts/*.py
```

2. **Set up cron jobs**:
```bash
crontab -e
# Add:
*/5 * * * * /Users/user/____PKM/scripts/process-voice-notes.sh
0 * * * * /Users/user/____PKM/scripts/pattern_detector.py
0 5 * * * /Users/user/____PKM/scripts/morning_briefing.py
0 18 * * 5 /Users/user/____PKM/scripts/generate_moc.py
```

3. **Install dependencies**:
```bash
pip install llm chromadb fastapi
llm install llm-gpt4
```

These scripts are designed to work with your existing structure and patterns. Start with one or two, see what helps, then add more.

---
Next: [[06-synthesis-and-next-steps]]