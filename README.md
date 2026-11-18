# AI Release Tracker (no-code, n8n)

A free, self-hosted n8n workflow that watches OpenAI, Anthropic and Google, and writes a weekly brief on **what changed and why it matters to product managers**. Every item carries a category tag, a PM lens, an urgency and a source link.

Built as a hands-on project to practise AI product management: scoping, prioritising under a quota, human review, and checking an LLM's output without spending money.

## The problem

AI vendors ship pricing changes, deprecations, new models and policy updates every week, spread across blogs and changelogs. A PM building on these models needs to know which changes affect roadmap, cost or risk, and which are noise.

Newsletters summarise the news. This workflow answers a narrower question: **what should a PM do about it?**

## What it does

→ Reads RSS feeds from a `sources` table (add a product by adding a row)
→ Keeps the last 7 days, removes duplicates, and scores items by keyword before spending any tokens
→ Caps items per company so one busy feed cannot crowd out the others
→ Sends each selected item to an LLM for tags, a summary and a suggested PM action
→ Writes a short brief of the top 5 items, in plain language
→ It also tells which "tags" or "lens" showed up most
→ Runs 10 rule checks on the output, then emails a draft for human review

## Architecture

```mermaid
flowchart TD
  subgraph TRIG["1. Triggers"]
    S["Weekly Schedule<br/>Mondays 9:00"] --> LIVE["Run Live<br/>mode = live"]
    M["Manual Test Run"] --> TEST["Run Test<br/>mode = test"]
  end
  LIVE --> SET
  TEST --> SET
  SET["Settings<br/>model, limits, keywords, prompts"] --> CH{"Use sample<br/>items?"}

  subgraph FETCH["2. Collect"]
    SRC[("sources table")] --> RSS["Read Feed<br/>one per source"]
    RSS --> TAG["Tag Source"]
    TAG --> DATE["Last N Days filter"]
    SAMP["Sample items<br/>test only"]
  end
  CH -- "no" --> SRC
  CH -- "yes" --> SAMP
  DATE --> DEDUP["Remove Duplicates<br/>by link"]
  SAMP --> DEDUP

  subgraph PRIO["3. Prioritise - no tokens"]
    SCORE["Score Items<br/>keyword points"] --> RANK["Rank In Company"]
    RANK --> PCAP["Per-Company Cap"]
    PCAP --> ORDER["Priority Order"]
    ORDER --> CAP["Cap Items<br/>max_items"]
  end
  DEDUP --> SCORE

  subgraph LLM["4. Analyse and write - LLM"]
    ITEM["Analyze Item<br/>1 request per item"] --> PARSE["Parse Reply<br/>and Shape Item"]
    PARSE --> PACK["Pack Items"]
    PACK --> BRIEF["Write Brief<br/>1 request"]
    BRIEF --> CLEAN["Clean Brief<br/>remove dashes"]
  end
  CAP --> ITEM
  PARSE -.-> LOG[("llm_log table")]
  CLEAN -.-> LOG

  subgraph QA["5. Quality gates - no tokens"]
    STATS["Run Stats<br/>tokens and cost"] --> RULES["Rule Checks"]
    RULES --> REPORT["Check Report"]
  end
  CLEAN --> STATS
  REPORT --> EMAIL["Compose Email<br/>brief, coverage, checks, stats"]
  EMAIL --> SEND["Send Email via SMTP"]
```

Full node list, data tables and settings: [docs/architecture.md](docs/architecture.md)

## Example run

Real output from a test run on live feeds
Email title: AI release brief: 18 Nov 2026

Email body:
Characters: 1358 / 3000

This brief covers the top product updates and releases in AI this week.

→ Google: Gemini 4 Hydrogen: a faster model for general purpose robotics
Why it matters: This model and plugin combo lets teams build and test real-to-sim-to-real robotic applications much faster.
https://developers.googleblog.com/gemini-hydrogen-for-robotics/

