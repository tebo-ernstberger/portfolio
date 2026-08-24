# CLAUDE.md

tebo.dev — a static resume page. Bootstrap + jQuery, `css/` compiled from
`scss/` by gulp and **committed**, so there is no build step at deploy time.

## Read DEPLOYMENT.md before touching anything that ships

[`DEPLOYMENT.md`](DEPLOYMENT.md) has the target URL, the trigger and the rules.
The short version:

- Production is https://tebo.dev
- The default branch is **`master`**, not `main`
- `push to master` -> **production, immediately**. One environment, deliberately.
- `vendor/` ships (Bootstrap, FontAwesome, jQuery); `scss/` and the gulp
  toolchain do not.

## Hard rules

- **This repository is public.** Nothing secret goes in it.
- **Never SSH as root to deploy**, never `git pull` on the server, never edit
  files on the box — the next deploy erases them.
- **If you regenerate `css/`, commit the output.** The server does not build.
