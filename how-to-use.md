# How to Use the Agent Workflow

This repository implements a seven-agent workflow for safely changing brownfield/legacy code. The workflow is designed to follow Feathers' legacy code discipline: understand the change, identify test seams, make the code testable, characterize existing behavior with tests, implement the feature, refactor safely, and then finish with verification and documentation.

## Overview

The agents live in the `agents/` folder:

- `1-plan.agent.md`
- `2-map.agent.md`
- `3-break.agent.md`
- `4-cover.agent.md`
- `5-implement.agent.md`
- `6-refactor.agent.md`
- `7-finish.agent.md`

Each agent has a narrow responsibility and should be used in sequence.

## Installation

If you are using Github Copilot, just copy the `agents` directory to your `.github` directory under the repo root.

If you are using some other coding assistant, please consult the user docs of your coding agent to see where these files can go.

## General Usage

1. Start with a clear user request or change ticket.  Write it down in a file or somewhere else, preferably in markdown.
2. Use `1-plan.agent.md` and send a message with your requirement to the agent, in order to create a written plan in `plan/plan.md`.
3. Move to `2-map.agent.md` to identify test points, seams, and obstacles.
4. Use `3-break.agent.md` to break dependencies that currently prevent testing.
5. Use `4-cover.agent.md` to write characterization tests for existing behavior.
6. Use `5-implement.agent.md` to add the new functionality.
7. Use `6-refactor.agent.md` to clean up structure while keeping tests green.
8. Finish with `7-finish.agent.md` to verify, document, and finalize the change.

> The important rule is: do not skip steps. Each agent exists to enforce one part of the workflow.