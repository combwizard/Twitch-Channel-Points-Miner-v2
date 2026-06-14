# Configuration reference

Most settings live in [`config.example.yaml`](../config.example.yaml). Copy it to `config.yaml` and edit.

Bet strategies, notifications, and per-streamer Python objects are still configured in [`example.py`](../example.py).

## YAML layout

| Section | Purpose |
| --- | --- |
| `username` | Twitch login name (required) |
| `password` | Optional; omit to use TV device login on first run |
| `miner` | Hermes, priorities, analytics collection, SSL, `@` matching |
| `logger` | Console/file logging, rotation, emoji, colors |
| `streamer_settings` | Defaults applied to every streamer |
| `analytics` | Optional Web UI (`enabled`, host, port, refresh) |
| `streamers` | Channel login names to mine (list order matters) |
| `mine` | Follower list mode, sort order, blacklist |

Example mapping: Python `use_hermes=True` → YAML `miner.use_hermes: true`. Python `Priority.STREAK` → YAML `miner.priority: [streak, drops, order]` (case-insensitive).

When using the Python API alongside YAML, settings resolve in this order (highest wins):

1. Per-streamer settings passed to `mine()`
2. `StreamerSettings` on the `TwitchChannelPointsMiner` instance
3. Built-in defaults

## Miner options (`miner`)

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `use_hermes` | bool | `True` | Use the Hermes WebSocket API instead of legacy PubSub |
| `enable_analytics` | bool | `False` | Collect point history for the Web UI charts |
| `claim_drops_startup` | bool | `False` | Claim all inventory drops when the miner starts |
| `disable_ssl_cert_verification` | bool | `False` | Disable TLS verification (use only to work around cert errors) |
| `disable_at_in_nickname` | bool | `False` | Match chat mentions without a leading `@` |
| `priority` | list | `[STREAK, DROPS, ORDER]` | Streamer selection order when multiple are live |

## Priority values

| Value | Behavior |
| --- | --- |
| `STREAK` | Catch watch-streak bonuses first; frees watch slots when a channel no longer needs streak progress |
| `DROPS` | Prioritize streamers with active drop campaigns |
| `SUBSCRIBED` | Prioritize subscribed channels (higher tiers first) |
| `ORDER` | Follow the streamer list order |
| `POINTS_ASCENDING` | Lowest balance first |
| `POINTS_DESCENDING` | Highest balance first |

Combine priorities as needed. Include at least one of `ORDER`, `POINTS_ASCENDING`, or `POINTS_DESCENDING` so the miner keeps watching after streaks and drops are handled.

## `mine` options

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `followers` | bool | `false` | Load your Twitch follow list instead of `streamers` |
| `followers_order` | str | `asc` | `asc` or `desc` by follow date when `followers` is true |
| `blacklist` | list | `[]` | Usernames to skip (also applies to the followers list) |

## Streamer defaults (`streamer_settings`)

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `make_predictions` | bool | `True` | Place channel-point predictions |
| `follow_raid` | bool | `True` | Follow raids for bonus points |
| `claim_drops` | bool | `True` | Count watch time toward drop campaigns |
| `claim_moments` | bool | `True` | Claim Moments when available |
| `watch_streak` | bool | `True` | Prioritize channels with an active watch streak |
| `community_goals` | bool | `False` | Contribute max points to community goals |
| `chat` | `ChatPresence` | `ONLINE` | IRC presence: `ALWAYS`, `NEVER`, `ONLINE`, `OFFLINE` |
| `bet` | `BetSettings` | - | Prediction behavior (Python only; see below) |

## BetSettings (Python only)

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `strategy` | `Strategy` | `SMART` | Outcome selection strategy |
| `percentage` | int | `5` | Bet this % of your balance |
| `percentage_gap` | int | `20` | Gap threshold for `SMART` strategy |
| `max_points` | int | `50000` | Cap on bet size |
| `minimum_points` | int | `0` | Skip bets below this balance |
| `stealth_mode` | bool | `False` | Bet just under the current leader |
| `delay_mode` | `DelayMode` | `FROM_END` | How `delay` is interpreted |
| `delay` | float | `6` | Seconds or fraction (see below) |
| `filter_condition` | `FilterCondition` | `None` | Optional gate before placing a bet |

**Strategies:** `MOST_VOTED`, `HIGH_ODDS`, `PERCENTAGE`, `SMART_MONEY`, `SMART`, `NUMBER_1` … `NUMBER_8`

**Delay modes:**

- `FROM_START`: wait `delay` seconds after the prediction opens
- `FROM_END`: place the bet `delay` seconds before it closes
- `PERCENTAGE`: place when `delay` fraction of the timer has elapsed (e.g. `0.2` = 20%)

**FilterCondition**: gate bets with `by` (`TOTAL_USERS`, `TOTAL_POINTS`, `ODDS`, `PERCENTAGE_USERS`, `ODDS_PERCENTAGE`, `TOP_POINTS`), `where` (`GT`, `LT`, `GTE`, `LTE`), and `value`.

## Logger (`logger`)

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `save` | bool | `True` | Write logs to `logs/` |
| `console_level` | level | `INFO` | Terminal verbosity |
| `file_level` | level | `DEBUG` | File verbosity |
| `less` | bool | `False` | Compact log format |
| `colored` | bool | `True` | ANSI colors in terminal |
| `emoji` | bool | platform | Disable on Windows if output breaks |
| `console_username` | bool | `False` | Prefix lines with account name |
| `auto_clear` | bool | `True` | Rotate logs daily, keep 7 days |
| `time_zone` | str | `""` | tz database name, e.g. `America/Denver` |
| `color_palette` | `ColorPalette` | - | Per-event terminal colors (Python) |
| `telegram` / `discord` / `webhook` / `matrix` / `pushover` / `gotify` | - | `None` | Notification backends (Python) |

**Notification events:** `STREAMER_ONLINE`, `STREAMER_OFFLINE`, `GAIN_FOR_RAID`, `GAIN_FOR_CLAIM`, `GAIN_FOR_WATCH`, `BET_WIN`, `BET_LOSE`, `BET_REFUND`, `BET_FILTERS`, `BET_GENERAL`, `BET_FAILED`, `BET_START`, `BONUS_CLAIM`, `MOMENT_CLAIM`, `JOIN_RAID`, `DROP_CLAIM`, `DROP_STATUS`, `CHAT_MENTION`

## Web UI (`analytics`)

| Key | Type | Default | Description |
| --- | --- | --- | --- |
| `enabled` | bool | `false` | Start the Web UI (implies `miner.enable_analytics`) |
| `host` | str | `127.0.0.1` | Bind address (`0.0.0.0` in Docker) |
| `port` | int | `5000` | HTTP port |
| `refresh_seconds` | int | `5` | Live status update interval (SSE) |
| `days_ago` | int | `7` | Default chart date range |

When disabled, memory and disk use are lower and no `analytics/*.json` files are written.
