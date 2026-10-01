# Pulling off a project in the Duitenberg Quant Committee?

*An opinionated guide to building Python-based quant development projects*

Welcome to the engineering onboarding guide for the Duitenberg Quantitative Finance Committee. Within our committee we mostly write code in Python and probably a bit of C++. This guide is specifically about writing Python. It is an overview of what I have learned over the last few years building our own research platform, plus my own Python experience.

Take whatever you think is useful, and treat (most) things as suggestions rather than something you have to follow. You do not have to share every opinion I have about which tools to use or SWE in general, but this is the way I prefer to work when working in Python.

Based on your current experience with coding, there are a few stages you can get into:

- **New to programming:** start with a smaller project, like loading price data from a CSV file ([exercise 1](exercises/01-prices-from-csv.md)).
- **Already a bit comfortable with Python:** you can get into writing, for example, a data loader adapter ([exercise 2](exercises/02-price-data-loader.md)).
- **Already building your own projects:** start working towards your first PR on one of our [committee projects](https://github.com/Duitenberg).

Each exercise is short and comes with a prompt for your coding agent: the agent sets up the layout of the project, you write the code yourself, and the agent reviews your work.

The sections are ordered from least to most experienced. The point is to help you be successful, or at least get familiar with the tools most people use to write software, without having to discover all of this yourself. Basically, the fundamentals.

## Operating system

Most of the projects I work on, I work on in Linux. I prefer Linux because it has much better tooling. AI agents are better at working in it, and most GitHub Actions runners and servers run on Linux. It is more widely used overall, and skills in a Linux terminal will help you further along in your career.

If you do not want to switch away from your Windows laptop, you can still use Linux through [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install). Installing WSL is easy nowadays and has first-party support. I use it every day and it works well:

```powershell
wsl --install
```

You can also just use Linux directly on any computer, which works great too.

The language of the Linux terminal is Bash. Every command in this guide is a Bash command, and the scripts in our projects (setup, lint and test scripts) are Bash scripts too. You do not need to master it, but learn the basics: moving around folders (`cd`, `ls`, `pwd`), reading files (`cat`, `less`), searching (`grep`, `find`), pipes (`|`) and environment variables. Agents also work through Bash all the time, so being able to read the commands they run is how you know what they are doing on your machine. [The Linux command line for beginners](https://ubuntu.com/tutorials/command-line-for-beginners) from Ubuntu is a good place to start.

## Version control: Git and GitHub

Git is the version control system itself: it runs on your machine and tracks every change to your code. GitHub is basically a wrapper around Git. It hosts your Git repositories and adds everything around them, like pull requests, issues, code review and GitHub Actions. There are a few other nice full-stack platforms around Git, such as GitLab and Bitbucket, but GitHub is the one I primarily use.

You probably already have a GitHub account, otherwise you would not be reading this. You have probably also set your identity already, but if not, you can do it like this:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Same with setting up an SSH key: [GitHub's guide](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent).

If you are new to Git, I recommend watching [How Git Works: Explained in 4 Minutes](https://www.youtube.com/watch?v=e9lnsKot_SQ) from ByteByteGo. It is a short four-minute video on repositories, branches and commits, and it links to a more extensive guide if you want to go deeper.

I really enjoy the ByteByteGo videos in general. They are a really good source of illustrations of pretty complicated system design topics. Watch the older videos especially: the newer ones are a little bit off, but the old ones are definitely really good.

## Python and dependencies: uv

Now we get to working in Python itself. Within Python you need something that can pin the dependencies of your projects. The standard tool is pip, which you are probably familiar with, but pip does not do exact dependency pinning, and that makes it hard to recreate a project on another machine.

[uv](https://docs.astral.sh/uv/) replaces pip (it even accepts the same `uv pip install` commands) and does exact dependency pinning through a lock file. That allows for a few things:

- Security scanning of all your dependencies becomes much easier.
- Everybody gets reproducible builds.
- Dependency clashes, where two packages need versions of a third that do not go together, show up immediately when you pin.

I use uv for installing all my Python libraries. You can install it like this:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv python install 3.13
```

Add packages with `uv add <package>`, not by typing version numbers into files yourself.

## Code editor

There are a lot of different code editors out there. I will not tell you which one to use, but I will tell you which one I prefer. I have tried a few. Over time I went from very heavy editors like Visual Studio to IntelliJ and a few others. The one I like most is [VS Code](https://code.visualstudio.com/), because you can shape and customize it to your own workflow, and it gives a good base to do that.

If you are more of a keyboard warrior, you can also get into Vim, but that mostly means working in a Linux terminal.

These are the extensions I use:

| Area | Extensions |
|---|---|
| Python | Python, Pylance, Python Debugger, Python Environments, Black Formatter, autoDocstring |
| Notebooks | Jupyter, Jupyter Keymap, Jupyter Notebook Renderers, Jupyter Cell Tags, Jupyter Slide Show |
| Git and GitHub | GitLens, GitHub Pull Requests, GitHub Actions |
| Coding agents | Claude Code, Codex |
| Containers and cloud | Docker, Container Tools, Bicep |
| Security | Aikido Security |
| Data | Rainbow CSV |
| Other languages | C/C++ Extension Pack, clangd, CMake Tools, LaTeX Workshop, Astro |

The coding agents I prefer to use inside VS Code, rather than through their web apps, are Codex and Claude Code. In the past I loved Copilot, back when you still paid per message instead of per token. That meant you could get the hell out of every agent request: you paid something like 8 cents for a request that burned through 5 or 10 million tokens, which was crazy.

## Project structure

Every Python project I make has a few things I always set up:

```
my-project/
  pyproject.toml
  uv.lock
  .gitignore
  .env.example
  src/my_project/
  tests/
  docs/
  .github/
  .vscode/
```

I will quickly go through the reasons:

- **`pyproject.toml`** is where you declare your dependencies and keep the configuration of all your tools.
- **`uv.lock`** is generated from the TOML by uv and pins the exact version of every package. You commit it.
- **`src/`**: I always use a source folder, so you can turn a project into a module that other people can reuse if necessary.
- **`tests/`**: I keep my tests out of the source folder and mirror the structure of `src/`.
- **`.env.example`**: within a project you want to keep the configuration out of the code itself, so that when deploying, for example, you can change the configuration without redeploying all of your code. The example file lists which settings exist, without the secret values.
- **`.gitignore`** is for everything you want to keep out of the repository itself, such as `.env`, `.venv` and caches.
- **`.github/`** is where quality assurance through CI/CD (continuous integration, continuous delivery) lives. Basically an automated test suite that tests your project in an environment similar to where it will be hosted; more on that under [GitHub Actions](#github-actions). In `.github/workflows` you define workflows, for example your deployments, your tests and auto-labeling of issues and pull requests, and with Dependabot you get automated dependency updates. GitHub workflows and automated testing make your own life a lot easier, through automated testing and automated maintenance of the dependencies your project uses.
- **`.vscode/`**: if you use VS Code, `extensions.json` lists the extensions you recommend for the project. In `settings.json` you can exclude certain files from the file watchers and the analysis, and change the type checking mode if you want to save some RAM. I like to keep my explorer clean, because I only want to see the files that are actually relevant to me. I hide all the caching files Python creates while running, like `__pycache__`, and keep generated files out of the watchers to save some performance on your computer.

```json
{
  "files.exclude": {
    "**/__pycache__": true,
    "**/.pytest_cache": true,
    "**/.mypy_cache": true,
    "**/.ruff_cache": true
  },
  "files.watcherExclude": {
    "**/.git/**": true,
    "**/.venv/**": true,
    "**/node_modules/**": true
  },
  "python.analysis.typeCheckingMode": "basic",
  "python.analysis.diagnosticMode": "openFilesOnly"
}
```

### Documentation and ADRs

Another thing I really like to do is keep a separate documentation folder. For every big decision within a project, I write an ADR (architecture decision record). If a decision is big or hard to reverse, you write a short document covering the context, the decision you made, and why you chose that specifically. For example: choosing a different database engine, a new data source, or a new programming language for certain components. Basically, anything that is worth writing down and cannot be retrieved from the code itself goes here.

When other people or AI agents are reasoning about decisions later, they can read back your documentation and see why you chose certain things in the past. That way you avoid repeating the mistakes of the past.

For every major component I also keep a README that explains:

- what is built
- what its entry points are
- how it is used
- the file structure
- extra information that is not obvious from just looking at the folder

### Full-stack projects

A full-stack project has a backend, with all the logic and APIs, and a separate frontend application that uses that API to present the data or give the user a way to interact with it, without having to hammer things into the command line. Basically, a GUI around your project.

If it is a full-stack project, I keep the frontend and backend code separate, and I use generated, typed API clients that you do not have to maintain yourself. The backend publishes an OpenAPI spec, and the frontend client is generated from it.

## Writing Python

### Learning Python itself

If you are new to Python, I recommend any 2, 3 or 6 hour YouTube course that covers all the different topics. They go over everything and explain all the different syntax. Play around and try things out as you are working through the topics. When you are learning, it is best to look at it, hear it, and practice it while you are looking at it. That way you get the full learning loop: seeing the functions explained, but also doing it yourself.

### Linting and naming

Within Python there is something called linting. You can write the same code in many different ways; a linter keeps your code readable and maintainable by enforcing one consistent way. I use [Ruff](https://docs.astral.sh/ruff/) for linting and formatting.

An example is variable naming. It is a much discussed topic and everybody is very opinionated about it. If you look at researchers' code, or the code that comes with papers, you will find a lot of short variable names. They write their code like math equations. In production, at real companies, people keep variable and function names as actual words that you can read in one go.

The rule: if you can read a function name and unambiguously understand what the function does, you wrote a good function name.

```python
# no
def calc(df, w, r):
    ...

# yes
def compute_portfolio_return(prices: pd.DataFrame, weights: pd.Series, risk_free_rate: float) -> float:
    ...
```

### Type annotations

Python is very loose. It does not require type annotations, and you are allowed to pass everything around as dicts. I really do not enjoy that, and it also makes your agents write very bad code. So for everything in Python, I keep it type-annotated: for any data structure you use, you make the type you want explicit.

```python
# no: every caller has to guess which keys exist
def load_quote(path: Path) -> dict:
    ...

# yes
@dataclass(frozen=True)
class Quote:
    ticker: str
    date: datetime.date
    close: Decimal
    volume: int

def load_quote(path: Path) -> Quote:
    ...
```

This helps you in a few different ways:

- Your agents write much better code.
- Your code editor gives you autocomplete on whatever you are writing. When the type is properly annotated, the editor knows what you are working with, and when you want to call a function on that type, it shows you the list of functions the type already has.
- Type checkers like Pyright (inside Pylance) and [mypy](https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html) can verify much more of your code before you ever run it.
- Most other languages, C++ definitely, require you to define the type. Making the type of whatever data structure you pass around explicit is a good habit, so that when you switch from Python to C++ or Go, you do not carry bad habits with you.

A good check to see if you are doing it well: if further down in the code you have to do type checks (`isinstance`) in the logic itself, you are not using type annotations correctly. Fix the type where the data comes from.

The configuration I use, from our research platform:

```toml
[tool.ruff]
target-version = "py313"

[tool.ruff.lint]
select = [
    "E", "W",   # pycodestyle
    "F",        # pyflakes: unused imports, undefined names
    "I",        # import sorting
    "B",        # bugbear: likely bugs
    "C4",       # comprehensions
    "UP",       # modern syntax for the target Python
    "ARG001",   # unused function arguments
    "T201",     # print() in committed code
]

[tool.mypy]
strict = true
```

### No default values for outside data

Something I have noticed and find very important: whenever data comes from outside the code, preferably do not use default values, or as little as possible.

What I mean is this. Say you are loading a dataset, and that dataset has a few holes or missing data because someone messed something up. That can happen. I prefer the code that loads it to throw an error when something is wrong with the data, rather than filling the missing data with placeholders. Otherwise, when you are analyzing it, it is much harder to know why certain numbers are off, especially when the numbers are not obviously wrong.

```python
# no: a missing price becomes 0.0 and your backtest makes up returns
close = row.get("close", 0.0)

# yes
if "close" not in row:
    raise ValueError(f"Missing close price for {ticker} on {date}")
close = Decimal(row["close"])
```

Default loading is very easy to do in Python, and if you use a lot of it, it becomes much less deterministic whether a value is a default or actually loaded from real data. That makes your debugging a living hell. If another component of your system relies on this component for its data, and that data is broken somewhere upstream, you have to go through all the different layers just to find out it was a default value all along.

### One function, one job

This is not really specific to Python, it is software engineering in general: I keep most functions to one job. Sounds obvious, but there are many different kinds of jobs, like fetching data, transforming it, splitting it and presenting it.

I really like the satisfying feeling of having a lot of small components that you call from a bigger function, and it all just works together. You can test each small component by itself, and the bigger function just becomes calling a bunch of components. If you go all the way up to your main file, the entry point of your program, you want to call one function, and that function calls all the different subsystems, where each subsystem can run and be tested on its own.

```python
def run_momentum_report(ticker: str, output_path: Path) -> None:
    """Build and save the momentum report for one ticker."""
    prices = fetch_daily_prices(ticker)
    clean_prices = drop_non_trading_days(prices)
    momentum = compute_momentum(clean_prices, lookback_days=90)
    write_report(momentum, output_path)
```

This gives a very maintainable structure and makes it easy for people to work together. Imagine that whole report was one function of 300 lines. One person is switching the price source from CSV files to an API, another is changing how momentum is calculated. They both have to edit the same function, their changes collide, and neither can test their part without running the whole thing. With the split above, each of them changes one small function and tests it on its own.

### A few smaller things

- For paths I use `pathlib`. For printing and formatting I use f-strings.
- Logging is required. In any project that will be hosted on a cloud server, or anywhere you cannot directly read the output, I use `logging`, because it gives you timestamps and a lot of other useful metadata that you do not want to write into every print statement yourself. It also gives you better telemetry and an overview of your project as it runs.
- `CamelCase` for classes, `snake_case` for functions and variables.
- For exception handling, when something breaks I catch the specific exception, rather than doing a catch-all `except Exception`.

### Docstrings

Within a file or module you have internal private functions and public functions. Public functions are the ones you present to other modules or to your main program, and they will probably be used the most by other systems.

When you write a function, you will have forgotten you wrote it in 3 or 4 months. You want to leave yourself a small breadcrumb: why you wrote the function, what the inputs are, what types they have, and very shortly what it does. That is where docstrings come in. Docstrings are Python's built-in documentation, basically comments attached to the function. For every argument, return value or error you raise, you write a short description of what it is or how it is used. Preferably write something better than just repeating the variable name, or add a bit of extra information.

I use the [Google style](https://google.github.io/styleguide/pyguide.html#38-comments-and-docstrings):

```python
def annualized_volatility(daily_returns: pd.Series, trading_days: int = 252) -> float:
    """Compute annualized volatility from daily simple returns.

    Args:
        daily_returns: Daily simple returns, one row per trading day.
        trading_days: Trading days per year used for scaling.

    Returns:
        Annualized standard deviation of returns.

    Raises:
        ValueError: If fewer than two returns are provided.
    """
```

### Further reading

- The [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) if you want to go a bit deeper.
- The [SOLID principles](https://en.wikipedia.org/wiki/SOLID) of programming.

## Before you commit

Before you commit or push any changes, there are a few checks you or your AI agents can run: linting, formatting, type checks and tests.

```bash
uv run ruff check --fix
uv run ruff format
uv run mypy src
uv run pytest
```

Please run your local test suite before committing, so you do not have to wait for CI to find out you broke something.

### Pre-commit hooks

Something I like to use is [pre-commit hooks](https://pre-commit.com/). A pre-commit hook is a script that runs before every commit. In it you can, for example, check that you are not uploading a 10 GB file to GitHub and fail the commit because of it, normalize line endings, or remove trailing whitespace. You can also use it to generate code graphs, run your local checks, or stop the commit when it does not pass certain checks or your test suite fails.

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v5.0.0
    hooks:
      - id: check-added-large-files
      - id: end-of-file-fixer
      - id: trailing-whitespace
  - repo: local
    hooks:
      - id: ruff-check
        name: ruff check
        entry: uv run ruff check --fix --exit-non-zero-on-fix
        language: system
        types: [python]
      - id: ruff-format
        name: ruff format
        entry: uv run ruff format
        language: system
        types: [python]
      - id: mypy
        name: mypy
        entry: uv run mypy src
        language: system
        pass_filenames: false
```

Install it once per repository with `uv run pre-commit install`.

### GitHub Actions

GitHub Actions is GitHub's CI/CD. It lets you run test suites, including more extensive ones, on a server: it rebuilds your whole project from scratch and runs all the tests, on a machine very similar to where the code will actually run. That checks that everything in the codebase still works as it did before, with your new change in it.

Believe it or not, your computer, where you write the code, will probably be slightly different from the place where the code will be hosted, or where the final version of the code will live. That difference is worth testing, and it is the main reason CI/CD exists. It lets other people, reviewers for example, see that your change passes the test suites, and it lets you set up the testing environment as close as possible to how the code will run on the actual server. For example, to check that you are not relying on some random operating system library that only exists on your computer.

```yaml
# .github/workflows/checks.yml
name: checks
on: [pull_request]
jobs:
  checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: astral-sh/setup-uv@v7
      - run: uv sync --frozen
      - run: uv run ruff check
      - run: uv run ruff format --check
      - run: uv run mypy src
      - run: uv run pytest -m "not integration"
```

## Testing

A test is an oracle that sits outside the implementation. It checks what the code is supposed to do, not how it happens to do it. So do not write tests that mirror the implementation itself. A test that repeats the code line by line passes when the code is wrong in exactly the same way.

I do not really like test-driven development. When you start writing code, it is pretty ambitious, maybe even delusional, to think you already know how all of the code will look, or what needs to be tested. There are other ways to develop. I prefer data-driven or domain-driven development: you first build the data types and the domain model for the system you are building. Once those are right, the code follows from them, and so does what you need to test.

Before writing the tests, I write down what the change actually needs in terms of testing, not what is easy to cover:

- **Behaviors:** every requirement of the change gets at least one test. "A missing close price raises an error" is a behavior.
- **Edge cases and failure modes:** boundary values, empty inputs, partial failures, bad responses from external APIs.
- **Integration surfaces:** where the change meets the rest of the system (the database, API contracts, queues, other modules), and how each of those seams gets exercised.
- **Explicitly not tested:** what is out of scope, and why that is acceptable.
- **Verification beyond tests:** run the affected flow end to end and look at the real output. Passing tests are necessary, not sufficient.

Tests that hit external services or the network get marked as integration tests, so the fast suite stays fast and runs offline:

```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
markers = [
    "integration: tests that hit external services or the network",
]
```

```bash
uv run pytest -m "not integration"   # fast, offline
uv run pytest -m integration         # external APIs
```

## Git workflow

A pull request (PR) is a request to merge the changes on your branch into the main branch, so other people can review them first.

When I work on a project by myself, my PRs can be as big as I want, because I am the only one responsible for them. When working with other people, it is smart to shape your PRs so someone else can actually review them.

For example, when you are building a certain function or submodule, every big change becomes one pull request. I do not care that much about the line count, especially when refactoring. What matters is whether the pull request is one story, one feature, rather than an ADHD brain where, while working, you go "oh, I can also fix this, and this, and that."

When I am working on something and find an issue that is worth fixing but is not what I am working on, I write a GitHub issue for the project. I come back to it later and decide whether it really needs fixing. If it needs fixing fast, I open a new [git worktree](https://git-scm.com/docs/git-worktree) based on the main branch I want to merge into, write the short fix there, merge it, and merge that back into my own branch.

If someone else is reviewing your work, it is nice to keep it to one thing.

For commit messages, I start with one word, for example `feat`, `fix`, `refactor` or `test`, followed by a short description of what you did in that commit:

```
fix: reject quotes with a missing close price
```

See every commit as one small update or upgrade of your project that you can refer back to if needed. Sometimes people cherry-pick certain commits into their own branch while they work. Even after your pull request is merged, they can still take specific commits out of it without merging everything else. So keep commits small, one change each.

## Working with coding agents

Depending on how experienced you are with coding: when starting out, I advise not to use coding agents very heavily. They write code very fast, but if you are unable to review their work or set the right boundaries and oracles, they will very quickly produce bad work that harms the project you are working on. The exception is a very quick proof of concept of a new idea. Go fast there, but assume the whole thing can be thrown away.

The rabbit hole of learning to work with coding tools is very, very deep. There are a ton of YouTube videos and guides on how to use them, but I recommend starting with the basic setup of [Claude Code](https://code.claude.com/docs) or [Codex](https://learn.chatgpt.com/docs). I use both, because they change every day and new models get released all the time.

### Rules

For most projects I give the agents rules. Most of the time I point them to `CONTRIBUTING.md` or the documentation for my coding standards. Anything agent-specific, or things agents normally struggle with, goes in `AGENTS.md`.

I like to keep all agent instructions in one place. Codex reads `AGENTS.md` by default, and for Claude I just point `CLAUDE.md` towards it, because I do not like updating multiple places for one thing:

```markdown
<!-- CLAUDE.md -->
@AGENTS.md
```

### Skills

Agents are more powerful when they have access to skills. A skillset I really like is [grill-me from Matt Pocock](https://github.com/mattpocock/skills); he has a wide set of skills I use. I also made my own skillset called ratchet, ask me about it.

### Prompting

The best prompts, for me, give the agent a specific goal to work towards, with very clear requirements and an oracle outside of itself that it can test against to see if it reached the goal. That way you can leave the agent running towards that goal within the limits and requirements you set, and you get everything you need from it for that project.

If the project or task is very large, make the agent write its goal and plan to a markdown file outside its own context. Then have it call another agent or subagent with a fresh context window, to review the current local work against the requirements as originally written down. That is usually the 80/20 of working with coding agents.

I still review all the code agents write when I care about the module or about its maintainability in the future. For a small UI feature or something similar, I sometimes skip the review if my whole testing pipeline already passes.

That only goes for my own projects. When you work on a shared project and open a pull request with agent-written code you did not review yourself, you are not saving time. You are handing the actual review, and the work of making the code correct, to the person who has to review it. That is a direct form of laziness and is generally not appreciated, here or anywhere you would work in the future! Review it yourself first, then ask someone else. In other words, respect people's time.

Something I really like about agents is that they can run end-to-end tests for you. They can set up a disposable database, run the end-to-end tests, and actually prove that whatever you are working on works all the way through to your original goal.

Whenever you use external APIs/integrations, make sure the agent takes the latest information from that API's documentation or source. If you let an agent work from memory, it will make a lot of mistakes. I made my own tool for this, [JevSeek](https://github.com/dej-h/jevseek), for finding API capabilities, using TypeSafe's new Jev model. Take a look if you want your agents to find APIs and their details faster.

It sounds obvious, but do not share your secrets, like API keys, with an agent. There is a good chance they get leaked, and you do not want someone maxing out your token bill, or whatever bill that key is linked to, or getting into your computer or Google account.

### If you are starting out

Like I said before: if you are still starting out with coding, I strongly encourage you to not only let agents write (all) your code. Try to understand what you are building and how it works (if you intend to get better in the field), before writing it yourself or using an agent to write some of it for you.

> *You do not want to be the person who can only write code as well as an agent can. Your skill ceiling becomes whatever an AI can make, and over time, leaning on it like that takes the joy out of creating/building as well (speaking from my own experience).*

The check I use for agent work: after you or the agent finished, can you explain the trade-offs you made? Why you chose a certain module, why you chose the abstractions you made, how you tested it, and why this part matters for the project? If you cannot, you are gambling. You closed your eyes, gave it a prompt, scrolled through the agent's summary and decided it was good enough.

## Debugging

I can go on for hours about debugging, but very simply:

1. First look at what the actual problem is, and see if you can reproduce it.
2. Add extra logging around the problem.
3. If that does not work, use the Python debugger and set breakpoints to see where the problem is.
4. If that still does not work, ask for help or have an AI agent look at it.

Whenever there is a problem with the code, I like to fix the fundamental problem for good rather than the symptom, even if it takes a bit longer. AI agents sometimes also tend to put a quick patch on the symptom, like a `try/except` or a default value that makes the error go away. That is putting a band-aid on a bullet wound: it looks fixed for now, but it will definitely cause issues later. So instead of patching around something you have control over, find the base of it and fix the real issue.

## Our research platform

The current research platform is a bit messy, because I have not cleaned it up in some time, but it uses most of the principles in this guide. It is the project we worked on last year.

The stack:

- **[FastAPI](https://fastapi.tiangolo.com/)** is the most common API framework for Python projects. It has really nice abstractions for building APIs and works very well with libraries like SQLAlchemy and Pydantic.
- **[SQLModel](https://sqlmodel.tiangolo.com/)** is an ORM (object-relational mapping), built on SQLAlchemy and Pydantic, that lets you do database modeling without having to learn all of SQL. To understand an ORM, you need to understand two things: the basics of **PostgreSQL** (tables, rows, objects and the different data types), and Python data modeling, typing your objects through Pydantic. Keeping those two separate can be annoying. Writing a model once, referencing it from your code, and having it map one-to-one to a table inside Postgres is pretty useful, and that is the reason I like to use an ORM. You still have to learn a bit of SQL for Postgres.
- **[LangChain](https://docs.langchain.com/)** for building agents. I enjoy it because it is model-agnostic and has a lot of functionality. It is not perfect, but it does most of the job. If you want graph-based agents, or a supervisor with subagents, **LangGraph** works well for that.
- **[pandas](https://pandas.pydata.org/)** for financial data. There is also an open-source library called **[edgartools](https://github.com/dgunning/edgartools)**, built specifically for SEC EDGAR data, which does better financial processing and filing document processing for us.
- **React, Vite and [Bun](https://bun.sh/)** for the frontend. Bun is a package manager similar to uv, but wider: it uses the npm packages you are probably already familiar with, and goes a bit further as a runtime and bundler too.

I have mostly talked about the backend, because I am mostly a backend/ML engineer. For frontends I cannot teach you as much, but I still recommend these tools, or finding your own approach and the tools you like.

### Docker and Docker Compose

Then reproducibility. For every project I will eventually host or put in prod, I use Docker and build a Docker image. A Docker image is a packaged snapshot of your application with its operating system, dependencies and code, so it runs the same on every machine. On top of that I use Docker Compose, which lets you combine different Docker images into services that run together. For example:

- a database service (PostgreSQL)
- a Redis cache service
- a file storage service (SeaweedFS, S3 compatible)
- a database viewer service (Adminer)
- a backend service
- a frontend service

You can run all of them together from one compose file:

```bash
docker compose up -d --build
```

## Conclusion

After reading through this guide, you have a decent understanding of the tools and fundamentals for building an application in Python, and of the standards most people and platforms use when working in Python. It only scratches the surface of software engineering, but it is a pretty good beginning to a wider view of the world of the projects you will be working in.

Just reading is not going to make you a better engineer, though. The real experience comes from actually trying, building and working with these tools. I encourage you to start playing around with your own projects, or try to contribute to the projects we already have.

Best of luck!

Dejan Honderd
