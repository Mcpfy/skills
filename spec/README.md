# Agent Skills Specification

The official Agent Skills specification is maintained at [agentskills.io](https://agentskills.io).

## SKILL.md Format

Each skill is defined by a `SKILL.md` file with the following structure:

### YAML Frontmatter

Every skill must include YAML frontmatter at the beginning of the file:

```yaml
---
name: skill-name
description: Clear description of what the skill does and when to use it
---
```

**Required Fields:**
- `name`: Lowercase alphanumeric with hyphens, max 64 characters
- `description`: Explains what the skill does and when to use it, max 1024 characters

**Optional Fields:**
- `license`: License identifier (e.g., Apache-2.0, MIT)
- `allowed-tools`: List of specific tools this skill can use
- `metadata`: Additional metadata as key-value pairs

### Markdown Body

After the frontmatter, the skill content is written in standard Markdown:

- Clear instructions for Claude to follow
- Code examples and patterns
- Best practices and guidelines
- Links to documentation and resources

## Creating a Skill

1. Fill in the frontmatter with appropriate values
2. Write clear, actionable instructions in the body
3. Include examples and best practices
4. Test the skill with Claude to ensure it works as expected

## Learn More

- **Official Spec**: [agentskills.io](https://agentskills.io)
- **Anthropic's Skills Guide**: [Creating Custom Skills](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- **Skills Platform**: [skills.sh](https://skills.sh)
