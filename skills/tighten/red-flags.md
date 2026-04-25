# Red Flags Checklist

Load this file when performing detailed analysis.

## Abstraction Smells

- [ ] Interface with single implementation
- [ ] Base class with one subclass
- [ ] Registry for fixed set of items
- [ ] Event system with no subscribers
- [ ] Configuration for single-option feature
- [ ] Mode switch that isn't used
- [ ] Strategy pattern with one strategy
- [ ] Factory that builds one type

## Over-Engineering Smells

- [ ] More than 3 constructor/function parameters
- [ ] "Handler", "Manager", "Registry", "Factory" class without justification
- [ ] Retryable/Cacheable/CircuitBreaker on day one
- [ ] Checkpointing without restart crashes in history
- [ ] Approval system without compliance requirement
- [ ] Feature flags for features that shipped
- [ ] Backwards-compatibility shims for internal code

## AI Slop Smells

- [ ] Comments restating what code does ("This function gets the user")
- [ ] Entry/exit logging (`logger.debug "Entering method X"`)
- [ ] Try-catch that only logs and re-throws
- [ ] Redundant null/nil checks in typed contexts
- [ ] `async` function with no `await`
- [ ] Unused imports from refactoring
- [ ] Defensive programming for errors never seen
- [ ] Empty rescue/catch blocks
- [ ] `rescue => e; raise e` pattern
- [ ] Comments like `# TODO: implement` left in final code
- [ ] Overly verbose variable names (`userAccountInformationData`)

## Ruby/Rails Specific

- [ ] `before_action` for single-use setup
- [ ] Concern with one includer
- [ ] Service object called from one place
- [ ] Serializer for internal-only data
- [ ] Form object wrapping single model with no extra logic
- [ ] Decorator/Presenter for trivial formatting
- [ ] Callback that should be explicit method call
- [ ] `delegate` for single method
- [ ] Module that's never mixed in elsewhere

## JavaScript/React Specific

- [ ] Custom hook used in one component
- [ ] Context provider with single consumer
- [ ] HOC wrapping one component
- [ ] Utility function called once
- [ ] Type definition used in one place
- [ ] `useCallback`/`useMemo` without measured perf issue
- [ ] Prop drilling "fix" for 2-level depth

## Test Smells

- [ ] Mocks for code that doesn't exist yet
- [ ] Testing implementation instead of behavior
- [ ] Test setup longer than test itself
- [ ] Shared examples with single use
- [ ] `let!` that should be inline
- [ ] Factory with unused traits

## Quick Severity Guide

| Severity     | Action                                                 |
| ------------ | ------------------------------------------------------ |
| **Critical** | Single-impl abstractions, >3 params, unused code paths |
| **High**     | AI slop (comments, log-rethrow), defensive programming |
| **Medium**   | Over-configuration, premature optimization             |
| **Low**      | Style inconsistencies, verbose naming                  |
