# NOTES.md

## What's in CLAUDE.md and what I left out

I kept it to four short sections: a one-line description, Commands, Conventions, and Architecture. Every command and convention listed is something I verified against the actual code — the five npm scripts match package.json, the routing/data-access conventions match how server.js, routes/, and db/store.js are actually wired together, and I added two smaller notes (the PORT env var behavior from server.js/.env.example, and the ESLint no-unused-vars exemption for req/res/next/_ from .eslintrc.json) because both are facts you can only get by reading two files together, not from glancing at either one alone.

I deliberately left out: the CI workflow details (.github/workflows/ci.yml), since it just re-runs lint and test, which are already documented commands — restating it wouldn't tell Claude anything new about how to work in the repo. I also skipped a file-by-file listing of routes/ and tests/ (that's one ls away), any of the course/assignment scaffolding from README.md (submission steps, grading checklist — not relevant to writing code), and generic advice like "write tests" or "handle errors," since those aren't specific to this project and just add noise. The goal was a file where every line teaches Claude something it couldn't get faster by reading the code itself.

## Permission rules: what I added, and what the deny rule protects against

In .claude/settings.json I added one "allow" rule for Bash(npm test:*), since running tests is safe, has no side effects on shared state, and is something I want Claude to be able to do without asking every time. I added one "ask" rule for Bash(git push:*), so Claude always confirms with me before pushing — a push affects a shared remote, so I want a human checkpoint even though it's not outright dangerous. For deny, I blocked reading .env*, *.pem, *.key, and anything under secrets/.

The deny rule is basically a hard stop. In Claude Code’s permission model, it’s different from "ask". An ask rule just pauses and waits for someone to approve it. A deny rule blocks the action completely.

So if Read("**/.env*") is denied, Claude simply cannot read those files. There isn’t an override button or a way around it because the whole purpose is to take the judgment call out of the situation.

That matters with files like .env because they are commonly used to store things you do not want exposed, such as database credentials, API keys, tokens, and session secrets. In this starter project, .env.example may only contain something harmless like a placeholder PORT, but that can change very quickly. As soon as someone connects a real database, API, or service, the actual .env file may contain sensitive credentials.

The same goes for .pem and .key files. Those can contain private cryptographic keys used for things like SSH, TLS certificates, or signing. There really isn’t a good reason for an AI assistant to read the contents of those files during normal development work.

The main concern isn’t that Claude would intentionally leak anything. The more realistic problem is accidental exposure during normal debugging.

For example, if an environment variable is not working, a perfectly reasonable troubleshooting step would be to check the .env file and see what value is actually there. If that file is readable, the secret is now part of the conversation context. From there, it could accidentally show up in a response, get copied into troubleshooting notes, or end up in something like a commit message or pull request description. It could also remain in conversation history or logs.

None of that requires anyone to do anything malicious. It only takes one unnecessary read of the file.

That’s also why I think deny makes more sense here than ask. With an ask rule, you still have to notice the permission request and make the right decision every time. During a long debugging session, it would be easy to approve something without thinking much about it.

A deny rule removes that possibility completely. The sensitive file simply cannot be read, which is exactly what you want for files where one accidental approval could expose credentials or private keys.
