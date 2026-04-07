# Claude Code Token Optimization Intel

This repository contains critical research and configuration adjustments to mitigate massive token waste in Claude Code and agentic workflows. 

## Source Material
Based on the audit of 926 Claude Code sessions by Om Patel (18,903 turns, $1,619 estimated spend).

## Key Findings & Required Changes

### 1. Tool Context Bloat (The 14k Token Tax)
By default, Claude Code loads the full JSON schema for every tool into context at the start of the session, whether used or not. This creates an immediate 20,000 token footprint.

**The Fix:**
Add this to your `settings.json`:
```json
"ENABLE_TOOL_SEARCH": "true"
```
*Impact: Drops context from 45K to 20K instantly by deferring tool schema loading until needed.*

### 2. Cache Expiry (The Coffee Break Tax)
54% of turns happened after a 5+ minute idle gap. Claude's cache expires at 5 minutes. If you pause to get coffee and send a new message, the *entire conversation* is reprocessed at full price (a 10x cost jump) because the cache expired.
*Impact: Wasted 12.3 million tokens across the audit.*

**The Fix:**
Be extremely aggressive about starting fresh sessions if you step away, or rely on the `THREAD_HANDOFF.md` and Fabrica pipeline approach to persist state outside of context windows.

### 3. Skill Overload
The audit found 42 skills loaded, with 19 of them used twice or less across 858 sessions. Every unused skill schema sits in the prompt, bleeding tokens on every turn.
**The Fix:**
Only load MCPs and skills specifically required for the task. Turn off globally active tools if they aren't needed.

### 4. Redundant File Reads
The audit found 1,122 redundant file reads where the same file was read 3+ times (one session read the same file 33 times).
**The Fix:**
Enforce the `MODULE_MAP.md` rule. The agent should read the map, not the full file, unless executing changes.

## Integration with Fabrica Protocol
These findings validate the Fabrica Protocol's core philosophy:
- We do not pass chat history (mitigates Cache Expiry tax).
- We use `VAULT.md` and `MODULE_MAP.md` (mitigates Redundant File Reads).
- We should ensure `ENABLE_TOOL_SEARCH` is active on any work machines running Claude Code.