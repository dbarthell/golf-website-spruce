---
id: GOL-1et74c
type: task
title: Match MVCC color (navy) instead of green
status: done
---
# Match MVCC color (navy) instead of green

## Work Done

Updated the five CSS custom property color values in `css/styles.css` from green to navy to match MVCC's color scheme. The variable names were kept the same so all 35+ references throughout the stylesheet automatically inherit the new colors without any further changes.

| Variable | Before (green) | After (navy) |
|---|---|---|
| `--green-dark` | `#1a3a0a` | `#0d2137` |
| `--green-mid` | `#2d5016` | `#1a3a6e` |
| `--green-accent` | `#4a7c34` | `#2563a8` |
| `--green-light` | `#6ba352` | `#4a7fc1` |
| `--green-pale` | `#e8f0e4` | `#e8eef7` |

The comment above the variables was also updated from `/* Golf-inspired palette */` to `/* MVCC navy palette */`.
