# Workshop Issues and Problems Report

This document details all issues, problems, and confusing portions found when navigating the Temporal + OpenAI Agents SDK Workshop as a student.

## 🔴 Critical Issues (Blockers)

### 1. Test Suite Completely Broken
**Location:** `tests/` directory  
**Impact:** HIGH - Tests cannot run at all

**Problem:**
- Tests try to import from directories starting with digits (e.g., `solutions.02_temporal_hello_world`)
- Python module names cannot start with digits, causing `SyntaxError: invalid decimal literal`
- Tests reference `.py` files that don't exist (e.g., `solutions.ex01_agent_hello_world.main`)
- Solutions are Jupyter notebooks (`.ipynb`), not Python modules

**Evidence:**
```bash
$ make test
ERROR tests/test_exercise_01.py - ModuleNotFoundError: No module named 'solutions.ex01_agent_hello_world'
ERROR tests/test_exercise_02.py - SyntaxError: invalid decimal literal
ERROR tests/test_exercise_03.py - SyntaxError: invalid decimal literal
```

**Impact on Students:**
- Students cannot run `make test` to verify their work
- CI/CD pipeline would fail
- No way to validate solutions programmatically

**Recommended Fix:**
- Either rename solution directories to valid Python module names (e.g., `ex01_agent_hello_world`)
- OR extract testable functions from notebooks into separate `.py` files
- OR remove the tests entirely if they're not meant to be run

---

### 2. Temporal Installation Notebook Hangs
**Location:** `temporal_installation.ipynb`  
**Impact:** HIGH - Blocks students from progressing

**Problem:**
The second cell runs `!~/.temporalio/bin/temporal server start-dev`, which is a long-running server process. In a Jupyter notebook, this will:
- Block the cell execution indefinitely
- Prevent students from running any subsequent cells
- Require students to interrupt the kernel, losing all state
- Not provide clear guidance on running the server in background

**Evidence:**
```python
# Cell 2 in temporal_installation.ipynb
!~/.temporalio/bin/temporal server start-dev  # This blocks forever!
```

**Impact on Students:**
- Students follow README instructions to use this notebook
- Notebook hangs on the second cell
- Students don't know if it's working or broken
- No clear path forward (should they interrupt? open a new terminal?)

**Recommended Fix:**
- Add explicit instructions to run Temporal in a separate terminal using `make temporal-up`
- OR modify the notebook to start Temporal in background with proper output capture
- OR remove the notebook and direct students to `make temporal-up` exclusively
- Add a "Verify Installation" cell that checks if Temporal is running

---

### 3. Exercise 4 API Mismatch Between README, Exercise, and Solution
**Location:** `exercises/04_agent_routing/`  
**Impact:** HIGH - Students will be unable to complete exercise

**Problem:**
The README instructions, exercise template, and solution use completely different APIs:

**README Instructions say:**
```python
new_message: ChatCompletionMessageParam = {"role": "user", "content": user_query}
result = await Runner.run(triage_agent(), new_message)
return result.final_output
```

**Exercise Template imports:**
```python
from agents import Agent, Runner, function_tool
from openai.types.chat import ChatCompletionMessageParam
```

**Solution Actually Uses:**
```python
from agents import Agent, RunConfig, Runner, TResponseInputItem, trace
config = RunConfig()
with trace("Routing example"):
    inputs: list[TResponseInputItem] = [{"content": msg, "role": "user"}]
    result = await Runner.run(triage_agent(), input=inputs, run_config=config)
```

**Impact on Students:**
- Following README instructions produces code incompatible with solution
- Imports in exercise don't match what solution needs
- Students will get type errors or runtime errors
- No clear path to correct implementation

**Recommended Fix:**
- Align all three (README, exercise template, solution) to use the same API
- Decide on one approach (preferably the simpler one from README)
- Update all TODOs and hints to match the chosen API

---

### 4. Broken Codespaces Link in README
**Location:** `README.md` line 3  
**Impact:** MEDIUM - Students cannot launch workshop via badge

**Problem:**
```markdown
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/nadvolod/temporal-openai-agents-sdk)
```

This links to `nadvolod/temporal-openai-agents-sdk` but the repository is actually `temporal-community/edu-ai-workshop-openai-agents-sdk`.

**Impact on Students:**
- Badge doesn't work (404 or wrong repo)
- Students using the badge get wrong/missing content
- Primary workshop entry point is broken

**Recommended Fix:**
```markdown
[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/temporal-community/edu-ai-workshop-openai-agents-sdk)
```

---

## 🟡 Major Issues (Confusing/Problematic)

### 5. Bootstrap Script References Non-existent Make Commands
**Location:** `scripts/bootstrap.sh` lines 36-38  
**Impact:** MEDIUM - Misleading instructions after setup

**Problem:**
```bash
echo "Next steps:"
echo "  2. Run 'make temporal-up' to start Temporal server"
echo "  3. Run 'make exercise-1' to start the workshop"
```

The command `make exercise-1` doesn't exist in the Makefile. Exercises are Jupyter notebooks, not make targets.

**Impact on Students:**
- Students run `make exercise-1` and get "No rule to make target" error
- Confusion about how to actually start exercises
- Bootstrap claims to provide "next steps" but gives broken commands

**Recommended Fix:**
```bash
echo "Next steps:"
echo "  1. Add your OPENAI_API_KEY to .env file"
echo "  2. Open and run temporal_installation.ipynb to start Temporal"
echo "  3. Open exercises/01_agent_hello_world/exercise.ipynb in VS Code"
```

---

