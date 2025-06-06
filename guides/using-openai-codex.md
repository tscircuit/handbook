# Using OpenAI Codex

You can easily 2x or 3x the number of PRs you close if you leverage Codex. In general, you should deploy the following flow:

1. Start 2-5 tasks simultaneously for unrelated tasks that need to be done using carefully crafted prompts
2. Find new tasks while waiting for old tasks to complete
3. Every 5 minutes, visit Codex and do code reviews and push fixes
4. After the Codex review looks good, push to Github and wait for CI checks to pass (you may need to ask Codex for more updates)


Example view of codex working:

<div align="center">
<img src="https://github.com/user-attachments/assets/403f5e55-06b2-4660-a0fa-4133c83a0cbb" style="height:320px" />
</div>


## Creating Environments

1. Always enable unrestricted internet access if there are no secrets and it's an open-source repo
2. The setup action is usually just "bun install" (but don't forget it!)
3. Create the repos you normally work in

## Crafting Prompts

- **ALWAYS START A PROMPT WITH YOUR INTENT**
  - "We want to show a downward ground symbol whenever a net label points downward to help makes schematics more readable"
- **NEVER PUT INCORRECT INSTRUCTIONS IN YOUR PROMPT**
  - It is better to be vague than incorrect
- **BE PRECISE, DO RESEARCH**
  - Go into the codebase and tell the AI what you think could be relevant, giving precise file names or paths

### Example Prompts that were Successful

```
#tscircuit/cli
when running "tsci init", prompt the user for the packageName of the package
with a default of the current directory name, and in the package.json "name"
field make it so it follows the tscircuit format of
"@tsci/<authorName>.<packageName>". The "authorName" should be available
in the global tscircuit config if the user has ever logged in (we can decode
the JWT from the session token to get github_username, as well as the account_id
and the session_id)

Also: add a script to the package.json "start": "tsci dev" for new projects
```

```
#tscircuit/core
When we are rendering/creating a net label (sometimes we do this inside Trace.ts),
if the symbol is downward facing connects to a ground net, replace the net label
with the ground_down symbol and keep the label text

To do this, introduce a new render phase called SchematicReplaceNetLabelsWithSymbols

Make sure to `bun update schematic-symbols --latest` 
```


```
# tscircuit/cli
For `tsci build`, check the circuit json for errors and log all the errors and throw an exit
code if there are any. Check out the circuit json spec
https://github.com/tscircuit/circuit-json for error definitions, if something has
"error_type" as a key or the .type ends with "_error" then it's an error.

Log errors in red, log warnings in yellow. Add an option `--ignore-errors` and `--ignore-warnings`.
```

```
# tscircuit/jlcsearch
Introduce a new page for browsing gyroscope chips, you'll need to create a derived table.
Make sure to include booleans for protocols support e.g. `i2c` `spi` so that we can easily filter by those
```

```
# tscircuit/core
Remove all the logic related to special handling for upward-facing and download-facing power
symbols inside Trace.ts and any utility functions
```