→ Anthropic: Operate Claude Managed Agents inside your security perimeter
Why it matters: This feature helps PMs deploy AI agents while keeping data and credentials inside their company security perimeter.
https://claude.com/resources/articles/self-hosted-claude-managed-agents

→ OpenAI: Introducing GPT-6.2 Astra and Sol
Why it matters: Lower costs and better computer use capabilities enable you to build more advanced AI workflows.
https://openai.com/index/introducing-gpt-6-2-astra-sol

→ Google: Build full-stack Android apps with Gemini in Google AI Studio
Why it matters: This feature significantly lowers the effort required to prototype and launch full-stack mobile applications.
https://developers.googleblog.com/build-full-stack-android-apps-with-gemini-in-google-ai-studio/

→ OpenAI: Introducing ChatGPT Wallet
Why it matters: This enables AI agents to autonomously handle financial transactions with user approval.
https://openai.com/index/introducing-chatgpt-wallet

The **Build** lens showed up most this week.


=== ALL ITEMS (25) ===  
[Model Release | Build | Plan this quarter | High] OpenAI: Introducing GPT-6.2 Astra and Sol
https://openai.com/index/introducing-gpt-6-2-astra-sol

[New Feature | Compete | Watch | Medium] OpenAI: Introducing ChatGPT for Sales
https://openai.com/index/introducing-chatgpt-for-sales

[Policy and Safety | Risk | Watch | Low] OpenAI: Strengthening the fight against online child sexual abuse with new tools and support
https://openai.com/index/strengthening-online-child-safety

[Other | Compete | Watch | Low] OpenAI: The King’s Trust and OpenAI launch £1m partnership
https://openai.com/index/the-kings-trust-and-openai

[Dev Tooling | Build | Watch | Medium] Google: Unlocking Stable On-Policy RL on TPU with Bitwise-Zero Train–Inference Mismatch
https://developers.googleblog.com/unlocking-stable-on-policy-rl-on-tpu-with-bitwise-zero-traininference-mismatch/

[Other | Compete | Watch | Low] OpenAI: Our new blog for science
https://openai.com/index/our-new-blog-for-science

[New Feature | Build | Plan this quarter | High] OpenAI: Introducing ChatGPT Wallet
https://openai.com/index/introducing-chatgpt-wallet

[Policy and Safety | Risk | Watch | Low] OpenAI: Supporting research on AI and children’s safety
https://openai.com/index/children-first-declaration

[New Feature | Build | Plan this quarter | High] Anthropic: Introducing Claude for Manufacturing
https://claude.com/resources/articles/claude-for-manufacturing

[Other | Compete | Watch | Low] OpenAI: Introducing OpenAI for Chile
https://openai.com/index/introducing-openai-for-chile

[New Feature | Compete | Watch | Medium] OpenAI: Broadening access to AI for scientists
https://openai.com/index/expanding-chatgpt-for-academic-researchers

[Model Release | Compete | Watch | Low] Google: Introducing Gemini 4 Hydrogen
https://deepmind.google/blog/introducing-gemini-4-hydrogen/

[New Feature | Build | Watch | Low] Anthropic: Introducing Claude Meetings
https://claude.com/resources/articles/claude-meetings

[Model Release | Compete | Plan this quarter | Medium] OpenAI: Astra for Law leads in legal intelligence
https://openai.com/index/astra-law-6-2

[Dev Tooling | Build | Plan this quarter | Medium] Google: TPU Model Design Considerations and Optimization Guide
https://developers.googleblog.com/tpu-model-design-considerations-and-optimization-guide/

[Model Release | Build | Plan this quarter | High] Google: Gemini 4 Hydrogen: a faster model for general purpose robotics
https://developers.googleblog.com/gemini-hydrogen-for-robotics/

[Policy and Safety | Risk | Plan this quarter | Medium] Google: Securing Your Agents: A Hands-On Guide to Agent Anomaly Detection
https://developers.googleblog.com/securing-your-agents-a-hands-on-guide-to-agent-anomaly-detection/

