# Runtime environment variables not required

For **ubuntu-mise:dev** with the default **`/cache`** volume mount, consumers
(**compose `environment` / `env_file`**, `docker run -e`, cluster `config/cache.env`)
do **not** need to set the variables below for package managers to use the shared
cache layout.

Those settings are already:

1. **Baked into the image** (`Dockerfile` `ENV`, `/etc/profile.d/zz-cache-env.sh`)
2. **Persisted in user config files** under `home/` → `$HOME` in the container

Machine-readable list: **[runtime-env-not-required.yml](runtime-env-not-required.yml)**.

## Still set at runtime (examples)

| Variable | Why |
|----------|-----|
| `DATABASE_URL`, app secrets | App topology / vault — not package caches |
| `PORT`, `RAILS_RELATIVE_URL_ROOT` | Per-service compose |
| `TZ`, `IMAGE` | Host compose / identity (not gem caches) |
| `CACHE_ROOT` | Only if **not** `/cache` (and then rebuild or rewrite user configs) |

## Not required at runtime (package / tool cache)

| Variable | Covered by (primary) |
|----------|----------------------|
| **Bundler** | |
| `BUNDLE_PATH` | `~/.bundle/config` → `/cache/bundle` |
| `BUNDLE_CACHE_PATH` | `~/.bundle/config` → `/cache/rubygems` |
| `BUNDLE_CLEAN` | `~/.bundle/config` → `false` |
| `BUNDLE_NO_PRUNE` | `~/.bundle/config` → `true` |
| `BUNDLE_CACHE_ALL` | `~/.bundle/config` |
| `BUNDLE_CACHE_ALL_PLATFORMS` | `~/.bundle/config` |
| `BUNDLE_DISABLE_SHARED_GEMS` | `~/.bundle/config` |
| `BUNDLE_JOBS` / `BUNDLE_RETRY` | `~/.bundle/config` |
| **RubyGems CLI** | |
| `GEM_HOME` / `GEM_PATH` | `~/.gemrc` (`gemhome` / `gempath` → `/cache/gem`) |
| **mise** | |
| `MISE_DATA_DIR` | Image ENV + profile.d → `/cache/mise` |
| `MISE_CACHE_DIR` | Image ENV + profile.d → `/cache/mise-cache` |
| `MISE_CONFIG_DIR` | Image ENV; settings in `~/.config/mise/config.toml` |
| `MISE_RUBY_COMPILE` | Image ENV + mise config `compile = false` |
| `MISE_TRUSTED_CONFIG_PATHS` | Image ENV / profile.d → `/work` |
| `XDG_STATE_HOME` | Image ENV / profile.d → `/cache/xdg-state` |
| **Yarn** | |
| `YARN_CACHE_FOLDER` | `~/.yarnrc` / `~/.yarnrc.yml` → `/cache/yarn-cache` |
| `YARN_OFFLINE_MIRROR` | `~/.yarnrc` → `/cache/yarn` (classic 1.x) |
| `YARN_GLOBAL_FOLDER` | `~/.yarnrc.yml` → `/cache/yarn-global` (Berry) |
| `YARN_ENABLE_GLOBAL_CACHE` | `~/.yarnrc.yml` → `true` |
| **npm** | |
| `NPM_CONFIG_CACHE` / `npm_config_cache` | `~/.npmrc` → `cache=/cache/npm` |
| **Python** | |
| `PIP_CACHE_DIR` | `~/.config/pip/pip.conf` → `/cache/pip` |
| `UV_CACHE_DIR` | `~/.config/uv/uv.toml` → `/cache/uv` |
| `POETRY_CACHE_DIR` | `~/.config/pypoetry/config.toml` → `/cache/poetry` |
| `POETRY_VIRTUALENVS_IN_PROJECT` | Poetry config `in-project = true` (+ image ENV) |
| **Layout** | |
| `CACHE_ROOT` | Default `/cache` in image; user files hardcode `/cache/...` |

## Host sync

```bash
bin/ensure-host-package-config   # copy home/* templates → $HOME
task verify -- package-config    # container check
```

## Caveats

1. **User config paths are literally `/cache/...`.** Changing only `CACHE_ROOT` at
   runtime does **not** rewrite `.bundle/config` / `.npmrc` / etc. Rebuild the
   image or edit those files if you use a non-default root.
2. **Image ENV remains** as a backup and for login shells via profile.d. That is
   not “runtime compose injection” — it is build-time. Compose still should not
   need to re-list these for the default layout.
3. **App-specific** secrets and service env (`DATABASE_URL`, vault files) are
   unrelated and stay in compose / `env_file` as usual.

## Related

- [home/.bundle/config](../home/.bundle/config) and other [home/](../home/) templates
- [docker/cache-layout.env](../docker/cache-layout.env) — directory names under `/cache`
- `task verify -- package-config` — asserts files + dirs in a running container
