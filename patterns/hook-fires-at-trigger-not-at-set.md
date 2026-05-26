# Hook fires at trigger time, not at set time

**Trigger:** a config field that stores a shell command with a placeholder for some piece of runtime state.

When an app lets you configure a hook — "run this command when X happens" — the command string is stored at config time but evaluated at trigger time. If any piece of the runtime context that gets substituted into the command is attacker-influenced, the injection lands in the future, not in the moment.

The cheap mental model: a stored procedure that does `bash -c "process $FILENAME"` where `$FILENAME` is whatever filename happened to trigger the event. Setting the config is benign — the substitution hasn't happened yet. Saving a file with a hostile name is the actual attack. Reviewers, audit logs, even the app's own validation layer often only check the config at set time, when nothing dangerous has happened.

The same principle applies to:

- Image processing hooks (MotionEye's `on_picture_save`, ImageMagick policy callouts, exif post-processors).
- Mail filter rules that pipe to commands with envelope fields interpolated.
- CI/CD step definitions where a downstream variable is shell-interpolated into a run script.
- Webhook payload processors that build a shell command from JSON fields.
- Any "templated command" feature — the template is harmless; the data that fills it tomorrow is the payload.

Audit the substitution moment, not the storage moment. If user input ever reaches a position where a shell will see it, ask when — not whether — that shell will run.
