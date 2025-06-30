---
source: https://www.pulsemcp.com/posts/how-to-use-claude-code-to-wield-coding-agent-clusters
edited: 2025-06-30
---

# How To Use Claude Code To Wield Coding Agent Clusters

## Pre-requisites

### This setup is most helpful for experienced software engineers working on mildly (or very) mature projects

Unlike most vibe coding how-to's, this setup is actually more useful the more mature your project is. If you can't imagine a team of human software engineers collaborating on your codebase without trampling over each other, you probably won't be able to parcel out work to multiple coding agents at once either.

And the degree of context switching and orchestrating you'll have to do is not something I believe would serve a non-technical user very well. Leave no doubt that this is a guide for software engineers, not vibe coders.

### We use Claude Code because it's optimized for capability

This [great writeup](https://x.com/nickbaumann_/status/1928143808870719743) from the Cline team explains why products like Cline (which includes Claude Code) are designed differently from monthly subscription products like Cursor, Windsurf, and GitHub Copilot. Briefly, the idea is that because Cline and Claude Code aren't trying to minimize token costs to maintain their own bottom line, they have made design decisions that might run up an API bill, but ultimately result in a better (more expensive) end-user experience.

An example of this: how [Claude Code decided to optimize for agentic search rather than RAG](https://x.com/pashmerepat/status/1926717705660375463).

The other major reason we're hitching our wagon to Claude Code is that it's so close to the source - Anthropic. Claude has been the state of the art coding model for a while now. So even if the Claude Code UX lags behind some bleeding edge innovations by teams like Cursor or Windsurf, we have faith that they'll continue to incorporate the best patterns in a relatively stable product that aligns well with new and upcoming state of the art model capabilities. Picking your stack here is as much a bet on what capabilities are to come in the future as it is on the capabilities available today.

Worth noting: you'll probably want a [Claude Max subscription](https://support.anthropic.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan) to use Claude Code generously for $100 - 200/mo, otherwise you're going to run up a pretty big API bill (likely in the many hundreds, if not breaching a thousand, per month).

### Our escape hatch: VS Code, but forks are fine

Agentic workflows in the modern state of AI are far from reliable. Every good agentic workflow needs a reliable escape hatch.

For developers, the ultimate escape hatch is to fallback into writing and editing code in an IDE.

We're choosing to go forth with VS Code as our escape hatch of choice. Our reason: VS Code is suddenly at the [bleeding edge of MCP](https://github.com/microsoft/vscode/issues/248627). As of this week, their Insiders build has full support for all MCP features, the first major MCP client app to do so.

And their Copilot UI/UX experience, with agent mode and other bells and whistles like built-in MCP server debugging capabilities, is suddenly at or near parity with the other AI code editors.

Top that with a [recent doubling down on open source](https://code.visualstudio.com/blogs/2025/05/19/openSourceAIEditor), and VS Code suddenly feels like both the stable, safe long-term choice and also a leader for a state of the art code editing experience. But there's no reason you can't opt for a Cursor or a Windsurf to be your escape hatch instead.

Worth noting: it's actually been surprisingly less common for me to dig into this last-ditch escape hatch than I expected. In general, Claude Code is very good at taking feedback and making minimal changes in response - so I'll often go back to prompting the CLI instead of making line-changes in the editor.

### This setup wasn't practical before May 2025

We only just made the transition to this stack last month because this setup was not possible before May. Three major pieces recently fell into place:

1. [Claude 4 Sonnet + Opus release](https://www.anthropic.com/news/claude-4). In particular, the announcement's note that "Rakuten validated its capabilities with a demanding open-source refactor running independently for 7 hours with sustained performance" caught our attention: the model has cracked a threshold where, in the right harness, it really might be able to go on productively coding forever.
2. [VS Code + Claude Code integration](https://docs.anthropic.com/en/docs/claude-code/ide-integrations#supported-ides). Prior to this integration, juggling the two would have been clunky, making that "escape hatch" experience full of friction. Now, you can view diffs in your editor and act on them, course-correcting Claude Code effectively as-needed.
3. [Claude Max subscription option](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md#0296). Before this, you had to pay usage-based API fees to use Claude Code. This created a tricky dynamic where you would probably find yourself optimizing for cost turn by turn, which handicaps the entire experience. Now instead, you commit to $100 or $200 per month and let it fly.

## Setup

### Agents need a closed feedback loop + observability

The magic in an LLM-powered agent comes when you provide it with two things:

1. A complete, autonomous feedback loop. Instead of requiring conversation turns to tell your agent "I just tried X and it didn't work, try again", you set up your environment so the agent can "try X" and assess the result on its own.
2. Observability into what went wrong. Taking the above a step further, it's often not enough to know an attempt at a solution didn't work. You want to know the error code or the stack trace that led to the failure. So you need to give your agent access to those logs or metrics.

These principles aren't anything new. You're probably already doing them as a software engineer. The key here is to make sure the agentic loop can do it without any interventions on your part.

There are at least two layers where we can do this within a codebase:

#### 1. Local automated test suites

With a local test suite, you get both that feedback loop + observability right in your CLI. All you need to do:

1. Have a pre-existing testing philosophy (unit vs. integration vs. system, etc) and architecture in place
2. Add notes in your `CLAUDE.md` file explaining the setup
3. When kicking off a Claude Code prompt to do a piece of work, mention to use TDD, and/or simply "write new tests in accordance with precedent" and to keep going until the tests work

Claude Code is great at running CLI commands and reading their output, so this is a pretty straightforward feedback loop to leverage out the gate.

#### 2. Use a staging/preview environment

In many cases, a local test suite is not quite enough. Maybe your unit testing philosophy doesn't (and shouldn't) have 100% coverage, or you're trying to develop some more infrastructure-level piece of work.

Even though it's not as fast as a local testing approach, Claude Code can work with that too. And because we're aiming for autonomy, the speed is not as critical as the closing of the loop. In this case, we'll be working with:

1. You need a good CI/CD pipeline capable of deploying to a staging environment. We set up our `CLAUDE.md` file to use the `gh` CLI to push to a branch, and then comment `/deploy staging` to kick off a GitHub action that deploys that branch to your staging environment. You can instruct it to "wait for the deploy to succeed before proceeding" so it doesn't delegate the waiting work back to you as the end-user.
2. You want to guide Claude Code (again, via `CLAUDE.md` file; or just via your one-off prompting) into actually being able to interface with the staging environment. Depending on your application, a nudge to use `curl` might be enough. See the section on MCP below for more advanced ideas.
3. Lastly, the observability piece. The simplest approach may be to provide `ssh` login instructions via your `CLAUDE.md` file. That'll empower it to check out infra configurations, pull log files, and more from the remote's CLI. MCP might be helpful here too (see below).

### Git worktrees + a simple bash script are the secret sauce to 10x

Even if you've done all the above, you might not feel that much more productive wielding Claude Code. Now you just sit there, waiting for Claude to (slowly) crawl files, pull together context, and derive insights that you might feel like you could have done more quickly yourself.

Enter: [git worktrees](https://git-scm.com/docs/git-worktree).

A little-known git feature that I've never found much use for in the past until inspired to revisit it by [the Claude Code docs](https://docs.anthropic.com/en/docs/claude-code/tutorials#run-parallel-claude-code-sessions-with-git-worktrees), git worktrees enable you to check out multiple branches at a time within a single filesystem. This is what enables you to wield multiple instances of Claude Code, all at the same time, all sharing your precious and quirky local development setup.

And we can use bash scripts (written by Claude Code, of course) to manage all the details for us. My final flow looks like:

1. I identify an issue I want to work on
2. I run my bash script via an alias, `pgw new-feature` while sitting on my `main` branch
3. This creates a new branch and worktree on `new-feature` and automatically opens VS Code for me with Claude Code loaded up
4. I prompt Claude Code, and it goes off and works for several minutes
5. In parallel, I can go back to my terminal and do another `pgw new-feature-2`
6. When I'm done, I run `pgwr`, which opens an interactive CLI that allow me to easily "clean up" my worktrees

You can infer what `pgw` and `pgwr` does from this flow. I'm sharing these as gists here; I recommend you grab them, tell Claude Code to adapt them for your codebase in a `bin` directory + bash alias to an easy command to invoke, and tweak them to your heart's content:

* [gw.sh](https://gist.github.com/tadasant/c549fe6f07017d8d307151bbe923ab69): utility that provides a one-line way to spin up a Claude Code-friendly `git worktree` and throw you right into a VS Code (Insiders) instance, ready to get to work
* [gwr.sh](https://gist.github.com/tadasant/879a63db0aab5ccb6fb446937f1fa848): interactive utility that, when invoked, checks the status of all your `git worktrees`, whether or not their associated PR's have been merged, and provides single-key functionalities to e.g. "delete all except main" for easy ergonomics.

There are definitely some codebase-specific details there, like copying .gitignore'd secrets into the worktree, and managing MCP servers (more on that below); so you'll definitely want to adapt these before using them yourself.

### Odds and ends

Here are a few minor tips that might not be obvious, but have been critical to me getting maximum value out of this setup.

#### Use `claude --dangerously-skip-permissions`

Of course, caveat with all the usual disclaimers: be very careful with this if giving Claude unfettered access to your CLI is a serious security risk for you personally or for your business.

That said, the experience of autonomously managing a cluster of agents is much harder if you have to approve actions repeatedly. You could, in theory, get there with enough "allow all" whitelists; but I personally found building that list too frustrating to deal with when I could instead just be careful with what accounts and MCP servers I handed Claude Code access to.

#### Leverage `CLAUDE.md` and other .md documentation

I've mentioned several times that you want to train Claude on your preferred SDLC methods with `CLAUDE.md` files, so I won't repeat that call here.

I also found other one-off documentation files to be helpful. For example, I have a file called `STAGING.md` that explains our staging setup. I don't always need Claude Code to be aware of this - only if I'm asking it to do a change that it should deploy to the staging environment. So, when I know I'll need that, I prompt Claude Code to "check the staging guidance in docs/`STAGING.md`" like I might to a junior engineer taking on a ticket.

#### Leverage custom slash commands

##### PR

For repeating development workflows, you'll find Claude Code's [custom slash commands](https://docs.anthropic.com/en/docs/claude-code/tutorials#create-custom-slash-commands) useful.

My often-used one is a command I use to "open a PR". After Claude Code has finished working, and the git diff it left behind is satisfactory to me, I will often do `/pr` to have Claude Code proceed to opening a thorough PR (with cleaning up lint, having good commit messages, PR explanation, handling any messy git branch gymnastics I may have done, etc).

Here's what that `.claude/commands/pr.md` looks like for me:

```markdown
# Push working state to a PR
Our goal is to take the current state of our git diff (ALL The files), commit the files (with a reasonable commit message), push them (to a branch), open a PR,and surface the link to the PR back to me so I can click it.

## Workflow Details
For detailed git workflow information including branch naming conventions and recovery procedures, see [docs/GIT_WORKFLOW.md](../../docs/GIT_WORKFLOW.md).

## Quick Reference

### If on wrong branch
  - **On main**: `git reset --soft` to origin/main, stash, create feature branch, pop stash
  - **On unrelated branch**: Same process - reset to origin, stash, checkout main, pull, create new branch

### Pre-PR checklist
  1. Run `bundle exec standardrb --fix-unsafely` and commit any fixes
  2. Run tests with `bin/test test/*/*.rb` if changes might impact them
  
### Post-PR
  - **Regular repo**: Checkout main and pull latest (while showing PR URL)
  - **Git worktree**: Leave as-is (user will delete when PR is merged)
```

##### Session management

I manage my development sessions with a [set of commands](https://github.com/iannuttall/claude-sessions/tree/main) that help me document my work for future reference. This is a great way to keep track of what I did, why I did it, and what I learned along the way.

#### Tasks.json is a nice VS Code feature to fire up Claude Code right away

When I run `gw.sh`, I want the resulting VS Code instance to already be running Claude Code, ready for my prompt. The config at `.vscode/tasks.json` makes that possible:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Run Claude Code Dangerously (Automatically)",
      "type": "shell",
      "command": "/Users/admin/.claude/local/claude --dangerously-skip-permissions",
      "presentation": {
        "reveal": "always",
        "panel": "new",
        "focus": true
      },
      "runOptions": {
        "runOn": "folderOpen"
      }
    }
  ]
}
```

### MCP is the missing piece for everything else

It's amazing how far you can get with having Claude Code just use the CLI with commands like `gh`, `curl`, `ssh`, etc. You can definitely get a lot of mileage out of Claude Code just using its native capabilities.

But MCP is where things can get really interesting.

When you've made that deployment to staging, you could hook up [Microsoft's Playwright MCP server](https://www.pulsemcp.com/servers/microsoft-playwright-browser-automation) to have Claude Code run a browser and test whether your change is working.

If you're trying to debug a production issue where you're running Logfire to manage your observability, you could set up the [Logfire MCP server](https://www.pulsemcp.com/servers/pydantic-logfire-opentelemetry) to have Claude Code build hypotheses that it then goes to test on its own in your staging environment.

To shortcut your Claude Code prompting, you could hook up the [Linear MCP server](https://www.pulsemcp.com/servers/linear) and just reference the issue number to get started.

Maybe you're having trouble making your `gh` calls deterministic, and the [GitHub MCP server](https://www.pulsemcp.com/servers/github) could smooth that over for you.

… and so much more.

You'll notice in my `gw.sh` file that I've introduced a notion of "MCP templates" (perhaps should be renamed to "profiles") where I've started to curate some common combinations of MCP servers I might want Claude Code to access. The way this works for me:

1. When I run `pgw new-feature`, I can do something like `pgw new-feature -m dwh` to invoke an MCP profile
2. That `dwh` (data warehouse) profile contains the MCP servers I might need to work with the data-heavy portion of my monorepo: BigQuery, dbt, Postgres, etc.
3. The bash script copies the .mcp.json template to be in the root folder, where it is .gitignored but active within that worktree branch
4. This means Claude Code is started with those MCP servers active

This is rather crude, and hopefully something for which there might be a more elegant solution before long. But it works, and has very minimal friction after an initial setup investment on a per-profile basis.

## Taking steps like these is how software engineers can adapt

I believe [Dario Amodei's claim](https://www.axios.com/2025/05/28/ai-jobs-white-collar-unemployment-anthropic) that AI is on course to wipe out a massive number of white collar jobs as they are defined today. I also believe you and I as individuals have the agency to adapt, and change the level of abstraction at which we work.

Instead of writing lines of code ourselves, we need to learn how to orchestrate coding agents to build features around thoughtfully designed architecture and, perhaps more importantly, construct agent harnesses that ensure the agents have the tools and context they need to do their job properly.

This transition might be as painful as the common failure of strong individual contributors to transition into engineering management. It does require a different kind of craft, one that a veteran software engineer might be loath to reinvent themselves around.

And maybe this particular path of adaptation is specific to application developers. Maybe those out there writing compilers, or hairy security infrastructure, or low level firmware, won't see quite the same upending of their professions.

But this pattern you can start using immediately, today, is very evidently the first sign of the "agent cluster" phase of the [revenge of the junior developer](https://sourcegraph.com/blog/revenge-of-the-junior-developer).

Happy coding.
