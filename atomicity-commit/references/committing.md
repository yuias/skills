# Committing cleanly

## Default: stacked `-m`

Each `-m` becomes its own paragraph. Literal newlines inside a double-quoted argument are preserved:

```
git commit \
  -m "feat(uploader): retry S3 uploads with exponential backoff" \
  -m "Transient 503s from S3 were failing ~2% of CI runs. The 5-attempt
cap keeps a genuine outage failing the build within a minute."
```

## Fallback: `-F -` when quoting gets awkward

If the message contains backticks, `$`, or nested double quotes, pass it on stdin with a quoted heredoc; `<<'EOF'` stops the shell from interpreting them:

```
git commit -F - <<'EOF'
fix(config): read $HOME before falling back to `pwd`

The previous order silently picked up the wrong root under sudo.
EOF
```

## Fixing a bad message

If `git log -1 --format=fuller` shows a wrong wrap, escaping artifacts, or a missing footer, amend it:

```
git commit --amend -m "<subject>" -m "<body>"
```

Amend only while the commit has not been pushed to a shared branch.
