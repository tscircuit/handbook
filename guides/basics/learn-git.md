# Learn Git

You're expected to know how to use git on the command line to work with tscircuit (and most open-source projects)

You can learn git from this interactive tutorial: https://learngitbranching.js.org/?locale=en_US

Git is so essential to your career it is worth taking that entire tutorial and others. Don't skip rebasing or other
"unusual" concepts. You will save hundreds of hours of debugging by just familiarizing with `git` early on. VS Code's
git management is not good enough to resolve the 5% of issues that you will spend 99% of your git time debugging. Do
not rely solely on vs code git functionality.

## Resolving `bun.lockb` conflicts

1. `git fetch`
2. `git merge origin/main`, you are now resolving the merge.
3. Go to the `package.json` file and pick the right dependencies/changes to incorporate, usually the latest versions
4. Run `bun install`, this automatically resolves `bun.lockb` issues
5. `git add package.json bun.lockb`
6. `git commit` to commit the merge result. Note that you should keep the default merge message
