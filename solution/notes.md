# Diagnosis scratchpad

Run the practice simulator, read YOUR telemetry, and note what you find.
Fault classes to hunt: error_spike · latency_spike · cost_blowup · quality_drift ·
infinite_loop · tool_failure · pii_leak · prompt_injection · fabrication · arithmetic_error · tool_overuse.

| symptom (from telemetry) | which requests | suspected cause | config fix? | wrapper fix? |
|---|---|---|---|---|
| **error_spike**: Tool calls fail intermittently (about 18% error rate) | Random queries throughout runs | `tool_error_rate` set to 0.18 in config | Set `tool_error_rate` to 0.0 in config | Implemented exponential backoff retries |
| **latency_spike**: P95 latency is extremely high (> 20s) | Queries requiring multiple tools or retries | `model_price_tier` set to premium, `cache` disabled | Set price tier to standard, enable `cache` | Added thread-safe caching |
| **cost_blowup**: Token usage and billing costs are high | Requests with long session history | Large `context_size` and `verbose_system` enabled | Reduce `context_size`, set `verbose_system` to false | None |
| **quality_drift**: Correctness drops in later turns of a session | Sessions with turn index >= 3 | `session_drift_rate` set to 0.06 in config | Set drift rate to 0.0, use `context_reset_every` | None |
| **infinite_loop**: Tool calls repeat until max steps is reached | Loop-prone requests with format bugs | `loop_guard` disabled, high `max_steps` | Enable `loop_guard`, reduce `max_steps` to 6 | None (prompt formatting controls help) |
| **tool_failure**: Accented cities fail; MacBook is incorrect | Queries for MacBook or accented cities (Hà Nội, v.v.) | `catalog_override` forces MacBook out, `normalize_unicode` off | Clear `catalog_override`, enable `normalize_unicode` | None |
| **pii_leak**: Email/phone numbers duplicated in response | Queries containing customer email/phone | `redact_pii` disabled in config, weak prompt | Set `redact_pii` to true, prompt restraint | Implemented regex PII redaction |
| **prompt_injection**: Fake prices/codes applied from notes | Queries with "GHI CHÚ" in private phase | Prompt lacks note restrictions | No | Strip/sanitize input, add prompt warning |
| **fabrication**: Agent invents total price for unavailable items | Queries for out-of-stock items | Grounding rules missing from prompt | No | None (handled via prompt constraint) |
| **arithmetic_error**: Totals or discounts computed incorrectly | High temperature (1.6) runs | High `temperature` in config, lack of step formulas | Reduce `temperature` to 0.2 | None (handled via prompt math formula) |
| **tool_overuse**: Calling tools unnecessarily | Queries with out-of-stock items | Missing cap on tools in prompt/config | Set `tool_budget` to 4 | None (handled via prompt halting rule) |

