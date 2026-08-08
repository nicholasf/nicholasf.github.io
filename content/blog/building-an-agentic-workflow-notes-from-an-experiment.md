+++
date = '2026-08-08T12:10:17+10:00'
draft = false
title = 'Building an Agentic Workflow: Notes from an Experiment'
+++

TL/DR - my current agentic programming model is to use Claude Code with another machine running qwen. Both machines use LLMs and agents to work on one task sequentially on one clone of a codebase. Sometimes I use multiple clones and accept git diffs from remote agents handed tasks. Sometimes well known companies produce questionable code using AI that you are running and probably don't know about.

![Agentic communication](/images/Fig_7_Le_Telephone_by_T_du_Moncel_Paris_1880_(Large).jpg)

I'm writing this with a slight cold and a touch of fatigue, so if that comes through, excuse it.

A few months ago I blogged about [a set of skills](https://nicholasf.dev/blog/i-dont-even-have-any-good-skills/) I'd developed which allow me to hand off work to Claude Code and some open models running on another machine in my home lab (here "home lab" means gaming machine in the next room). I've developed a reasonable way of planning work in the form of tasks, then handing them off to either Claude Code (mostly running Sonnet) or qwen3-coder-30b running in the next room. This blog post is about that.

Developing my own working model has coincided with the larger machinery of AI news in the wider industry and my own thoughts around it. I think LLMs are a tremendous technology which a software engineer needs to adopt. To my mind, they are something like hallucinating encyclopedias that can help you form input parameters to other programs, which they may or may not be involved in writing. They are fun, incredibly informative, in need of truth checking and, if approached correctly, useful.

I think the economics and sides being formed in the industry racing toward IPOs and winning an AGI race are particularly distortive on how to use the technology properly. It's clear that the weekly news cycles about Anthropic, OpenAI and rich men sending things into space are affecting financial markets and workplaces alike.  

I'm a fan of [Ed Zitron's reporting](https://www.wheresyoured.at/) lately. If that helps position my thinking on the AI hype. I am also a fan of Yuval Harari and I find his thoughts about [language being a system that could produce both human and other forms of intelligence](https://www.youtube.com/watch?v=_V_ed5fuexA&t=235) to be better articulations of the vague ideas I've had regarding the connections between language and consciousness and what comes first.

Anyway, to the point of this post, my simplified model of using agents has worked well enough (not perfectly). I can build a series of specifications, break them into smaller pieces called tasks, calculate their complexity sizes and have an LLM perform the work for me. 

That's the good. The bad has been losing time to hallucinations or with debugging why agents do not respond in a timely fashion. 

Note, I wanted to build this foundation of work myself and chose to do so using skills. Simultaneously, I've formed opinions about how to write a good skill, which I might discuss elsewhere. 

I could have adopted [gastown](https://github.com/gastownhall/gastown) or any number of other projects. I've deliberately wanted to build my own agentic workflow, as a learning exercise. Also, partly as a test to see if we are now at a stage where coders like myself can engineer large systems alone, understanding how to designate work to LLM systems - where software becomes cheaper and easier to produce but still requires understanding to reach a quality result.

Pragmatically, due to having a few machines in my house, my approach has differed to some of my friends, who instead rely on using several models running locally (usually on Macs) and using many agents that contribute via [git worktrees](https://git-scm.com/docs/git-worktree).

I've been more interested in using multiple machines with their own copies of source code, LLMs and agents, mostly to leverage my own expensive GPU at home but also out of concern for excessive token usage. On another level, I think pricing programming by token consumption is one of the biggest exhibitions of the emperor's new clothes currently in the industry. No one can afford this model, a token count doesn't equate to an outcome and has anyone really questioned why we the industry has swallowed this pill? At any rate, that's a digression.

At the same time, I have been experimenting with building my own product - this might become its own blog post eventually. It has a Node backend, a React Frontend, a Postgres db layer, with Python sidecar services for agentic services. This has been a labour of love (I'd like to use this tool) but also an experiment to see how much productivity I can take out of LLMs as a single engineer. This experience has been interesting - it has let me think at a product level as I hand out more of the predictable work to LLMs. 

I won't digress and am more concerned to blog about my "simple agentic model" here ... 

So, my coding sessions currently resemble this.

Run Claude Code. Review a roadmap.md of features that correspond to feature specifications and architectural decisions kept in a docs folder. Clarify plans or decide I can write/generate code. 

For generating code I have Claude [wake up another LLM and/or agent](https://github.com/nicholasf/load-topology-skill) on another machine. We then break down a feature and/or a decision document into a [series of tasks](https://github.com/nicholasf/track-tasks-skill) (also estimating their complexity level). 

There are three ways I can ask an agent to work on something. I delegate harder tasks to Claude Sonnet or simpler tasks to another remote agent/LLM. I can coordinate with the other agent/LLM in two ways - using a [remote agent](https://github.com/nicholasf/ask-remote-agent-skill) or using a [remote llm](https://github.com/nicholasf/ask-remote-llm-skill).

Work is considered complete when I have reviewed a task and a complete test run has been completed for unit, integration and e2e tests.

Here's an example of a coding session from today. I had reached the stage where I was happy with a number of architectural decisions.

| # | Decision | One-liner |
|---|---|---|
| 0001 | Game system attribute schema registry | Generic `ingame`/`campaign` entities + a game-system attribute schema registry; the registry table itself later superseded by 0005 |
| 0002 | World/campaign entity forking and publishing | Entities and world Pages stay single/shared; all campaign-specific difference lives in forked Pages + Lore; publishing is a merged, frozen view |
| 0003 | Access and Secret (visibility model) | Two-layer model — Access (`gm`/`party`/`viewer`/`custom`) gates existence, Secret (same value set) gates narrower per-account facts |
| 0004 | Publishing, versioning, and Custom content | Publishing releases a version into a catalogue via versioned tables; Custom-gated content ships unassigned, for the purchasing GM to assign to their own party |
| 0005 | Game-system entity classification hierarchy | `gamesystem.entity_types` (classifications with real parent/child hierarchy) + `campaign.game_system_instance_data`, superseding 0001's flat `attribute_schemas` |
| 0006 | Simplify to Access and Secret | Removed Knowledge as a layer, renamed `Lore` to `Secret` — content now folded directly into the rewritten 0003 |

Two coherent implementation areas: **Information Visibility** (0003/0006) and **RPG System Data** (0001/0005). 0002 and 0004 sit outside both — scoping/publishing-model work, not part of either pairing.

I then asked Claude to break them into particular tasks, estimate complexity and to relate them to decisions:

| Task | Decision | Complexity | Agent | Status |
|---|---|---|---|---|
| Rework `campaign.permissions` to `gm`/`party`/`viewer`/`custom` | 0003/0006 | Low | qwen | Created |
| Rename `campaign.lore` → `campaign.secret`, `revealed` → `level` | 0003/0006 | Medium | qwen | Created |
| `viewer`/`Visitor` naming cleanup | 0003/0006 | Low | qwen | Created |
| Access → Secret resolver logic | 0003/0006 | Medium–High | Claude | Not delegated — my own next-session work, no task file yet |
| Replace `gamesystem.attribute_schemas` with `entity_types` | 0001/0005 | Medium | qwen | Created |
| Rename `entity_game_system_attributes` → `game_system_instance_data` | 0001/0005 | Medium | qwen | Created |
| Drop `magic_items`, migrate to plain `item` + RPG System Data | 0001/0005 | Medium | qwen | Updated (existing) |
| Frontend dynamic form rendering | 0001/0005 | Medium–High | qwen | Updated (existing) |
| New `Creature` base entity | 0001/0005 | Medium | qwen | Created |

**Dependency order:** `entity_types` before `game_system_instance_data` (references it); both before the `magic_items` and frontend-form tasks (both re-specced against the new shape). Everything else is independent.

- Pond (`qwen3-coder-30b`, `http://pond:9337`) was confirmed running as of this session — re-check with `curl -s http://pond:9337/v1/models` before delegating, it isn't always left running.


Importantly, there is a [defined FSM](https://github.com/nicholasf/track-tasks-skill/blob/5321d1d575d776b9bd990d206298af982fb08762/scripts/workflow.py#L12) that needs to be followed for a task to be completed. Some, I mark as hallucinated and move to a separate folder in case I want to analyze them later.

Initially I thought working with a remote agent and LLM would be the most useful option. I have since found it simpler to use the abilities of the remote LLM skill, which relies on letting a remote LLM [*use the tools* of the calling agent](http://github.com/nicholasf/ask-remote-llm-skill#how-it-works) - so, qwen, in some circumstances, will tell Claude to execute [tools](https://github.com/nicholasf/ask-remote-llm-skill/tree/main/tools). 

The main issue with adopting agents themselves was due to a lack of visibility about what a particular agent was doing. In particular, two months ago, I found these [issues with Hermes](https://gist.github.com/nicholasf/a808730c305969f6f2c59a8b6bc73e2e) which I think are fundamental security problems. I did email the Hermes team about them back then, so I feel okay about sharing this here. I received no reply. Perhaps I have missed the point about it being okay to let your content be distributed to LLMs that you may not be aware of. Perhaps there is a lot of software being produced currently that isn't well understood and could be thought through a little further.

I do particularly like [Goose](https://goose-docs.ai/). It's what I've been using lately when I want to introduce another agent fronting qwen. Again, I have found difficulty in receiving feedback from it when handing it a task and overseeing how it executes. Interestingly, [integrating with it](https://github.com/nicholasf/ask-remote-agent-skill/blob/main/goose/acp.py) required use of [Agent Client Protocol](https://agentcommunicationprotocol.dev/introduction/welcome) which was different to the [HTTP integration](https://github.com/nicholasf/ask-remote-agent-skill/blob/main/goose/acp.py) I needed to use for Hermes.

I do have plans for a more refined approach to using many agents at once. I have formed notes for it - it would rely on an eventing and brokered model using [Google's Agent2Agent protocol](https://github.com/a2aproject/A2A). Participating agents will need to publish events about progress. And more. I've not built it yet, I may not, and I only mention it to call out that I see the limitations of my current process *but* it is yielding results and has helped me build opinions. I've been too preoccupied with building my product to keep building out my own tools. It's been useful to use what I have, sometimes at a rudimentary level, and to form ideas about how to improve them, rather than rushing to a more advanced model.

More to come. Once I have a "usable product" (not MVP) in place I would hope to blog about building it with these tools.