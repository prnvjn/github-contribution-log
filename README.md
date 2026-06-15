# github-contribution-log

# Contribution [Report static error when prop/attr_reader method name is invalid #6315]: 

**Contribution Number:** [1]  
**Student:** [Pranav]  
**Issue:** [[GitHub issue link](https://github.com/sorbet/sorbet/issues/6315)]  
**Status:** [Phase I]

---

## Why I Chose This Issue
While I am comfortable with Ruby's internals, working on this issue allows me to dive into a production-grade C++ codebase. I hope to learn how Sorbet architectures its structural validation rules and how to properly implement checks that safeguard code quality regardless of a file's explicit strictness level.

---

## Understanding the Issue

### Problem Description

Sorbet is silent when prop/const (T::Props) or attr_reader receive an invalid method name — e.g. a symbol with a hyphen like :'foo-bar'. Ruby raises at runtime in this case.

### Expected Behavior

```
  # typed: false
  class A
    include T::Props
    const :'foo-bar', Integer      # should error: Bad attribute name `foo-bar`
    attr_reader :'left-right'       # should error: Bad attribute name `left-right`
  end
```
  Sorbet should emit a 3501 (BadAttrArg) error on both lines, regardless of # typed: level.

### Current Behavior
  - `attr_reader :'foo-bar'` at `# typed: true`: error is reported (validation exists).
  - `attr_reader :'foo-bar'` at `# typed: false`: no error — BadAttrArg is defined as StrictLevel::True, so it only fires
  in true/strict/strong files.
  - `const :'foo-bar'`, Integer at any typed level: no error — parseProp() in rewriter/Prop.cc extracts the name from
  the symbol but never calls any validation function.

### Affected Components

 - rewriter/Prop.cc  
 - rewriter/AttrReader.cc 

---

## Reproduction Process

### Environment Setup

Install the dependencies

`brew install bazel autoconf coreutils parallel`

Clone this repository

`git clone https://github.com/sorbet/sorbet.git`
`cd sorbet`

Build Sorbet

`./bazel build //main:sorbet --config=dbg`

### Steps to Reproduce
```
bazel-bin/main/sorbet -e "# typed: false

class A
  include T::Props

  const :'foo-bar', Integer

  attr_reader :'left-right'
end"
```

Observed result: No errors! Great job.

### Reproduction Evidence

<img width="662" height="625" alt="Screenshot 2026-06-15 at 1 33 01 AM" src="https://github.com/user-attachments/assets/e543e761-46b7-4f2b-a589-865d1d1c3628" />
---

## Solution Approach

### Analysis

Root cause 1 — parseProp skips validation (rewriter/Prop.cc:237–253)

Root cause 2 — wrong StrictLevel (core/errors/rewriter.h:6)

### Proposed Solution

Two targeted changes:
  
  1. Lower the error level: Change BadAttrArg from StrictLevel::True to StrictLevel::False. This fixes attr_reader at
  # typed: false.
  2. Add validation to parseProp: Expose validAttrName from its anonymous namespace in Util.cc as a public ASTUtil
  static method, then call it from parseProp after extracting the name. This fixes const/prop at all typed levels.



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