[New Feature | Cost | Plan this quarter | Medium] Google: Agents are sessions, not requests: size GKE node pools for AI agent sessions
https://developers.googleblog.com/agents-are-sessions-not-requests-size-gke-node-pools-for-ai-agent-sessions/

[New Feature | Risk | Plan this quarter | High] Anthropic: Operate Claude Managed Agents inside your security perimeter
https://claude.com/resources/articles/self-hosted-claude-managed-agents

[New Feature | Compete | Watch | Low] Anthropic: J. Crew brings its signature approach to styling to life with Claude
https://claude.com/resources/articles/j-crew-style-advisor-with-claude

[New Feature | Risk | Plan this quarter | Medium] Google: How Agent Anomaly Detection scores on benchmarks we didn't write
https://developers.googleblog.com/how-agent-anomaly-detection-scores-on-benchmarks-we-didnt-write/

[Pricing and Limits | Cost | Watch | Low] Anthropic: Linked billing on the Claude Platform
https://claude.com/resources/articles/linked-billing-on-claude-platform

[New Feature | Build | Plan this quarter | High] Google: Build full-stack Android apps with Gemini in Google AI Studio
https://developers.googleblog.com/build-full-stack-android-apps-with-gemini-in-google-ai-studio/

[Dev Tooling | Build | Watch | Medium] Google: Accelerating development for massive Git repositories
https://developers.googleblog.com/accelerating-development-for-massive-git-repositories/

[Other | Build | Watch | Low] Anthropic: The Claude Tag guide for startups
https://claude.com/resources/articles/claude-tag-guide-for-startups

=== COVERAGE ===  
Fetched after date filter and duplicates: 45
Analysed: 25
Not analysed: 20
Top 10 not-analysed items by priority score:
- [OpenAI] More ways to move care forward with ChatGPT for Healthcare (score 2, cut by per-company cap)
- [OpenAI] At Holiday Extras, dots get to work before anyone asks (score 1, cut by per-company cap)
- [OpenAI] Holiday Extras is rethinking what it can build with Codex (score 1, cut by per-company cap)
- [OpenAI] Strengthening NATO and allied cyber defenses with AI (score 0, cut by per-company cap)
- [OpenAI] Science at the Frontier: How scientists use ChatGPT (score 0, cut by per-company cap)
- [OpenAI] How Revolut raises ambition without lowering the bar (score 0, cut by per-company cap)
- [OpenAI] LVMH’s Media Division announces strategic partnership with OpenAI (score 0, cut by per-company cap)
- [OpenAI] Our first phase of embedded assessments (score 0, cut by per-company cap)
- [OpenAI] Advancing computer-use agents for healthcare teams (score 0, cut by per-company cap)
- [OpenAI] A step toward defining the Standard Model on a lattice (score 0, cut by per-company cap)

=== RULE CHECKS (9 of 9 passed) ===  
- PASS    Brief under 3000 characters
- PASS    No em-dashes or en-dashes
- PASS    No hashtags
- PASS    Every link in the brief is a real fetched link
- PASS    Brief covers 5 items or fewer
- PASS    All tags use allowed values
- PASS    Every item has summary, so-what and action
- PASS    No duplicate items
- PASS    Every selected item was analysed
- PASS    Numbers in the brief appear in the source text (can false-alarm, check by eye)

=== RUN STATS ===  
- Mode: live
- Model: gemini-3.5-flash-lite
- Tokens in / out: 12996 / 3027
- Estimated cost (USD): 0.0000

This email was sent automatically with n8n
https://n8n.io


## Design decisions

