# UCAN Audience Domain Change (llm -> router)

## Summary
UCAN auth failed with `UCAN audience mismatch` after the public domain moved from `llm.yeying.pub` to `router.yeying.pub`. The fix was to align Router's expected audience to the new domain via systemd environment variables and restart the service.

## Root Cause
Router expected `did:web:llm.yeying.pub` while UCAN tokens were issued with `did:web:router.yeying.pub`, causing a strict string mismatch in UCAN verification.

## Changes Applied
- systemd service updated:
  - `Environment=UCAN_AUD=did:web:router.yeying.pub`
  - `Environment=AUTO_REGISTER_ENABLED=true`
- Service reloaded and restarted:
  - `systemctl daemon-reload`
  - `systemctl restart router`

## Verification Evidence
- Mismatch (before):
  - `expected=did:web:llm.yeying.pub actual=did:web:router.yeying.pub`
  - Example request id: `2026020119430022524911775057287`
- Success (after):
  - `token auth via ucan success user=18 addr=0xcf13850cc9ac9f07e8767d3e6e50795357f59243 use_token=21`
  - Logged at `2026-02-02 05:08:15/50/53`

## Notes / Lessons
- UCAN audience is a strict string match; DNS/Nginx/HTTPS does not affect it.
- systemd does not load `.env` automatically; set UCAN/Router-related env vars in the service or an EnvironmentFile.
- Ensure the UCAN issuer (chat side) uses the same `did:web:router.yeying.pub` audience.
