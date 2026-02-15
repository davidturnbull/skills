# Skills

## Introduction

[Agent Skills](https://agentskills.io/what-are-skills) are a lightweight, open format for extending AI agent capabilities with specialized knowledge and workflows. This repo is for sharing skills that I've consistently found to work (very) well.

## Installation

Install a skill using the [`skills` CLI](https://skills.sh/docs/cli):

```sh
npx skills add davidturnbull/skills/<skill-name>
```

## Available skills

### commit

```sh
npx skills add davidturnbull/skills/commit
```

Group working tree changes into atomic commits with well-crafted messages:

- Drafts conventional commit messages.
- Splits unrelated changes into separate commits and orders them for bisectability.
- Presents a numbered plan you can selectively approve, edit, or commit all at once.

## License

[MIT](LICENSE)
