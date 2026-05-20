# Agent Instructions for Test-Case Authoring

Use these instructions when asking an agent to draft test cases.

## Goal

Create clear, actionable test cases from requirements, bugs, or user stories.

## Requirements for Every Test Case

- Use the repository template at `/templates/test-case-template.md`.
- Keep wording specific and measurable.
- Include at least:
  - Preconditions
  - Test steps
  - Expected results
  - Negative or edge-case considerations (when applicable)
- Map each test to a requirement, issue, or acceptance criterion.
- Call out assumptions and missing information explicitly.

## Suggested Prompt for Agents

> Draft test cases for the requirement below using
> `/templates/test-case-template.md`.  
> Keep each test independent, include positive and negative coverage, and link
> each test back to a specific requirement.  
> If details are missing, list assumptions at the end.

## Quality Checklist

- Are steps reproducible without tribal knowledge?
- Are expected results objective (not ambiguous)?
- Does coverage include core flow + edge cases?
- Is each test scoped to one behavior?
