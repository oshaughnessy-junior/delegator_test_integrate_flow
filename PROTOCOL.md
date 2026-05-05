# Delegator Task Board Protocol

## Overview
This document defines the exact workflow that any `coding-agent‑unsafe` (or any other agent) must follow when interacting with the shared task board located in this repository. By keeping the protocol version‑controlled alongside the board, every new machine that clones the repo automatically receives the latest rules.

---

## 1. Board‑access Contract (Steps A‑H)

| Step | Action |
|------|--------|
| **A** | **Pull the latest repository** – `git pull` (or `gh repo clone` on first run). |
| **B** | **Read the task list** – open `board/tasks.json` (or whichever file the board uses) and parse the JSON array of tasks. |
| **C** | **Select a task** – Choose the first entry whose `status` is `"open"` and whose `owner` field is empty. |
| **D** | **Claim the task atomically** –
> 1. Edit the JSON to set `owner` to the current machine’s identifier (`HOSTNAME` or a UUID) and `status` to `"in‑progress"`.
> 2. Commit the change with a clear message, e.g. `"Claim task <id> on <machine-id>"`.
> 3. Push the commit; the first push wins, so if another agent pushes first the claim will be rejected and the agent must restart at **Step B**. |
| **E** | **Execute the task** – Run the code, build, test, or any other work described by the task. |
| **F** | **Write artefacts** – Place generated source files, patches, tests, and logs in the shared output location (see below). |
| **G** | **Mark the task done** – Update the task entry: `status: "done"`, add an optional `result` field with a short description, commit, and push. |
| **H** | **Post a summary comment** – Use the GitHub CLI (`gh issue comment <ISSUE>`) to post a brief note linking to the artefacts (e.g. `output/code/<file>`). |

---

## 2. Shared Output Location

All artefacts are stored **inside this repository** under a top‑level `outputs/` directory so that every agent can fetch them with a normal `git pull`.

```
outputs/
├─ code/   # generated source files or patches
├─ tests/  # test scripts, data, or results
└─ logs/   # execution logs, benchmark tables, etc.
```

### Bootstrap (run once per machine)
1. Create the directories if they do not exist:
   ```bash
   mkdir -p outputs/{code,tests,logs}
   ```
2. Record this machine’s identifier:
   ```bash
   echo "$HOSTNAME" > outputs/MACHINE_ID.txt
   ```
3. Commit the new directories (empty placeholder `.gitkeep` files are allowed) and push.

---

## 3. Conflict Resolution
- **Claim race:** The first agent to push its claim wins. If your push is rejected, simply re‑run the board check from **Step B**.
- **Output collisions:** If two agents try to write the same file, prepend the filename with the machine ID (e.g., `machine1_mycode.py`).

---

## 4. Heartbeat Integration
Add the following entry to `HEARTBEAT.md` (run every 30 min):

```
## Distributed Task‑Board Check (Every 30 min)
- [ ] **Check Delegator Board:**
  - Pull repository
  - Run through Steps A‑H above
```

---

## 5. Extending the Protocol
Any future changes (new fields, additional artefact types, etc.) should be made via a normal pull request to this repo so that all agents automatically adopt the updated rules after their next `git pull`.

---

*Version: 2026‑05‑05*