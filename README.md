# github-contribution-log

# Contribution [Report static error when prop/attr_reader method name is invalid #6315]: 

**Contribution Number:** [1]  
**Student:** [Pranav]  
**Issue:** [[GitHub issue link](https://github.com/sorbet/sorbet/issues/6315)]  
**Status:** [Phase III]

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
  Sorbet should emit a 3501 (BadAttrArg) error on both lines, regardless of `# typed`: level.

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
  `# typed: false`.
  2. Add validation to parseProp: Expose validAttrName from its anonymous namespace in Util.cc as a public ASTUtil
  static method, then call it from parseProp after extracting the name. This fixes const/prop at all typed levels.



### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** prop/const never validates the symbol name passed as first argument. attr_reader validates it but only
  in `# typed: true files`. Both should error at any strictness level, matching Ruby's runtime behavior.

**Match:** The exact validation logic already exists in `validAttrName()` in `rewriter/util/Util.cc:331`. It checks:
  non-empty, first char is alpha or _, all chars are alphanumeric or _. The `DuplicateProp` check in `Prop.cc:654` shows
  the pattern for reporting an error from a `parseProp`-adjacent function: `ctx.beginIndexerError(loc, 
  core::errors::Rewriter::SomeError)`.


**Plan:** [Step-by-step implementation plan]
  1. core/errors/rewriter.h:6 — Change StrictLevel::True → StrictLevel::False for BadAttrArg:
  inline constexpr ErrorClass BadAttrArg{3501, StrictLevel::False};
  2. rewriter/util/Util.cc:329 — Move validAttrName from the anonymous namespace {} into the ASTUtil namespace so it
  can be called from other files. No logic changes.
  3. rewriter/util/Util.h — Add the public declaration alongside getAttrName:
  static bool validAttrName(core::MutableContext ctx, core::LocOffsets loc, core::NameRef name);
  4. rewriter/Prop.cc:245 — After ret.name = sym->asSymbol(), add the validation call:
  ret.name = sym->asSymbol();
  if (!ASTUtil::validAttrName(ctx, sym->loc, ret.name)) {
      return nullopt;
  }
  5. test/testdata/rewriter/attr_bad_string.rb — Add # typed: false cases with hyphens to confirm the attr_reader fix.
  6. test/testdata/rewriter/prop_bad_name.rb — New test file:
 ```
# typed: false
  class A
    include T::Props
    const :'foo-bar', Integer  # error: Bad attribute name `foo-bar`
    prop :'left-right', String # error: Bad attribute name `left-right`
  end
```


**Implement:** [Link to your branch/commits as you work](https://github.com/prnvjn/sorbet/tree/fix-issue-6315-invalid-prop-attr-name)

**Review:** 
  - [ ] Commit message matches project style (imperative, no period — e.g. Report error for invalid prop/attr_reader 
  method names)
  - [ ] No new error code needed — reusing BadAttrArg{3501}; check core/errors/rewriter.h to confirm no collision
  - [ ] Changes to error StrictLevel can introduce new errors in existing # typed: false files — flag this in the PR;
  Sorbet team will run a one-off check against Stripe's codebase per CONTRIBUTING.md
  - [ ] Announce intent in Sorbet's #internals Slack channel before opening the PR (per CONTRIBUTING.md)


## Testing Strategy
 Added two new test files under test/testdata/rewriter/:
- attr_bad_string_false.rb — verifies attr_reader/attr_writer/attr_accessor/attr with hyphenated symbol names produce BadAttrArg errors at # typed: false
- prop_bad_name.rb — verifies const :'foo-bar' and prop :'left-right' produce BadAttrArg errors at # typed: false

All three relevant test targets pass: attr_bad_string, attr_bad_string_false, prop_bad_name. Manually verified the original reproduction case now produces 2 errors instead of 0.

### Manual Testing
Ran the original reproduction case from the issue directly against the built binary:
 ```bazel-bin/main/sorbet -e "# typed: false
  class A
   include T::Props
   const :'foo-bar', Integer
   attr_reader :'left-right'
  end"
```
Before fix: 0 errors. 

After fix: 2 errors reported 

 - Bad attribute name \foo-bar`on theconst` line
 - Bad attribute name \left-right`on theattr_reader` line

Confirmed both errors fire at # typed: false, which was the core bug.

---

## Implementation Notes

Fixed in two stages. 
**First** lowered BadAttrArg (error 3501) from StrictLevel::True to StrictLevel::False in core/errors/rewriter.h, so attr_reader :'foo-bar' reports an error even in # typed: false files. 
**Second** exposed the existing validAttrName() function from its anonymous namespace as a public ASTUtil static method (rewriter/util/Util.h, rewriter/util/Util.cc),
then called it from parseProp in rewriter/Prop.cc after extracting the symbol name — returning nullopt on invalid input. This causes const :'foo-bar', Integer to report a BadAttrArg error at any typed level.
Files modified: `core/errors/rewriter.h`, `rewriter/util/Util.cc`, `rewriter/util/Util.h`, `rewriter/Prop.cc`

### Code Changes

 - Branch: https://github.com/prnvjn/sorbet/tree/fix-issue-6315-invalid-prop-attr-name
 - Commit 1 : https://github.com/prnvjn/sorbet/commit/066c3e1a7
 - Commit 2 : https://github.com/prnvjn/sorbet/commit/79fdd88ab



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
