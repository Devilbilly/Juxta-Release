# Juxta

## What Juxta is

Juxta turns a change described in a GitHub issue into a reviewed pull request.
Agent workers implement the change on a branch, run the repository's tests, and open a PR.
An independent reviewer checks the result. You verify it and reply `verified` on the
issue to authorize the final checks and merge. If someone else filed the issue,
both the filer and the repository owner must sign off.

## Requirements

- Linux x86_64: the platform supported by this executable.
- `git`: clones, branches and worktrees for the repository being changed.
- `gh`: authenticated GitHub CLI login with write access to your target repository,
  used to read issues, open PRs, post reports and merge approved changes.
- At least one agent runtime installed and logged in: Claude Code or Codex CLI,
  used to implement changes. Keep its login available to the account running Juxta.
  Agent usage follows that runtime's account and billing terms.

## Download, verify and install

Download the Linux x86_64 executable, SHA256SUMS and KEY_ID from this repository's Releases page.
The signing key id is `1deec3d9d5dcbef8`; its public key is in `release.pub`.
Compare KEY_ID with the signing key id above using a trusted copy of this repository.
The checksum detects download corruption; the executable also verifies its signed payload.
Replace `<version>` and `<platform>` with the downloaded filename's values.
Keep the executable and checksum file in the same directory, then run:

```sh
sha256sum -c SHA256SUMS
chmod +x juxta-<version>-<platform>
./juxta-<version>-<platform> verify
./juxta-<version>-<platform> install --home "$HOME/juxta-home"
```

`sha256sum -c` checks every file listed; download those files too, or check only the
line for your executable. Installation prints the installed release directory.
Use its `juxta` launcher below (add that directory to PATH, or use its full path).

## Connect a repository

Start with an existing clone of your GitHub repository. Save this as `project.json`.
Replace the GitHub login/repository and absolute paths with your own values.
Keep the Juxta home separate from the application checkout. This example is a
Python/pytest project: set `setup` to install its dependencies in fresh worktrees,
and `test` to your full test command; `[]` assumes dependencies are already available.
Other test runners need a `runner` configuration rather than a pytest command.

```json
{
  "name": "project",
  "slug": "YOUR_GITHUB_LOGIN/YOUR_REPOSITORY",
  "owner": "YOUR_GITHUB_LOGIN",
  "home": "/absolute/path/to/juxta-home",
  "checkout": "/absolute/path/to/project",
  "setup": [],
  "test": ".venv/bin/python -m pytest tests/ -q",
  "workers": {"max": 1}
}
```

Preview the connection plan, then apply it:

```sh
juxta setup project.json --dry-run
juxta setup project.json
```

Setup connects the repository, prepares state and issue-template proposals, installs
a watchdog, and starts its daemon. Read its output and merge the issue-template PR.
Use the same home path when managing that daemon:

```sh
juxta daemonctl --home "$HOME/juxta-home" status
juxta daemonctl --home "$HOME/juxta-home" stop --keep-workers
juxta daemonctl --home "$HOME/juxta-home" start
```

Append `--dry-run` to start, stop or status to print a plan without accessing GitHub
or changing the home. `stop --keep-workers` stops polling but lets active workers finish;
plain `stop` also stops workers. The watchdog can restart a stopped daemon: use
`juxta setup project.json --stop` to stop workers and remove its watchdog for maintenance.

## Daily use

1. File a GitHub issue using the installed template. Describe the desired behavior,
   reproduction steps or input, and what should count as done.
2. Juxta posts a completion report, test evidence and a PR link. Its status labels
   show progress: InProgress while working, NeedInput when details are missing,
   and VerificationRequired when the result is ready for you to check.
3. Open the PR and follow the report's verification steps. To send work back,
   reply on the issue with what is wrong, the expected result and reproduction details.
   The worker uses that feedback to revise the PR.
4. When satisfied, reply on the issue with a standalone `verified` line.
   If the report has checklist items, also reply `id N verified` for each checked item.
   Juxta performs final gates before merging; a failing gate prevents the merge.

For public static web targets, opt in with `"preview": {"kind": "pages", "root": "."}`
in your config. Merge setup's preview workflow PR and select GitHub Actions as the
repository's Pages Source. The worker's Review line then links to a browser preview
of the PR, with its PR number and commit banner. Check that result before signing off.
Private repositories and backend applications do not have this browser preview.

## Update

Download and verify the new file as above. From an installed launcher, install it
into the existing home (replace `<downloaded-file>` and `<home>`):

```sh
juxta release install <downloaded-file> --home <home>
```

Installation stages the release; it does not switch the running daemon. Use
`juxta daemonctl --home <home> deploy <downloaded-file>` to activate it through the
worker-drain barrier, or `juxta daemonctl --home <home> rollback` to return to the
previous release. Active workers keep their pinned release until their round ends.

## Help

Setup proposes issue templates in your target repository. Use those issue templates
for bug reports and feature requests: they list the input and acceptance details
workers need. For a setup problem, include the failing command and its output in
an issue here, with credentials and private data removed.
