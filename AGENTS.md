# Agent guidance for authoring C# Snoopynet tests

Use this repository as shared context for agents that help draft, review, or maintain C# tests for Snoopynet.

## Test authoring principles

- Prefer clear, deterministic tests over broad end-to-end coverage when a smaller test can prove the behavior.
- Name tests after the behavior being verified, including the condition and expected result.
- Keep setup, action, and assertion phases easy to identify.
- Avoid sleeps, timing assumptions, shared mutable state, and dependencies on test execution order.
- Use explicit test data that documents the scenario being tested.
- Assert observable behavior rather than private implementation details.
- Keep each test focused on one behavior; add separate tests for distinct edge cases.

## C# conventions

- Follow the existing Snoopynet test project's framework, naming style, namespaces, helpers, and fixture patterns.
- Use `async Task` tests for asynchronous code and await all asynchronous operations.
- Prefer strongly typed objects and constants over stringly typed values when existing APIs support them.
- Use nullable annotations and null checks consistently with the surrounding code.
- Keep helper methods local to the test class unless they are broadly reusable.
- Avoid adding new dependencies unless the current test framework cannot reasonably express the scenario.

## Snoopynet-specific focus areas

When helping author tests, look for coverage around:

- successful request and response handling
- authentication, authorization, and permission boundaries
- invalid input, missing data, and malformed payloads
- error handling and exception translation
- edge cases for empty collections, null values, duplicate values, and boundary values
- persistence, cleanup, and isolation between tests
- concurrency, retries, timeouts, and cancellation when relevant

## Suggested test structure

For each proposed test, capture:

1. the behavior under test
2. the preconditions or fixture setup
3. the action being performed
4. the expected result and assertions
5. any cleanup or isolation requirements

## Review checklist

Before considering a generated test complete, verify that it:

- compiles against the current solution
- follows existing test project conventions
- has deterministic assertions
- does not require external services unless the surrounding tests already do
- cleans up data it creates or uses isolated fixtures
- covers both happy path and meaningful failure or edge cases when appropriate
- avoids weakening or deleting existing tests
