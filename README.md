## Context
pnpm version 10.9.0

In a pnpm monorepo with:
* Workspace
* Package patches

One would expect that if the `PNPM_VERIFY_DEPS_BEFORE_RUN=error` feature complains, it means that the lockfile isn't up-to-date, and we should run `pnpm install`, that would update the lockfile, and get things to work.

### Steps to reproduce
If pnpm's `lastValidatedTimestamp` is older than some other files (package.json files of internal packages?), the `PNPM_VERIFY_DEPS_BEFORE_RUN=error` feature will consider the project as 'needs pnpm install run',
even though the lockfile is up-to-date. When checking lockfile validity with this command:
```shell
pnpm install --frozen-lockfile
```

No change in lockfile is introduced, but this does update the `node_modules/.pnpm-workspace-state.json` file's `lastValidatedTimestamp` field, and with that fixes the issue `PNPM_VERIFY_DEPS_BEFORE_RUN=error` feature complained about.
So when running:
```shell
pnpm -w run x
```

### Current behavior
you see this error:
```
 ERR_PNPM_VERIFY_DEPS_BEFORE_RUN  Setting patchedDependencies of lockfile in ... is outdated
```
but there's nothing wrong with `patchedDependencies`, or with the lockfile as a whole.

### Expected behavior
When running `pnpm -w run x`, you should see `ERR_PNPM_NO_SCRIPT  Missing script: x` error, that means that the deps verification succeeded (and you're running a script that doesn't exist).

Instead you'd expect no error, as the lockfile is up to date, and when running `pnpm install --frozen-lockfile` it won't complain.
The state of internal untracked files shouldn't affect the `PNPM_VERIFY_DEPS_BEFORE_RUN=error` feature.
