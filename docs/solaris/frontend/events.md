# Event Contracts

Return semantics are Component-specific. Do not assume every event can cancel work.

| Contract | Behavior |
|---|---|
| Standard SolarComponent | `on(type, callback, name)`; named handlers can be replaced or removed |
| Form lifecycle | Async notification; throwing aborts but enters error handling |
| StageButtons `click` | A result other than `true`, `null`, or `undefined` vetoes transition |
| Table custom Delete | Must resolve `true` after custom deletion succeeds |
| SolarListDetail | `on(type, callback)` only; no listener name or public `off()` |
| Library-backed input | May provide library detail after the Solaris value |

Use events for orchestration, not to duplicate standard requests. Prefer validation APIs over throwing
from lifecycle handlers. Explicitly remove native listeners owned by dynamic UI.
