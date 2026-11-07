# Roomba V400 → Home Assistant (cloud control)

A short, field-tested guide to getting a **Roomba V400** working in Home
Assistant when local control is closed on the robot's firmware, using the
cloud path. Confirmed on real hardware (see [Verified](#verified)).

> This is a **guide**, not an integration. All the actual code is
> [`a-mavrides/roomba_v4`](https://github.com/a-mavrides/roomba_v4) (MIT).
> Full credit to its author — this repo only documents how to use it on a V400
> and the gotchas I hit. If it helped you, star *their* repo.

---

## Who this is for

You have a recent Roomba (here a **V400**) and:

- the official **iRobot app** works, but
- Home Assistant's built-in / local-MQTT integrations (e.g. `roombapy`) **fail
  to connect** — timeouts, "password" that never comes, nothing on port 8883.

That's expected: iRobot has **closed local control on recent firmware**. The
robot now only talks to iRobot's cloud. So the integration has to go through
the cloud too — which is exactly what `roomba_v4` does.

## What you get

Full control, all from Home Assistant:

- A native `vacuum.*` entity: **start / pause / stop / return to base**, battery,
  status — the standard HA *Vacuum* card works on it directly.
- Sensors: battery %, a human-readable **status message**, **mission phase**
  (`charge`, `run`, `stuck`…).
- A **map** camera entity.
- Buttons (clean all, dock, pause/resume/stop, spot clean) and selects
  (cleaning mode, suction level, per-room).

~32 entities in total on the V400, `commands_supported_yet: true`.

## Prerequisites

- Home Assistant with **HACS** installed.
- Your **iRobot account** (the same login as the iRobot app). Set your robot up
  in the official app **first** — give it a name, let it finish onboarding.
- Your HA instance has outbound Internet (this is a **cloud** integration).

## Install

1. In **HACS → Integrations → ⋮ → Custom repositories**, add
   `https://github.com/a-mavrides/roomba_v4` as an *Integration*.
2. Install it from HACS, then **restart Home Assistant**.
3. **Settings → Devices & Services → Add Integration → Roomba (v4)**.
4. Enter your **iRobot account** credentials and your **country** (FR here).
   The flow logs in to the iRobot cloud and discovers the robot.
5. Done — the `vacuum.*` entity and its sensors/buttons appear.

(Exact menu labels follow the upstream repo; check its README if a step moved.)

## Gotchas (the useful part)

These are the things that aren't obvious until you hit them:

- **It's cloud polling, not local.** State can lag a little and depends on
  iRobot's cloud + your Internet. Don't treat a shown state as live truth — the
  entity exposes freshness attributes (`telemetry_stale`, `status_stale`,
  `telemetry_age_seconds`); use them in your dashboard to grey out / timestamp
  the tile when the cloud goes quiet, instead of showing a frozen state as real.
- **`unavailable` is usually the auth, not the robot.** If the entity drops to
  `unavailable`, it's often the iRobot cloud auth (e.g. after an app update),
  not a broken vacuum. Re-auth before assuming hardware trouble.
- **Only one `room1` until you remap.** Per-room cleaning shows a single room
  until the robot builds a fresh map. Run a full clean / remap from the app, and
  the room list fills in.
- **Local integrations will keep failing** on this firmware — that's not a
  misconfiguration, local control is simply off. The cloud path is the answer.

## Verified

| | |
|---|---|
| Robot | `Roomba_V400` |
| SKU | **G185240** |
| Firmware | **p25-405+9.3.7+I4.6.150** |
| Country | FR |
| Result | full control, start → pause → dock obeyed within ~2 min |

Reported upstream here:
[a-mavrides/roomba_v4#7](https://github.com/a-mavrides/roomba_v4/issues/7).

## Not covered

- Genuine local control (closed by firmware — nothing to do about it here).
- Other robot models: the integration may well work on them, but this guide
  only confirms the V400. If yours works, open an issue upstream so the list
  grows.

---

## Credits & license

- Integration: **[`a-mavrides/roomba_v4`](https://github.com/a-mavrides/roomba_v4)** — MIT. All the credit for the working code goes there.
- This guide: MIT (see [LICENSE](LICENSE)). No warranty; use at your own risk.
