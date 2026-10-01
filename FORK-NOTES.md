# Backup fixes in this fork

The `codex/backup-quota-retries` branch retries transient per-message batch
failures in new batches with exponential backoff, up to ten batch attempts.
Successful downloads are committed before retrying or stopping. Permanent
per-message failures stop the backup after successful messages are committed.

The local quota table counts `messages.get` as 20 units. The Gmail bucket holds
100 units and replenishes 60 units per second. This is a conservative local
setting, not a guarantee that quota errors cannot occur; our live runs still
received quota errors, and Google's reported usage differed from local accounting.

Add `--debug-backup` to your normal source command for timestamped batch sizes,
accounted units, token waits, active limiter settings, message errors and retries.
Without the flag, detailed diagnostics are disabled; normal backoff notices and
non-quota errors remain visible. This flag is separate from the existing `--debug`.

## Updating from the original repository

With a clean working tree, run from this checkout:

```bash
git switch codex/backup-quota-retries
git fetch upstream
git merge upstream/main
git push origin codex/backup-quota-retries
```

If Git reports conflicts, resolve them and commit the merge before pushing.
The original repository is `upstream`; your fork is `origin`.

Continue using your existing config and backup folders outside the checkout.
Source changes do not update the packaged `gyb` executable.

Individual imports in `--action restore` commit their resume records after each
message, including the default batch size of one. An interruption between Gmail
accepting an import and the local commit can still cause that message to be
imported again on the next run.
