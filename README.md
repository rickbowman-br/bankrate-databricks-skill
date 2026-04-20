# Bankrate Databricks Analytics Skill

A collaborative Claude Code skill for querying and analyzing Bankrate's Databricks data warehouse.

## Installation

### Option 1: Clone into Claude Skills Directory

```bash
# Clone into your Claude skills directory
git clone <this-repo-url> ~/.claude/skills/bankrate-databricks

# Or create a symlink
ln -s /path/to/this/repo ~/.claude/skills/bankrate-databricks
```

### Option 2: Use as a Team Skill

If your organization has Claude Code team configuration:

```bash
# In your team's .claude/skills/ directory
git clone <this-repo-url>
```

## Usage

Once installed, Claude will automatically use this skill when you ask Databricks-related questions:

```
"Show me the top 10 pages from last week"
"What's our revenue by vertical?"
"Analyze the Cobrand Purchase funnel"
"What's the continuation rate for our landing pages?"
```

## Updating the Skill

### Pull Latest Changes

```bash
cd ~/.claude/skills/bankrate-databricks
git pull
```

### Make Changes

1. Edit `skill.md` to add new query patterns or improve existing ones
2. Test your changes with Claude Code
3. Commit and push:

```bash
git add skill.md
git commit -m "Add query pattern for X"
git push
```

4. Notify team to pull updates

## Skill Structure

```
bankrate-databricks/
├── skill.md          # Main skill definition (queries, patterns, instructions)
├── README.md         # This file
└── CHANGELOG.md      # Version history
```

## Contributing

Everyone on the team can contribute! Common improvements:

- **Add new query patterns** - Found a useful query? Add it to skill.md
- **Improve formatting** - Make output more readable
- **Add visualizations** - Create HTML charts for complex data
- **Optimize queries** - Speed up slow queries
- **Fix bugs** - Correct errors in SQL or logic
- **Update docs** - Keep table schemas current

### Contribution Workflow

1. **Pull latest**: `git pull` to get latest changes
2. **Make changes**: Edit `skill.md` with your improvements
3. **Test**: Use Claude Code to verify your changes work
4. **Commit**: `git commit -m "Descriptive message"`
5. **Push**: `git push origin main`
6. **Notify**: Let team know in Slack/Teams about the update

## Version History

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.

## Support

- **Documentation**: See the main [databricks repo](../databricks) for full table schemas
- **Questions**: Ask in #analytics-team Slack channel
- **Issues**: Open a GitHub issue for bugs or feature requests

## Related Resources

- [Databricks CLI Docs](https://docs.databricks.com/dev-tools/cli/index.html)
- [Full Table Documentation](../databricks/tables/)
- [Query Examples](../databricks/queries/)
