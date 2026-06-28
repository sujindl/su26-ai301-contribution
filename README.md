# Contribution #1: [Give people a timeout for yelling too much]

**Contribution Number:** 1  
**Student:** Sujin Lee  
**Issue:** [duckbot #78](https://github.com/duck-dynasty/duckbot/issues/78)  
**Status:** **Phase IV Complete** 

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

I attempted to set up local development environment on macOS Tahoe 26.2 (beta), but encountered incompatibility — Python 3.13 and 3.11 both fail with ImportError: Symbol not found: _XML_SetAllocTrackerActivationThreshold due to a libexpat version mismatch between Homebrew Python bottles and macOS Tahoe's system libraries. This is a known issue with pre-release macOS. I worked around this by using a GitHub Codespace instead, where I installed Python 3.13 via the deadsnakes PPA, resolved a missing python3.13-dev and libpq-dev dependency for psycopg2, and successfully ran tests.

### Steps to Reproduce

1. Navigate to ```duckbot/cogs/corrections/``` in the repository
2. Review all existing files: ```bezos.py```, ```bitcoin.py```, ```kubernetes.py```, ```tarlson.py```, ```typos.py```
3. Confirm that no file detects and address all-caps messages or apply role-based timeouts

### Reproduction Evidence
Searched the entire codebase using grep ```-r "caps"```, ```grep -r "upper"```, and ```grep -r "timeout"```. No all-caps detection or role-based timeout mechanism exists anywhere in ```duckbot/cogs/```. Confirmed the feature is completely missing and needs to be built from scratch.

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

- [x] Test case 1: Bot messages are ignored and never trigger a timeout
- [x] Test case 2: Short messages under 5 alphabetic characters are ignored
- [x] Test case 3: Lowercase messages do not trigger a timeout
- [x] Test case 4: All-caps messages from a server member trigger a reply and timeout
- [x] Test case 5: All-caps messages in DMs do not trigger a timeout (discord.User has no timeout method)
- [x] Test case 6: is_yelling() returns True for all-caps messages
- [x] Test case 7: is_yelling() returns False for lowercase messages
- [x] Test case 8: is_yelling() returns False for short messages

### Integration Tests

Not applicable for this feature — the cog integrates directly with Discord's API which requires a live server to test end-to-end.

### Manual Testing

Could not manually test in a live Discord server as the project is a private friend group bot without access credentials. Verified behavior through unit tests with mocked Discord objects. All 2971 tests pass with 31 skipped.

---

## Implementation Notes

### Week 3 Progress

Created ```duckbot/cogs/corrections/timeout.py``` — a new Discord cog that listens for all-caps messages using on_message, detects yelling via an is_yelling() helper that checks if 80%+ of alphabetic characters are uppercase, applies Discord's native member timeout for 5 minutes, and sends a welcome-back message after the timeout expires. Registered the cog in duckbot/cogs/corrections/__init__.py. Used isinstance(message.author, discord.Member) to skip DMs where timeouts aren't possible.

### Code Changes

- **Files modified:** ```duckbot/cogs/corrections/timeout.py```
- **Key commits:**
  - https://github.com/sujindl/duckbot/pull/1/changes/601d1b3c1db9d0666fdeb162a855e76cf7a43b0c
  - https://github.com/sujindl/duckbot/pull/1/changes/16a80e6b075287f5da01942d003aa7fc22e07c07
- **Approach decisions:** Used Discord's native ```member.timeout()``` method instead of manually assigning a restricted role, since the codebase had no existing role management patterns and the native timeout is simpler and more reliable. Used isinstance(message.author, discord.Member) check to skip DMs where timeouts aren't supported. Set yelling threshold at 80% uppercase letters to avoid false positives from messages with abbreviations or acronyms. Ignored messages with fewer than 5 alphabetic characters to avoid timing out short messages like "OK" or "Hi".

---

## Pull Request

**PR Link:** https://github.com/duck-dynasty/duckbot/pull/1355

**PR Description:** 

Summary: Adds a timeout feature for users who yell (post in all caps) in the server. When a user sends a message that is 80%+ uppercase letters, DuckBot replies with a timeout warning, applies Discord's native 5-minute member timeout, and sends a welcome-back message when the timeout expires. Fixes #78.

Testing: ran format and pytest (2971 passed, 31 skipped).

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** Awaiting review
---

## Learnings & Reflections

### Technical Skills Gained

Learned how to build a Discord.py cog from scratch using event listeners, specifically on_message. Gained experience with Discord's native timeout API and how to distinguish between discord.Member (server) and discord.User (DM) objects. Learned how to mock asyncio.sleep in tests to prevent them from hanging, and how to structure unit tests for async Discord bot code using pytest-asyncio.

### Challenges Overcome

The biggest challenge was environment setup — macOS Tahoe 26.2 beta has a libexpat incompatibility that prevented Python 3.13 from installing pip, blocking local development entirely. Solved this by switching to GitHub Codespaces and installing Python 3.13 via the deadsnakes PPA. Also encountered tests hanging for 5 minutes due to the real asyncio.sleep being called — fixed by mocking it at the correct module path duckbot.cogs.corrections.timeout.asyncio.sleep.

### What I'd Do Differently Next Time

I'd commit more incrementally rather than making several changes before committing.

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
