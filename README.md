# Staying Safe on GitHub

### A Student Training Guide

**Version 1.0 · For classroom use · No prior experience assumed**

---

## How to use this guide

This guide teaches you the habits that keep you, your classmates, and your future employers out of trouble on GitHub. Each section follows the same shape:

- **What it is** — plain explanation
- **Why it matters** — what actually goes wrong
- **How to do it** — commands and examples you can copy
- **Practice** — a small exercise

Work through it in order the first time. After that, use the [final checklist](#the-one-page-checklist) as a quick reference.

---

## Part 1 — The three rules that matter most

Everything else in this guide is detail. These three rules prevent almost every serious mistake a beginner makes.

### Rule 1: Never commit a secret

A **secret** is anything that proves who you are or unlocks something: passwords, API keys, access tokens, private keys, database connection strings, `.env` files.

**Why it matters.** Git remembers everything, forever. If you commit a password and then delete it in the next commit, the password is still sitting in the history. Anyone who clones the repo can run one command and read it. Even deleting the whole repository may not help if someone already copied it, and public repos are scanned by bots continuously — a leaked key is often abused within minutes.

**How to protect yourself.**

Create a `.gitignore` file *before* your first commit, and list anything secret:

```gitignore
# .gitignore
.env
.env.local
*.pem
*.key
credentials.json
secrets/
```

Then check what you are about to commit, every single time:

```bash
git status          # what changed?
git diff --staged   # what exactly is going into this commit?
```

**If you do leak something:**

1. **Revoke the secret immediately.** Go to the service and reset the password or rotate the key. This is the only real fix.
2. Tell your instructor or team lead. Do not hide it.
3. Only then clean the history, using `git filter-repo` or the BFG Repo-Cleaner.
4. Remember: cleaning history does *not* undo the exposure. Step 1 is the fix; step 3 is tidying up.

> **Common trap:** adding a file to `.gitignore` does not remove it if it is already tracked. You must also run `git rm --cached .env` and commit that removal.

---

### Rule 2: Never paste private information into an AI tool

AI coding assistants are useful, and you are allowed to use them. But treat the chat box like a public forum.

**Never paste:**

- passwords, keys, or tokens
- real customer or student data
- unpublished security fixes
- anything covered by a confidentiality agreement
- someone else's code that you do not have permission to share

**Why it matters.** Many AI services retain what you send them, and some use it to improve their models unless you opt out. Once it is sent, you cannot recall it. Different tools have different policies, and access to specific features varies by plan and changes often — so check the current terms rather than assuming.

**Safe habit:** replace real values with obvious fakes before pasting.

```
❌  "Here's my connection string: postgres://admin:Hunter2@prod.db.example.com/app"
✅  "Here's my connection string shape: postgres://USER:PASSWORD@HOST/DBNAME"
```

**If you use AI to help write code, say so.** A short line in your pull request is enough:

```
AI assistance: used <tool name> to draft the regex and the unit tests.
```

You are still the author. You still have to understand it, test it, and answer questions about it in review.

---

### Rule 3: Ask for the least permission you need

When you connect an app, a workflow, or a bot to a repository, it asks for permissions. Grant the smallest set that does the job.

**Why it matters.** Every extra permission is a door someone else could walk through if the tool is compromised or misconfigured. A spell checker has no business being able to delete your branches.

**How it looks in practice.** In a GitHub Actions workflow, you declare permissions explicitly:

```yaml
permissions:
  contents: read          # read files — needed
  pull-requests: write    # post a comment — needed
  # everything else: not granted
```

Notice what is *absent*: no `contents: write`. The job can read your code but cannot push to it.

> **Rule of thumb:** if a tool asks for write access and you cannot explain why it needs write access, do not grant it.

---

## Part 2 — Making your work reproducible

### Pinning dependencies

**What it is.** A dependency is code someone else wrote that your project uses. **Pinning** means recording the exact version you used, instead of accepting whatever is newest.

**Why it matters.** Imagine your project works today. Next Tuesday a library you depend on publishes a small update that breaks something. Now your project is broken and you changed nothing. This is called "works on my machine" — and it wastes enormous amounts of time.

**How to do it.** Keep two kinds of files:

| File | Who edits it | What it contains |
|---|---|---|
| `requirements.in` | You, by hand | Loose ranges, e.g. `httpx>=0.27,<0.28` |
| `requirements.txt` | A tool, never you | Exact versions with checksums, e.g. `httpx==0.27.2` |

Generate the locked file:

```bash
pip install pip-tools
pip-compile --generate-hashes --output-file=requirements.txt requirements.in
```

Install exactly what the lock says, refusing anything that does not match:

```bash
pip install --require-hashes -r requirements.txt
```

**The rule to remember:** never hand-edit a generated lock file. If it needs to change, change the `.in` file and regenerate. Commit the regeneration on its own, with a message like `chore(deps): recompile lock files`.

**Practice.** Take any small project you have. Write a `requirements.in` with one dependency. Generate the lock file. Look inside it — notice the `==` signs and the `--hash=` lines.

---

## Part 3 — Your name is your signature

### Understanding blame

**What it is.** `git blame` shows, for each line of a file, who last changed it and in which commit.

```bash
git blame README.md
git blame -L 10,25 src/main.py     # just lines 10 to 25
```

**Why it matters.** It is not an accusation — the name is unfortunate. Blame is how anyone answers "who wrote this, and why?" It is how you find the person who can explain a strange line, and how an auditor confirms that code was written by someone authorised to write it.

### Keeping one name for one person

People contribute from different machines and email addresses. Without care, one person shows up as five different names in the history.

**Fix it with a `.mailmap` file** in the root of your repository:

```text
# .mailmap
Ada Lovelace <ada@example.com> <ada.lovelace@old-school.example>
Ada Lovelace <ada@example.com> Ada Laptop <ada@localhost>
```

The format is: `Canonical Name <canonical@email> Other Name <other@email>`.

Now every one of those addresses is attributed to Ada.

### Ignoring reformatting commits

If you run a formatter across the whole project, that one commit touches every line and destroys the blame information.

**Fix it with `.git-blame-ignore-revs`:**

```text
# .git-blame-ignore-revs
# chore: apply prettier to entire repository
9c2f1a4e8b7d6c5f4e3a2b1c0d9e8f7a6b5c4d3e
```

Then tell git to use it:

```bash
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

### Signing off on your work

Some projects require a `Signed-off-by` line, which is a statement that you wrote the code or have the right to submit it.

```bash
git commit -s -m "feat: add login validation"
```

That `-s` adds:

```
Signed-off-by: Ada Lovelace <ada@example.com>
```

**Practice.** Run `git log --format='%an <%ae>' | sort -u` in a repository you have contributed to. Count how many distinct names appear. Are they all really different people?

---

## Part 4 — Teaching a spell checker your name

### The problem

Spell checkers flag anything not in their dictionary. Surnames, usernames, and organisation names are not in the dictionary. So a checker will happily tell you that "Okonkwo" is misspelled — and an aggressive autocorrect might silently "fix" it.

That is worse than annoying. It is an attribution error: you have just changed a person's name in their own work.

### The solution: authorship becomes the dictionary

Generate a list of protected terms from the people git already knows about, then tell the spell checker that those terms are correct.

The idea in four steps:

1. Read every author and committer name out of git history.
2. Read every `@handle` out of the `CODEOWNERS` file.
3. Read every canonical name out of `.mailmap`.
4. Write them all into a dictionary file, and mark that dictionary as **protected** — never corrected.

```bash
# 1. Get every author name ever seen in this repo
git log --format='%an%n%cn' | sort -u

# 2. Get every sign-off name
git log --format='%(trailers:key=Signed-off-by,valueonly)' | sort -u
```

Then point your spell checker at the result:

```jsonc
// cspell.json
{
  "dictionaryDefinitions": [
    {
      "name": "author-names",
      "path": "./.github/author-names.txt",
      "addWords": true
    }
  ],
  "dictionaries": ["author-names", "softwareTerms"],
  "protectFromCorrection": ["author-names"]
}
```

**Three things to remember:**

- The generated file carries a `DO NOT EDIT` header. To add a name, edit `.mailmap` and regenerate — otherwise your edit gets wiped next time.
- If a token is both a protected name *and* a known misspelling, **protection wins**. Never let autocorrect rename a person.
- Corrections should arrive as suggestions you accept, never as silent edits.

### Words that look like typos but are not

Project vocabulary is not misspelled just because a dictionary has never heard of it. Keep a separate, hand-maintained file for it:

```text
# .github/project-terms.txt
blacme
webhook
idempotent
sdkconfig
```

Keeping it separate from the generated list means regenerating the author names never erases your curated words.

### Symbols and short tokens

Not everything in a file is a word. Punctuation (`!`, `.`, `/`, `*`, `{}`, `[]`), escape sequences (`\n`, `\t`), hex numbers (`0x1f`), and one- or two-letter variables (`n`, `f`) should not be checked at all.

Most spell checkers skip short tokens by default — the common setting is a minimum word length of four characters. If you see false alarms on short tokens, raise that threshold:

```jsonc
{
  "minWordLength": 4,
  "ignoreRegExpList": [
    "0x[0-9a-fA-F]+",
    "/{1,2}[a-zA-Z0-9._/-]+"
  ]
}
```

**Practice.** Write a sentence containing your own surname and one deliberate typo. Run a spell checker over it. Does it flag the surname? Does it flag the typo? Now add your surname to a dictionary and run it again.

---

## Part 5 — Writing documentation people can trust

### Ambiguity is a bug

In a specification, an ambiguous sentence is a defect. Someone will build the wrong thing.

Watch for these patterns:

| Pattern | Weak | Strong |
|---|---|---|
| Vague quantity | "several contributors" | "five contributors: Ada, Grace, Alan, Linus, Ken" |
| Vague time | "soon" | "within 5 business days" |
| Vague reference | "see the docs above" | "see `docs/config.md`, section *Timeouts*" |
| Unsupported absolute | "always works" | "works for inputs under 1 MB" |
| Hidden requirement | "may be required" | "is required" or "is optional" |

The last one is the most dangerous. In a specification, "may be required" tells the reader nothing. Pick one.

**Practice.** Find one sentence in your own documentation containing "soon", "several", or "may be". Rewrite it so a stranger could act on it without asking you a question.

---

## Part 6 — Validating input you did not write

Any time your automation accepts input from a human or another system, that input can be wrong, malformed, or hostile.

**The rule:** validate before you act.

```python
ALLOWED_MODES = {"auto", "full", "client-only", "skip"}

def validate_mode(value: str) -> tuple[bool, str]:
    if value not in ALLOWED_MODES:
        return False, f"Unknown mode: {value}"
    return True, ""
```

Three principles:

1. **Allow-list, don't deny-list.** List what is permitted. Anything else is rejected. A deny-list always misses something.
2. **Reject, don't repair.** If a file pattern contains shell characters like `;` or `$(`, fail loudly. Silently stripping them produces a confusing, misleading result.
3. **Fail closed.** On invalid input, stop. Do not continue with a guess.

### Naming your inputs clearly

When you define inputs for an automated task, name them so nobody has to guess:

```yaml
on:
  workflow_dispatch:
    inputs:
      blame_mode:
        description: 'Which lines to attribute: auto, full, client-only, or skip'
        required: true
        type: choice
        options: [auto, full, client-only, skip]
        default: auto
```

Give every input a description, a type, and a default. A bare input name with no description is a support ticket waiting to happen.

---

## Part 7 — The daily routine

### Before you start

```bash
git fetch origin
git switch -c feat/short-description origin/main
```

Work on a branch. Never commit straight to `main`.

### While you work

```bash
git status                 # what have I changed?
git diff                   # what exactly did I change?
git add -p                 # stage in small, logical pieces
git commit -m "feat(scope): describe the change"
```

Commit messages follow a pattern: `type(scope): description`.

| Type | Use for |
|---|---|
| `feat` | a new capability |
| `fix` | a bug fix |
| `docs` | documentation only |
| `chore` | tooling, dependencies, CI |
| `refactor` | restructuring with no behaviour change |
| `test` | adding or fixing tests |

### Before you push

```bash
git diff --staged          # final look — any secrets in here?
git log --oneline -5       # do the messages make sense?
```

### Opening the pull request

A good description answers four questions:

```markdown
## What changed
Added input validation to the review workflow.

## Why
Invalid patterns caused confusing, misleading reports.

## How I tested it
Ran the validator against 12 good and 6 bad inputs. All behaved correctly.

## AI assistance
Used a coding assistant to draft the regex; verified and rewrote by hand.
```

### During review

- Respond to every comment, even if only to say "done" or "I disagree because…".
- Keep pull requests small. Under about 400 changed lines is a good target. Large PRs get rubber-stamped, which defeats the purpose.
- Prefix unimportant suggestions with `nit:` so authors know what actually blocks the merge.

---

## Part 8 — Working with automation safely

Automation runs with the permissions you give it. Two habits keep it safe.

**First, give it the minimum.** Declare permissions explicitly in every workflow:

```yaml
permissions:
  contents: read
```

If you leave this out, the workflow may inherit broad default permissions. Being explicit is safer and self-documenting.

**Second, give it enough history when it needs history.** Tools that inspect authorship need the full commit history:

```yaml
- uses: actions/checkout@v4
  with:
    fetch-depth: 0        # 0 means "all history"
```

Without this, the checkout is shallow — only the most recent commit. Any tool looking for author names will find almost nothing, and you will get a flood of false positives that look like a bug in the tool but are actually a configuration mistake.

**Third, degrade gracefully.** If a permission is missing, say so rather than failing silently:

```jsonc
{
  "status": "partial",
  "degraded": true,
  "reason": "pull-requests:write not granted; report uploaded as artifact instead"
}
```

---

## Part 9 — Common mistakes and their fixes

| Symptom | Likely cause | Fix |
|---|---|---|
| Password visible in repo history | Committed before `.gitignore` existed | Rotate the secret, then clean history |
| `.gitignore` seems ignored | File was already tracked | `git rm --cached <file>` and commit |
| Build breaks with no code change | Unpinned dependency updated | Pin versions, use a lock file |
| Same person listed five times in blame | Multiple email addresses | Add `.mailmap` entries |
| Blame points at a formatting commit | Reformat swept the repo | Add the commit to `.git-blame-ignore-revs` |
| Spell checker flags contributor names | Names not in a dictionary | Generate a protected-terms list from git |
| Autocorrect renamed someone | Protection not applied | Mark the author dictionary as protected |
| Automation finds no author names | Shallow clone | Set `fetch-depth: 0` |
| Confusing automation results | Invalid input silently repaired | Reject invalid input instead |
| Reviewer approved without reading | Pull request too large | Split into smaller PRs |

---

## The one-page checklist

### Every commit

- [ ] `git status` and `git diff --staged` reviewed
- [ ] No secrets, keys, or `.env` files included
- [ ] Commit message follows `type(scope): description`
- [ ] Committing to a branch, not `main`

### Every pull request

- [ ] Describes what changed, why, and how it was tested
- [ ] AI assistance disclosed, if used
- [ ] Under ~400 changed lines
- [ ] Tests included for new behaviour
- [ ] Lock files regenerated if dependencies changed

### Every repository

- [ ] `.gitignore` covers secrets from day one
- [ ] `.mailmap` collapses alternate identities
- [ ] `.git-blame-ignore-revs` lists formatting sweeps
- [ ] `CODEOWNERS` names who owns what
- [ ] Dependencies pinned with a lock file
- [ ] Workflows declare minimum permissions
- [ ] Protected-terms dictionary generated, not hand-edited

### Every time you use an AI tool

- [ ] No secrets, real data, or confidential code pasted
- [ ] Real values replaced with obvious placeholders
- [ ] Output reviewed, tested, and understood before committing
- [ ] Use disclosed in the pull request

---

## Glossary

| Term | Meaning |
|---|---|
| **Blame** | Per-line record of who last changed each line |
| **Branch** | An independent line of work, so `main` stays stable |
| **Clone** | A local copy of a repository, including its history |
| **CODEOWNERS** | A file naming who must review changes to which paths |
| **Commit** | A saved snapshot of changes, with a message and an author |
| **Dependency** | Code written by someone else that your project uses |
| **Lock file** | A record of the exact dependency versions you used |
| **`.mailmap`** | A file mapping several identities to one canonical person |
| **Pin** | To fix a dependency to one exact version |
| **Pull request** | A proposal to merge your branch, with discussion and review |
| **Secret** | Anything that proves identity or grants access |
| **Shallow clone** | A copy with truncated history |
| **Sign-off** | A `Signed-off-by` line asserting you may submit the code |
| **Upstream** | The original repository you forked from |

---

## Where to go next

- Practise on a throwaway repository first. Break things there.
- Read the `CONTRIBUTING.md` of a project you admire. Notice how much of it is about safety and clarity.
- When you join a team, ask three questions: *Where do secrets live? Who reviews what? What is our branch policy?*

---

*Training guide version 1.0 · Prepared 29 October 2026 · Suitable for secondary, undergraduate, and workplace induction use*
