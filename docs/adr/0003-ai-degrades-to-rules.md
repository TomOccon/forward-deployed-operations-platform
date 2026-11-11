# ADR 0003: Every AI feature has a rule-based fallback

Status: accepted

If an AI call fails (rate limit, model unavailable, malformed output), the app shows the rule-based result instead of an error. The same principle shaped the sibling [ai-support-intelligence](https://github.com/TomOccon/ai-support-intelligence) pipeline, which validates strict JSON and records failures instead of raising.

Tradeoff: users may not notice AI degradation. That is intentional; a visible error is worse than a correct but simpler answer.
