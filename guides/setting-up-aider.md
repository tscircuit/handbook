# Setting Up Aider

[aider](https://aider.chat) is a command-line tool that can complete entire github issues for you.

Aider can be an incredible tool for getting work done. Ask @seveibar for an API key then put the
following in your `.zshrc`

```bash
export ANTHROPIC_API_KEY="sk-ant-api03...."
export AIDER_SONNET=1
alias aid="aider --no-auto-commit --no-auto-lint"
```
