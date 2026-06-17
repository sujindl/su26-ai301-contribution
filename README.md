# Contribution #1: [Give people a timeout for yelling too much]

**Contribution Number:** 1  
**Student:** Sujin Lee  
**Issue:** [duckbot #78](https://github.com/duck-dynasty/duckbot/issues/78)  
**Status:** **Phase I Complete** 

---

## Why I Chose This Issue

The duckbot Discord bot currently has no way to discourage users from posting in all caps. This issue asks for a feature that detects excessive all-caps messages, temporarily moves the user to a restricted role for five minutes, and welcomes them back when the timeout ends. It matters because it improves server civility automatically. I chose it because the expected behavior is clearly defined and it's a self-contained Python feature matching my skills.

---

## Understanding the Issue

### Problem Description

Duckbot should move users in a new role that prevents them from posting if they post in all caps.

### Expected Behavior

Move people into a new role that disallows them from posting if they post in all caps.

### Current Behavior

No action is taken

### Affected Components

```duckbot/cogs/corrections/```

---

## Reproduction Process

### Environment Setup

I attempted to set up local development environment on macOS Tahoe 26.2 (beta), but encountered incompatibility — Python 3.13 and 3.11 both fail with ImportError: Symbol not found: _XML_SetAllocTrackerActivationThreshold due to a libexpat version mismatch between Homebrew Python bottles and macOS Tahoe's system libraries. This is a known issue with pre-release macOS. I worked around this by exploring the codebase directly on GitHub.

### Steps to Reproduce

1. Navigate to ```duckbot/cogs/corrections/``` in the repository
2. Review all existing files: ```bezos.py```, ```bitcoin.py```, ```kubernetes.py```, ```tarlson.py```, ```typos.py```
3. Confirm that no file detects and address all-caps messages or apply role-based timeouts

### Reproduction Evidence
As this issue is a feature request and not a bug, it does not exist anywhere in the codebase. 

- **Issue branch:** [https://github.com/sujindl/duckbot/tree/issue-78](https://github.com/sujindl/duckbot/tree/issue-78)
- **My findings:** The closest pattern to our issue is ```typos.py``` which uses on_message to detect message content and reply. The timeout feature would follow this same pattern.

---

## Solution Approach

### Analysis

This is a feature request, not a bug. The root cause of the missing behavior is that no cog exists to monitor message content for excessive capitalization. The corrections cog directory contains similar behavior-correction features but none address all-caps messages.

### Proposed Solution

Create a new cog file ```duckbot/cogs/corrections/timeout.py``` that listens to all incoming messages, detects when a message is predominantly uppercase, temporarily assigns a restricted role to the user for 5 minutes, notifies the channel, and welcomes the user back when the timeout expires.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Duckbot has no mechanism to detect all-caps messages or temporarily restrict users who "yell" repeatedly.

**Match:** Match: ```duckbot/cogs/corrections/typos.py``` uses ```@commands.Cog.listener("on_message")``` to detect message content and respond. This can be adapted to our solution to detect all-caps.

**Plan:** 
1. Create ```duckbot/cogs/corrections/timeout.py``` following the same structure as ```typos.py```
2. Add ```on_message``` listener that detects if a message is mostly all-caps
3. Assign a restricted Discord role to the user for 5 minutes using ```asyncio.sleep(300)```
4. Send timeout message to the channel
5. Remove the restricted role after 5 minutes and send welcome-back message
6. Add tests in ```tests/cogs/corrections/```

**Implement:** [https://github.com/sujindl/duckbot/tree/issue-78](https://github.com/sujindl/duckbot/tree/issue-78)

**Review:** Will follow project's Black formatting style and existing cog structure per CLAUDE.md.

**Evaluate:** Write unit tests mocking Discord message objects with all-caps content and verify role assignment and removal logic

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
