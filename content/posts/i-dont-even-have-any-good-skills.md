+++
date = '2026-06-11T19:07:12+10:00'
draft = false
title = "I Don't Even Have Any Good Skills"
+++

As Napoleon Dynamite said ...  {{< youtube-lite S1xQgYYhku4 >}}

This collection of skills exists to help my practice of coding at home. I typically use my Claude Pro subscription in a terminal on my Framework and, to preserve tokens, assign grunt tasks to [qwen3-coder:30b](https://ollama.com/library/qwen3-coder:30b) running on an RTX 4090 on a machine I bought from Scorptect two years ago. 

My primary motivations for building out these skills have been:

* experiment with agentic development
* experiment with different, open models running on my own local hardware
* ensure I am economical with Claude Pro tokens

The below began as a [monorepo of three skills](https://github.com/nicholasf/skills). As I began to refactor them last weekend they became five. I think this in itself was indicative that it was the right moment to refactor them into individual git repos.   

I might find time at a later date to blog at greater length about my view on [skills](https://agentskills.io/home). I've kept those viewpoints out of this post, in order to be brief. I'd be happy to discuss skills generally with anyone who wants to reach out.


## 1. Manage Skills

[manage-skills-skill](https://github.com/nicholasf/manage-skills-skill). 

This skill lets you `/manage-skills` with your agent to install, sync, etc. skills. It will also resolve dependencies of one skill to another in a skill's frontmatter (and detect dependency cycles).

I've modelled it on [`pnpm`](https://pnpm.io/), [`uv`](https://docs.astral.sh/uv/) and [`cargo`](https://doc.rust-lang.org/stable/cargo/) with some differences.

It takes a SKILLS_HOME env var, which is a location where all skills are kept globally. Here, a `skills.md` file is kept.

| name | url | load_at_startup |
|------|-----|-----------------|
| manage-skills-skill | git@github.com:nicholasf/manage-skills-skill.git | true |
| track-tasks-skill | git@github.com:nicholasf/track-tasks-skill.git | true |
| load-topology-skill | git@github.com:nicholasf/load-topology-skill.git | true |
| ask-remote-agent-skill | git@github.com:nicholasf/ask-remote-agent-skill.git | false |
| ask-remote-llm-skill | git@github.com:nicholasf/ask-remote-llm-skill.git | false |

_Table trimmed for readability — local paths and pinned versions omitted._


You will also see that there's a `load_at_startup` flag. For Claude, if you add the following to your `~/.claude/settings.json`  this will ensure that the skills you want (and their commands) are loaded on startup.

```
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"${SKILLS_HOME:-$HOME/.agents/skills}/manage-skills-skill/manage_skills.py\" context",
            "statusMessage": "Loading skills..."
          }
        ]
      }
    ]
  }

```

You can, however, use it in individual project with a `skills.md` file, that will then sync from the global installation. `skills.md` acts like a lockfile, also, specifying which version of a skill to install.

Lastly, it has a simple integration with `.env` files. I found that my skills began needing environment variables for API access, etc., and it was cleaner to let the management solution define a central ability for reuse.

## 2. Load Topology

[load-topology-skill](https://github.com/nicholasf/load-topology-skill).

This is probably the most interesting skill. It lets you build a map of machines on your network that you can then refer to to do other things. At the start of each of my coding sessions I now run `/load-topology`.

So, yes, natural language phrases about asking your primary agent to ssh into a machine and start an nginx instance, warm up an LLM or whatever. I find this aspect of using LLMs with machines enjoyable. 

**THERE ARE SECURITY RISKS HERE - ENSURE YOU ARE DOING THIS WITHIN YOUR HOME LAB AND THAT YOU UNDERSTAND WHAT YOU ARE DOING AND WHAT THE AGENT CAN DO.**

Running `/load-topology discover` will give you a detailed set of specs about your machines and the LLMs on them. 

Running `benchmark` can help you calculate [tokens per second](https://www.reddit.com/r/LocalLLaMA/comments/162pgx9/what_do_yall_consider_acceptable_tokens_per/) on the various LLMs you might be running. I currently get around 215 tks with qwen on my RTX 4090.


Note that a later skill, `track-tasks-skill` relies on the benchmark.

### topology.md

This is the primary table in topology.md

| hostname | os | ssh | gpu |
|----------|-----|-----|-----|
| pond | linux | yes | NVIDIA GeForce RTX 4090, 24GB VRAM |
| gollum | linux | yes | AMD Phoenix1 (ROCm) |

_Table trimmed for readability — IPs, agent config, and other implementation columns omitted._

The topology file has a convention that other skills can append data to it, as needed, in separate tables in the same file. This becomes necessary for other skills which leverage the topology for agent specific tasks. 


## 3. Ask Remote Agent (depends on load-topology-skill)

[ask-remote-agent-skill](https://github.com/nicholasf/ask-remote-agent-skill).

This skill allows one agent, Claude Pro, to communicate with another agent on another machine for any need, but for me it's typically coding.

I run [Goose](https://goose-docs.ai/) and [Hermes](https://hermes-agent.nousresearch.com/) as agents, right now fronting qwen.

Importantly, for code related tasks, the two agents will `sync` (a subcommand) and ensure they have the same language and code dependencies installed.

Then, the remote agent will run its work and send back a git diff, which I use to review with Claude. This is an example of such a flow.

```
local-claude-agent
  │
  └─ delegates ─────────────────► pond-qwen-hermes
                                        │
                                   [reads files]
                                   [runs bash]
                                   [generates diffs or opens PRs]
                                        │
                                   own tools, own machine
                                        │
  [pond-qwen-hermes] response ◄─────────┘
```

Importantly, ask-remote-agent introduces the notion of `agent handles`. You can see one used above - "pond-qwen-hermes" means the machine called pond, using qwen, running the hermes agent.

## 4. Ask Remote LLM  (depends on load-topology-skill)

[ask-remote-llm-skill](https://github.com/nicholasf/ask-remote-llm-skill) is a precursor to ask-remote-agent.

It lets your primary agent (for me that's Claude Code) ask another LLM to do something without leveraging an agent. In this case, the remote LLM doesn't need to have the source code or even the language installed. It will just use its model to produce results.

The skill itself contains a list of tools that the primary agent can run on behalf of the remote LLM. So, for example, the remote LLM could ask the primary agent to read a particular file on its file system and send back the response. 

## 5. Track Tasks (depends on load-topology-skill, ask-remote-agent-skill, ask-remote-llm-skill)

[This](https://github.com/nicholasf/track-tasks-skill) is a classic task tracking ability which uses the file system to plan tasks and execute them in `pending`, `completed` or `deprecated` states.

A `development-log.md` is kept of all task work.

In a sense, it's very boring. In another, this is where a lot of the agentic logic occurs. I regularly ask Claude to create pending tasks, debate their size and their use, and then hand them off to Goose or Hermes running on pond, my RTX 4090. Then I review what comes back, with Claude.

A [programme task](https://github.com/nicholasf/track-tasks-skill#programme-tasks) will let you create a parent task that organises many tasks under one umbrella.

Interestingly, I have been able to generate an estimated processing time for [the duration a remote agent might take](https://github.com/nicholasf/track-tasks-skill#estimate-time) to complete a task. This is done by taking the `context-window` size of an LLM calculated by running `/load-topology benchmark`, then calling a `tokenize` function on the target agent with the task itself, understanding how many files need to be parsed by the remote agent, and then understanding how much of the context window is being consumed by the task. That lets the primary agent put a level of difficulty on the task itself - L1, L2 or L3 - that can be correlated to how many tokens can be processed in how many seconds. 

The gap here is the computation time of the LLM itself, I suppose, in how long it takes to process certain token scenarios but, remember, these machines aren't taking time to think. They are just processing tokens along decision trees, etc. So far it seems reasonable to assume it's a matter of understanding token processing size constraints and throughput. This feature needs more testing but has been fun.

