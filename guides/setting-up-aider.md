# Setting Up Aider

[aider](https://aider.chat) is a command-line tool that can complete entire github issues for you.

Aider can be an incredible tool for getting work done. Ask @seveibar for an API key then put the
following in your `.zshrc`

```bash
# FOR OPENAI USERS
export OPENAI_API_KEY="sk-openai-..."


# FOR ANTHROPIC USERS (legacy- use o3-mini!)
export ANTHROPIC_API_KEY="sk-ant-api03...."
export AIDER_SONNET=1


alias aid="aider --no-auto-commit --no-auto-lint --model o3-mini"
```

Now when you're working on a repo, just do `aid` in your terminal.

Aider works better when you add the files you want to edit, so make sure to do `/add src/my/file/path.ts`
before you ask it to do work

Many repos will have a `/docs` directory, `/docs` directories are almost always designed to be added to
the AI context. Just do `/add docs` and the AI will get smarter!

If your repo is small, try adding everything with `/add lib tests README.md` then ask aider to complete
your task! This is what [bunaider](https://github.com/tscircuit/bunaider) does when you tag an issue with
`bunaider`!
