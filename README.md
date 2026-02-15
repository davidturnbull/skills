# Skills

## Introduction

[Agent Skills](https://agentskills.io/what-are-skills) are a lightweight, open format for extending AI agent capabilities with specialized knowledge and workflows. A skill is a folder containing a `SKILL.md` file with instructions that tell an agent how to perform a specific task.

## Installation

Install a skill using the [`skills` CLI](https://skills.sh/docs/cli):

```sh
npx skills add davidturnbull/skills/<skill-name>
```

## Available skills

### commit

Group working tree changes into atomic commits with well-crafted messages. Reads diffs, splits unrelated changes into separate commits, orders them for bisectability, and drafts conventional commit messages. Presents a numbered plan you can selectively approve, edit, or commit all at once. Flags secrets and large binaries before committing.

```sh
npx skills add davidturnbull/skills/commit
```

## License

[MIT](LICENSE)
