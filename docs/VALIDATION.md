# Verification record

Verified locally on 2026-10-19 (Asia/Shanghai). All records are synthetic. The event timestamps shown in screenshots are UTC, so the calendar date can differ from this local date.

## Build and API integration suite

Environment: macOS ARM64, .NET SDK 10.0.400, runtime 10.0.11, Microsoft.Data.Sqlite 10.0.13, Python 3 standard library.

```text
dotnet build src/TraceFlow.Web --no-restore
Build succeeded.
    0 Warning(s)
    0 Error(s)

python3 -m unittest discover -s tests -v
Ran 22 tests in 1.246s
OK
```

The suite starts a real ASP.NET Core process with an empty temporary SQLite database. Assertions cover:

1. Service health and fixed route metadata.
2. Invalid quantities and missing serials.
3. Global serial-number uniqueness across work orders.
4. Planned-quantity cap.
5. Skip-station rejection with unchanged state and events.
6. Full route completion and rejection after completion.
7. Failure → hold → release → same-station retest → packing.
8. Required failure and release reasons.
9. Rejection of release on a unit without a hold.
10. Registration retry and changed-payload conflict.
11. Station-report retry without duplicate events.
12. Invalid station, result, and empty operator.
13. Invalid identifier / SQL-like input rejection.
14. Eight concurrent registration requests against capacity three: three accepted, five rejected.
15. Six concurrent retries: one registered unit and one event.
16. CSV round-trip quoting and formula-prefix protection.
17. Dashboard counts agree with work-order data.
18. Database triggers reject event UPDATE and DELETE.
19. Restart preserves both unit history and idempotency response.
20. Injected event INSERT failure rolls back the unit change; retry later succeeds using the same request ID.
21. Identifier normalization and missing request ID validation.
22. Structured JSON errors for malformed query types and unknown endpoints.

Test 20 intentionally causes a server error using a temporary test-only SQLite trigger. The assertion verifies that database internals are absent from the HTTP response. No failure-injection endpoint exists in the application.

## Browser checks

Checked through the actual local browser UI at the default desktop viewport:

- Initial dashboard: one seeded order, three units, one completed, one held.
- `SN-DEMO-0001`: attempted PACK; UI displayed `The next required station is TEST.`
- `SN-DEMO-0002`: released with `Demo fixture checked; retest required.`; next station remained TEST.
- Reported TEST PASS, then PACK PASS; trace showed Completed and six chronological events, including the original failure and release.
- Created `WO-UI-DEMO-1002` / `DEMO-SENSOR`, planned quantity two, and registered `SN-UI-DEMO-0004`; UI showed Ready at ASSEMBLY.
- Event filter selected `WO-DEMO-1001`; CSV download completed through the visible Export CSV link.
- JavaScript syntax check passed with `node --check`.

Screenshots in `docs/images` were captured from the running application, at different points in that sequence. Their counts therefore differ. They are not AI-generated UI mockups.

## Startup correction

The sandbox's configuration file watcher stalled ASP.NET Core startup. This environment-configured demo now defaults `DOTNET_HOSTBUILDER__RELOADCONFIGONCHANGE` to `false` before creating the builder. Configuration changes take effect after restart. An explicit environment value can override that default. The final binary started successfully without extra runtime workaround flags.

## Limits

These are local integration and desktop browser checks. Windows, mobile devices, external equipment, multi-server deployments, sustained load, authentication, authorization, backup recovery, and production security have not been validated. The GitHub Actions file is included for reproducibility; hosted CI results must be checked separately. This prototype is not suitable for direct use on a factory network.