### 6. Undocumented `example-notebook` Directory
**Location:** `example-notebook/` directory  
**Impact:** MEDIUM - Confusion about workshop structure

**Problem:**
- README doesn't mention `example-notebook/` directory
- WORKSHOP_SPEC.md doesn't include it in structure
- Contains notebooks `02-Adding-Durability-Solution.ipynb` and `03-Human-in-the-Loop-Solution.ipynb`
- Unclear if these are for students, instructors, or deprecated content
- Different naming convention than exercises (numbers vs descriptive names)

**Impact on Students:**
- Students see extra notebooks and wonder if they should use them
- Unclear relationship to exercises 1-4
- Naming suggests they might be newer/better than official exercises
- Creates confusion about which notebooks to follow

**Recommended Fix:**
- Document the purpose of `example-notebook/` in README
- OR move to a separate directory like `extras/` or `demos/`
- OR remove if deprecated/not needed for the workshop

---

### 7. Exercise 4 Starter.py Uses Wrong Workflow Execution Method
**Location:** `exercises/04_agent_routing/starter.py`  
**Impact:** MEDIUM - Students get wrong guidance

**Problem:**
Exercise TODOs suggest:
```python
handle = await client.start_workflow(...)
result = await handle.result()
```

Solution actually uses:
```python
result = await client.execute_workflow(...)
```

**Impact on Students:**
- `execute_workflow()` combines both operations
- Following TODOs produces different (but also valid) code
- Students may be confused about which approach is correct
- Inconsistency between exercise guidance and solution

**Recommended Fix:**
- Choose one approach consistently
- Update either exercise TODOs or solution to match
- Document why one approach is preferred over the other

---

## 🟢 Minor Issues (Polish/Improvements)

### 8. Exercise 3 Variable Naming Inconsistency
**Location:** `exercises/03_durable_agent/exercise.ipynb`  
**Impact:** LOW - Minor confusion

**Problem:**
- Solution names activity function `get_weather`
- Exercise TODO says to use `@activity.defn(name="get_weather")`
- But then references `user_query` as parameter name
- Some inconsistency in parameter naming between components

**Recommended Fix:**
- Standardize naming conventions across all exercises
- Ensure parameter names match between exercise and solution

---

### 9. Makefile Comment Inconsistency
**Location:** `Makefile` lines 28-34  
**Impact:** LOW - Outdated comments

**Problem:**
Comments say "Note: Exercises 1-3 are Jupyter notebooks" but Exercise 4 is also listed among those comments even though it uses Python files.

**Recommended Fix:**
Update comments to clearly distinguish:
```makefile
# Exercises 1-3 are Jupyter notebooks (.ipynb files):
#   exercises/01_agent_hello_world/exercise.ipynb
#   exercises/02_temporal_hello_world/exercise.ipynb
#   exercises/03_durable_agent/exercise.ipynb
#
# Exercise 4 uses separate Python files (production pattern):
#   exercises/04_agent_routing/workflow.py
#   exercises/04_agent_routing/worker.py
#   exercises/04_agent_routing/starter.py
```

---

### 10. Missing Guidance on Temporal UI Access in Codespaces
**Location:** README troubleshooting section  
**Impact:** LOW - Students may struggle to access UI

**Problem:**
README mentions "In Codespaces: The port should be automatically forwarded" but doesn't explain:
- How to find the forwarded port URL
- That they need to click "Open in Browser" from Ports tab
- What to do if port forwarding doesn't happen automatically

**Recommended Fix:**
Add explicit Codespaces instructions:
```markdown
**In GitHub Codespaces:**
1. Click on the "PORTS" tab in the bottom panel
2. Find port 8233 (Temporal UI)
3. Click the globe icon to open in browser
4. If port 8233 isn't listed, ensure Temporal server is running
```

---

## 📊 Summary Statistics

- **Total Issues Found:** 10
- **Critical (Blockers):** 4
- **Major (Confusing):** 3
- **Minor (Polish):** 3

## 🎯 Priority Recommendations

### Must Fix Before Workshop:
1. Fix or remove broken test suite (Issue #1)
2. Fix temporal_installation.ipynb blocking issue (Issue #2)
3. Align Exercise 4 API across README/exercise/solution (Issue #3)
4. Fix Codespaces badge link (Issue #4)

### Should Fix for Better Experience:
5. Update bootstrap.sh next steps (Issue #5)
6. Document or remove example-notebook (Issue #6)
7. Align Exercise 4 starter approach (Issue #7)

### Nice to Have:
8-10. Polish issues (documentation, naming, comments)

---

## 🧪 Testing Done

To identify these issues, I:
1. ✅ Ran `make setup` successfully
2. ✅ Ran `make test` (failed with syntax errors)
3. ✅ Examined all 4 exercise notebooks and their solutions
4. ✅ Compared exercise templates with solutions
5. ✅ Checked README instructions vs actual code
6. ✅ Reviewed all configuration files (Makefile, pyproject.toml, devcontainer.json)
7. ✅ Traced the student journey from README through each exercise

## 📝 Student Perspective Notes

**Most Confusing Aspects:**
1. Test failures immediately after setup create doubt about installation
2. Temporal installation notebook hangs without explanation
3. Exercise 4 has three different "correct" approaches
4. Unclear relationship between examples and exercises

**Positive Aspects:**
1. README is comprehensive and well-structured
2. Exercises 1-3 follow a logical progression
3. Solution notebooks are well-commented
4. Architecture diagrams are helpful

**Overall:** The workshop has excellent educational content and structure, but several critical bugs prevent students from successfully completing it. Fixing issues #1-4 would make it immediately usable.
