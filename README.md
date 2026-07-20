# Contribution [#3102]: Multiple API endpoints for courseDetails



**Contribution Number:**  [2]   
**Student:** Pranav Jain   
**Issue:** [GitHub issue link](https://github.com/openedx/frontend-app-authoring/issues/3102)  
**Status:** [Phase I] [completed]

---

## Why I Chose This Issue

The frontend-app-authoring codebase currently has three separate API endpoints that all fetch essentially the same `course details` data: one in `course-outline/data/api.ts`, one in the top-level `data/api.ts`, and one in `schedule-and-details/data/api.ts`. 
This duplication means the same information is fetched and maintained in multiple places, which increases the risk of inconsistencies and makes the codebase harder to maintain as changes need to be replicated across all three locations. I chose this issue because it's a well-scoped refactoring task that will help me learn how this frontend app's data layer is structured, while also giving me practice identifying and safely consolidating duplicated logic in a real production codebase.

---

## Understanding the Issue

### Problem Description

The course details functionality is implemented redundantly across the codebase. Instead of one shared piece of logic for fetching/updating course details, there are (at least) three separate API modules that each define similar or identical logic, leading to code duplication and inconsistency risk.

### Expected Behavior
There should be a single, shared API module (e.g. a common getCourseDetails / updateCourseDetails) that all features needing course details import and reuse, rather than each feature maintaining its own copy of the same logic.

### Current Behavior
Course details logic is duplicated across three separate files:

`src/course-outline/data/api.ts` (line 22)
`src/data/api.ts` (line 5)
`src/schedule-and-details/data/api.ts` (line 6)

Per bradenmacdonald's clarification, the course-outline and schedule-and-details versions are effectively the same API and are true duplicates, while the one in src/data/api.ts is a genuinely different endpoint that just serves a similar role — so it may not be a straightforward one-to-one consolidation.

### Affected Components

`course-outline` feature (`src/course-outline/data/api.ts`)
`schedule-and-details` feature (`src/schedule-and-details/data/api.ts`)
Shared/global data layer (`src/data/api.ts`)
No established convention yet exists in the repo for where shared cross-feature API code should live, so this fix will also involve a small architectural decision (module placement) alongside the actual dedup work.

Related/overlapping work: PR #3136 by taimoor-ahmed-1, which touches the course details API but is believed to be focused on the backend endpoint rather than the Authoring frontend code 

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

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
