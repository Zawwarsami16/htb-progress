# Two-stage supply-chain RCE

**Trigger:** a target has an auto-update cronjob pointing at a package registry you can publish to.

The basic chain is well known. Publish a malicious package version with a `preinstall` hook that runs your code. The cronjob installs it. Your code runs.

The detail that matters in practice is staying alive after the first install. If your malicious package is a stub, the target application crashes when it imports it. A crashed application means a broken exfil channel, which means the secret you just stole has nowhere to land.

The fix is two stages.

Stage one is the exfil package. It contains the original module exports plus your `preinstall` hook, so the application keeps working after the install. The `preinstall` writes your loot to a known path inside the container.

Stage two, if stage one's package broke something or you had to rush it out as a stub, is the recovery package. It contains the real exports plus the same `preinstall`. The cronjob updates to it. The application comes back up. You read your loot through whatever exfil channel you had before the crash.

You read the loot through the same surface you used to leak the registry token in the first place. Often a file-SSRF on the application. The channel and the application are the same thing. If you crash the application, you lose the channel.

**Seen in:** [Prison Pipeline](../challenges/misc/prison-pipeline.md).
