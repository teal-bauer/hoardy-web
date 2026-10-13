# Archival regression tests

From the repository root, with Node.js 22 or newer:

```sh
node --test extension/tests/archival.test.cjs
```

The tests execute the actual extension persistence, local-storage, base, and issue-accumulator sources in a VM. Browser storage and HTTP requests are mocked. They cover bounded parallel submission, payload budgets, oversized records, serial opt-out, mixed archival methods, successful acknowledgements before deletion, failure recovery, pausing, endpoint changes, scheduler priority, and local-storage metadata bookkeeping.

Build a Firefox MV2 package with the existing build toolchain:

```sh
git submodule update --init vendor/pako
cd extension
./build.sh firefox-mv2
```

For manual integration testing, use a disposable Firefox profile and an isolated archive directory. Enable HTTP-only archival, set `HTTP upload batch size` to 4, queue test captures, and compare upload throughput with batch size 1. Verify local counters and archive contents after completion. Test HTTP errors, server outages, pause/resume, and a browser restart with a partially processed queue. Repeat with `Prefer IndexedDB API` both enabled and disabled.

The default batch size remains 1. Re-archival and mixed-method archival retain serial behavior. A pause or endpoint change during a prepared batch may cause acknowledged uploads to be sent again; disk copies remain available until successful finalization. The payload budget describes uncompressed captures and excludes browser/network bookkeeping overhead.

`storage.local` still uses separate metadata-write and record-removal calls, matching the existing persistence implementation. These tests do not establish crash atomicity for that API. Native browser lifecycle and actual IndexedDB integration remain manual checks before release.
