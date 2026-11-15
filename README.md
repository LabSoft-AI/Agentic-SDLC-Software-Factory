# Agentic Software Development Lifecycle: Building a Software Factory

3-week master's course where students build a software factory: specialized
agents that automate each phase of the software development
lifecycle, wrapped in a harness and run in loops, then used as a team to
build a real application. You can run the agents in either of two agent
harnesses: [Kiro CLI](https://kiro.dev) or [Pi](https://pi.dev).

A **software factory** is a repeatable pipeline where specifications go in
and tested, deployed software comes out. Each phase of the lifecycle is a
station run by a specialized agent. The harness is quality control at every
station, loops let a station retry until its checks pass, and humans define
the specs, set the rules, and review the output.

The course is grounded in four emerging disciplines:

- **Context Engineering** — controlling what goes into an agent's context
  window (system prompt, resources, tools, conversation history) to
  maximize output quality.
- **Harness Engineering** — building guides (feedforward controls) that
  steer the agent before it acts, and sensors (feedback controls) that
  detect and self-correct issues after it acts. Pre-commit hooks, linters,
  test suites, and review agents are all part of the harness.
- **Loop Engineering** — designing the cycle an agent repeats (act, check,
  correct) with a guide that sets the goal, a sensor that judges the result,
  an exit condition, and an iteration cap. TDD's RED-GREEN-REFACTOR cycle is
  a loop, and BDD acceptance scenarios are its exit condition.
- **Spec-Driven Development** — writing specifications (tests, schemas,
  acceptance criteria) before implementation, then using AI agents to
  generate code that satisfies them. TDD and BDD with AI agents flip the
  traditional workflow: the spec becomes the feedforward guide, and the
  test suite becomes the sensor that validates the agent's output.

Each module teaches a software development process, then has students
encode it as context (agent config), harness (guides + sensors) and loops
(retry until the sensors pass) around an agent (run in Kiro CLI or Pi). The five agents
together form the stations of the software factory.

See the [Intro slides](modules/intro/) for a deeper dive into these
concepts.

📚 **[Course Documentation](https://labsoft-ai.github.io/Agentic-SDLC-Software-Factory/)**

## Course Structure

The course has two parts:

1. **Modules 1–5 (Individual)** — Each student builds one agent
   per module, learning the underlying software development process along
   the way.
2. **Module 6 (Team)** — Students form teams and use all five agents
   together to build the final project (to be defined).

## Modules

| # | Module | Process Learned | Agent Built |
|---|--------|-----------------|-------------|
| 0 | [Intro](modules/intro/) | Context engineering, harness engineering | — |
| 1 | [Git](modules/module-1-git/) | Branching, commits, PRs, issue tracking | Issue creator, PR creator |
| 2 | [Software Architecture](modules/module-2-architecture/) | arc42 template, C4 model | Architecture doc generator |
| 3 | [Software Requirements](modules/module-3-requirements/) | User stories, DDD, Pareto prioritization | Story derivation agent |
| 4 | [CI/CD](modules/module-4-cicd/) | GitHub Actions, Docker, pipelines | CI/CD pipeline agent |
| 5 | [TDD/BDD](modules/module-5-tdd-bdd/) | RED-GREEN-REFACTOR, backend-first | TDD/BDD implementation agent |
| 6 | Final Project | Full lifecycle (team project) | All agents from M1–M5 |

## Course Project: Final Project

To be defined. The team project will be built entirely by AI agents, using
the five agents from Modules 1–5.

## 3-Week Timeline

| Days | Activity | Deliverable |
|------|----------|-------------|
| 1–2 | [Module 1: Git](modules/module-1-git/) | Git agent + practice exercise |
| 3–4 | [Module 2: Architecture](modules/module-2-architecture/) | Architecture agent + kata design |
| 5–6 | [Module 3: Requirements](modules/module-3-requirements/) | Story agent + kata backlog |
| 7–8 | [Module 4: CI/CD](modules/module-4-cicd/) | Deploy agent + pipeline exercise |
| 9–12 | [Module 5: TDD/BDD](modules/module-5-tdd-bdd/) | Multi-agent TDD system + practice |
| 13–18 | Module 6: Final Project (teams) | Working app deployed |
| 19–21 | Buffer / presentations | Demo + retrospective |

## Assessment

| Criterion | Weight |
|-----------|--------|
| Architecture doc covers all 12 arc42 chapters | 15% |
| Story backlog has correct IDs, BDD ACs, traceability | 20% |
| RED phase committed before any implementation | 15% |
| All MVP tests GREEN | 25% |
| Traceability comments in every implementation file | 10% |
| App deployed and reachable | 15% |

## Branching

| Branch | Purpose |
|--------|---------|
| `main` | Production-ready code |
| `feature/*` | New functionality |
| `fix/*` | Bug fixes |
| `hotfix/*` | Urgent fixes |
| `chore/*` | Maintenance tasks |

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime | Python 3.12 |
| Frontend | React 18 SPA |
| CI/CD | GitHub Actions |
| Testing | pytest, pytest-bdd |

## Prerequisites

- An agent harness installed: [Kiro CLI](https://kiro.dev) or [Pi](https://pi.dev) (or both)
- Python 3.12+
- Node 20+
- Java (for PlantUML diagram generation)
- GitHub account + [GitHub CLI](https://cli.github.com/) (`gh`)
- Basic knowledge of Python, REST APIs, and React

## Setup

Clone the repo and run the setup script:

```bash
git clone https://github.com/LabSoft-AI/Agentic-SDLC-Software-Factory.git
cd Agentic-SDLC-Software-Factory
./setup.sh
source .venv/bin/activate
```

This creates a Python virtual environment and installs:

- PlantUML (for diagram generation)
- `pre-commit` and `detect-secrets`
- Pre-commit hooks:
  - Filesystem checks (trailing whitespace, YAML, JSON, merge conflicts)
  - Secret detection (with `.secrets.baseline`)
  - PlantUML SVG auto-generation from `.puml` files
  - Commit message format validation (`#<issue> <type>(<scope>): <description>`)
  - Direct commits to `main` blocked (must use PRs)
  - Docker test: builds image and runs tests before commit (when `Dockerfile` exists)

## Running a Cohort

This repository is the **baseline** and is marked as a GitHub template
repository. It holds course content only and never contains student data.
Each cohort runs in its own repository created from this one.

### Create a cohort repository

1. Create the cohort repo, named with the start date of the cohort:

   ```bash
   gh repo create LabSoft-AI/Agentic-SDLC-Software-Factory-<YYYY-MM-DD> \
     --template LabSoft-AI/Agentic-SDLC-Software-Factory \
     --private
   ```

2. Protect `main` in the cohort repo: restrict pushes and merges to the
   instructor, and block force-push and deletion.
3. Fill in the cohort-specific files, in the cohort repo only:
   - `COURSE-TRACKER.md` (location, time, dates, leaderboard, attendance,
     student directory)
   - `modules/*/ASSIGNMENTS.md` (one row per student)
   - `survey/COURSE-SURVEY.md` (survey form link)
4. Update `github_url` in `docs/conf.py` and the course documentation link
   in this README to point to the cohort repo.

### Keep the baseline clean

- Never commit student names, GitHub handles, grades, or survey results to
  the baseline.
- Improvements made during a cohort go to the baseline through a pull
  request, then are brought into running cohort repos.
- A repo created from a template shares no git history with the baseline, so
  a merge does not work. Use cherry-pick instead:

  ```bash
  git remote add baseline git@github.com:LabSoft-AI/Agentic-SDLC-Software-Factory.git
  git fetch baseline
  git cherry-pick <commit>
  ```

## References

- [Harness Engineering (Fowler)](https://martinfowler.com/articles/harness-engineering.html)
- [Context Engineering (Fowler)](https://martinfowler.com/articles/exploring-gen-ai/context-engineering-coding-agents.html)
- [Spec-Driven Development (Fowler)](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html)
- [arc42 Template](https://arc42.org/overview)
- [C4 Model](https://c4model.com/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Pre-commit Framework](https://pre-commit.com/)
- [Coding Dojo Kata Catalogue](https://codingdojo.org/kata/)

## Author

**Momcilo Krunic** — [LabSoft AI](https://labsoft.ai) — momo@labsoft.ai

## License

This work is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
You are free to share and adapt, with attribution to the original author.
