# Deployment

| | |
|---|---|
| **Production** | https://tebo.dev |
| **Staging** | none — deliberately. See below. |
| **Default branch** | `master` |

Hosted on a self-managed VPS behind Caddy, which terminates TLS and serves the
files directly. Host provisioning lives in a separate private repository, not
here.

## How a deploy happens

```
push to master  ->  production
```

One environment, on purpose. This is a static resume page with no build step,
no database and no form: the blast radius of a bad commit is a page that looks
wrong until the next push. Sites where that is not true — anything with a form,
a client, or a database — get a staging instance instead.

```bash
git push    # live within a minute
```

`.github/workflows/deploy.yml` rsyncs the checked-out tree with `--delete`, so
removing a file removes it from the site.

## What ships, and what does not

`css/` is committed **already compiled**, so there is nothing to build at deploy
time. The gulp toolchain that produces it is source, not content.

| ships | excluded |
|---|---|
| `index.html`, `selkicon.ico` | `scss/`, `gulpfile.js`, `package*.json` |
| `css/`, `img/`, `js/` | `node_modules/`, `*.md`, `.github/` |
| `vendor/` — Bootstrap, FontAwesome, jQuery | |

`vendor/` must ship: `index.html` loads Bootstrap, FontAwesome and jQuery from
it. If you ever regenerate `css/`, run gulp locally and **commit the output** —
the server does not build.

The `*.md` exclusion is a shape rather than a list of filenames, so a document
added here later cannot end up in the document root by accident. An enumerated
list is correct only until someone adds a file.

## Rules

- **Never deploy by SSHing as root**, and never `git pull` on the server. CI
  rsyncs the tree it checked out; the box holds no GitHub credentials and needs
  no network reach to GitHub.
- **Never edit files on the server.** `--delete` means the next deploy erases
  them and the change exists nowhere else.
- **This repository is public.** Nothing secret belongs in it — no keys, no
  tokens, no host detail beyond what DNS already reveals. Deploy credentials
  live in GitHub Actions secrets.
- **`www.tebo.dev` is not served.** It exists as a DNS CNAME but has no
  certificate and returns nothing, which was also true before the migration. Add
  it deliberately if you want it, rather than assuming it works.

## Verifying

```bash
curl -sI https://tebo.dev/ | head -1
curl -s -o /dev/null -w '%{http_code}\n' https://tebo.dev/vendor/bootstrap/css/bootstrap.min.css
```

There is no `Server` header — Caddy is configured to strip it. If you see
`Server: Apache`, you have reached the old host, which usually means a stale
local DNS cache rather than a real problem.