| Decision | Why |
|---|---|
| **Three tags plus impact** (category, PM lens, urgency, impact) | A summary is not a decision. The PM lens (Build, Cost, Risk, Compete, Users) says why the change matters. Urgency says when to act. |
| **Prioritise before spending tokens** | The first version kept the 15 newest items, which let minor posts push out an older pricing change. Keyword scoring, a per-company cap and a visible coverage report fixed this. |
| **Requests, not tokens, are the scarce resource** | On a free LLM tier, daily request limits bite long before token limits do. One request per item plus one for the brief keeps the cost predictable. |
| **A human reviews every draft** | The brief arrives by email as a draft. There is no auto-posting. |
| **Links come from the data, never from the model** | A rule check confirms every link in the brief was actually fetched. |
| **One provider setting** | The LLM call uses an OpenAI-compatible endpoint. Switching model or provider is two Settings fields and one credential. |
| **A plain HTTP request, not an AI agent node** | Each step is one prompt in and one JSON out. That makes it cheaper (often free), faster and more deterministic. No memory and no tool calling are needed. |
| **Free first** | n8n Community Edition (self-hosted), Gemini free tier, Gmail SMTP. Total cost is zero. |

## How output quality is checked

Ten deterministic checks run on every execution and add no LLM cost. They cover length, banned characters, link integrity, item count, allowed tag values, completeness, duplicates, and whether every selected item was analysed. A tenth, soft check flags numbers in the brief that are missing from the source text.

What these checks **do not** cover: whether the tags are *correct*. A brief can pass every check and still label a pricing change as a new feature. Measuring accuracy needs a hand-labelled golden set, which is the first item under next steps.

The checks themselves needed tuning. The first version flagged real links as fake when the model wrapped them in markdown bold or added a trailing slash.

## Cost and limits

- **LLM requests per run = items analysed + 1.** With `max_items = 25`, a full run is about 26 requests. Retries can add more.
- Token usage is logged per call in the `llm_log` table and summarised in each email. In testing, the API's total tokens matched prompt plus completion tokens.
- Free-tier quotas are set per project and change over time. Check your own limits in Google AI Studio before raising `max_items`.
- On the free tier, Google may use prompts to improve its models. All inputs here are public content.
- The machine running Docker must be on at the scheduled time. A quiet week with no new items sends no email.

## Known gaps and next steps

- **Golden-set evaluation.** Hand-label about 30 real items and measure tag accuracy and which categories get confused.
- **API changelog pages are not covered.** Pricing changes and deprecations often appear only there, and they have no feed. This is the biggest gap for PM value.
- **Keyword scoring is a heuristic.** It is transparent and free, but it will miss unusual wording. The coverage report shows what was dropped, so the keyword list can be tuned.
- **Community-generated feeds can break.** The Anthropic and Google Developers feeds are third-party. Disable one by setting `enabled` to false in `sources`.
- **Batching or title-only triage** would cover more items per request.

## Repo contents

```text
workflows/   Three n8n workflow exports (tracker, one-time setup, error alert)
prompts/     The two prompts, as text
docs/        Architecture and setup guide
```

## Quick start

1. Run n8n with Docker.
2. Import `workflows/AI_Release_Tracker_-_one-time_setup.json` and run it once.
3. Import the tracker and error alert workflows, then attach your credentials.
4. Run a test with sample items.

Step by step: [docs/setup.md](docs/setup.md)

## Tech stack

n8n Community Edition (self-hosted, Docker), n8n Data tables, Google Gemini API (OpenAI-compatible endpoint, free tier), Gmail SMTP.

The Anthropic and Google Developers feeds come from the open-source [rss-feeds](https://github.com/Olshansk/rss-feeds) project by Olshansk. The OpenAI and Google DeepMind feeds are official.

## Intention

- Light weight no code prototyping/automation learnt by connecting to LLM (via HTTP node) in n8n for purpose of analyzing and summarizing AI releases only, at 0 cost. 
- This can be easily tailored to do any Product's competitor release analysis and brief. 
- Only deterministic checks are in scope as of now and native Agent nodes in n8n have not been explored yet as they not required for this simple use-case. (memory/tooling/self correction loops not required)
- It was easier to make tweeks in .md files in this repo as I had learnt/experimented with them earlier. See [lessons-learnt-markdown](https://github.com/somi-iyer/allocation-algorithm-simulation/blob/main/lessons_learnt_md_file_formatting.md)


<!-- TODO: add a LICENSE file (MIT is a common choice) and a license line here -->
