# ⚠️ Fork Patches — Sync Warning

This fork has custom patches that **need re-application after syncing with upstream**.

## Patched files

### 1. `headroom/telemetry/beacon.py`
- **What changed:** `_SUPABASE_URL` redirected from `https://dtlllcsudcoasebbamcq.supabase.co` → `http://127.0.0.1`
- **Why:** Prevent any possibility of anonymous telemetry data leaving the network
- **Note:** Telemetry is already opt-in (OFF by default), this is a defense-in-depth measure.

### 2. `headroom/telemetry/reporter.py`
- **What changed:** `cloud_url` default redirected from `https://app.headroomlabs.ai` → `http://127.0.0.1`
- **Why:** Same reason — defense-in-depth.
- **Note:** Reporter only activates when `HEADROOM_LICENSE_KEY` is explicitly set (enterprise/managed). Normal users never hit it.

## After upstream sync

Run these commands to re-patch:

```bash
# Patch beacon.py
sed -i 's|_SUPABASE_URL = "https://dtlllcsudcoasebbamcq.supabase.co"|_SUPABASE_URL = "http://127.0.0.1"|' headroom/telemetry/beacon.py

# Patch reporter.py
sed -i 's|cloud_url: str = "https://app.headroomlabs.ai"|cloud_url: str = "http://127.0.0.1"|' headroom/telemetry/reporter.py
```

## Other telemetry notes

| Component | Default state | Outbound target | Notes |
|---|---|---|---|
| `beacon.py` (anonymous telemetry) | **OFF** (opt-in) | Supabase → now 127.0.0.1 | Only sends aggregate counts, no PII |
| `reporter.py` (usage reporter) | **OFF** (license required) | app.headroomlabs.ai → now 127.0.0.1 | For enterprise/managed users only |
| `toin.py` (traffic learner) | Local only | None | No outbound |
| `collector.py` | Local only | None | No outbound |
| `context.py` | Local only | None | No outbound |
