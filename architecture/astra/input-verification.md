# Independent input verification

Verified 2026-10-07 before architecture work. All seven required findings, exact intent and architecture brief are present, readable, and match their entries in the project-root `TRANSFER-SHA256SUMS`. Only these entries were used for content verification. Findings were read in full. No application or system experiments were run.

| File | Bytes | SHA-256 | Result |
|---|---:|---|---|
| `intent.md` | 705 | `f4fbe9eda9500a68bb629375651b40e02d737dabe01bf57da643d6f697fc0de0` | PASS |
| `architecture-brief.md` | 3018 | `f6f5f9d0a96919384e3d02efa5a7436d029a0df99f73ea62084709242a4f69e9` | PASS |
| `research/findings/native.md` | 24643 | `bed990e201e9d3df5332022603be65150812f9996a9ddec69aab1493390e50e2` | PASS |
| `research/findings/web.md` | 27824 | `5e45b9afa3c55c4b679d1430bfe069643086f5b3e39591e5ae568714ff0621d1` | PASS |
| `research/findings/precedents.md` | 25359 | `78f6296b26d0f42b72572449f26b1e374c65f3ee5b7dbb71bff92aef6bf149bb` | PASS |
| `research/findings/modding.md` | 13490 | `8954420c72842a24f7e2e7346117c3fc4687b3cdb67ce826ebdfbeae91421fa6` | PASS |
| `research/findings/semantics.md` | 9339 | `1c7007358001956536c4447d9240ef903bc76bf8366b38c8b0947c9c7ffcb25e` | PASS |
| `research/findings/report.md` | 1154 | `0f313c7536ac19c0db5bccb09f6e9df62a03e2c4a4fc0c0fff36b8498800ac3d` | PASS |
| `research/findings/critique.md` | 5799 | `c55edb66ed48a0739e151106fa77783e9a283808426903b1ec651494ba214bc1` | PASS |

Evidence boundary: recommendations/, other architect directories, source-project material outside eversion, and prior report/history were not consulted. Applicable ancestor/project AGENTS.md and project .agents instructions were checked; none were found. Principal and research subagents are restricted to architecture/astra for documentation writes. No Git writes or publishing.
