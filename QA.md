# Validation

Validated against the surrounding `j3` checkout of Jac on Linux, using port 8127 for the fleet
and port 8128 for a static build of the mobile entry point.

## Automated checks

- `jac check --nowarn`: all six apps passed.
- `JAC_TEST_JOBS=0 jac test`: 3 tests passed, including ownership and the line limit.
- `jac fmt . --check`: all 10 Jac files passed.
- Compiler cross-app import suite: 15 tests passed, including the new regression
  for provider routes and aliased RPC calls.
- Desktop development and native-target suites: 17 tests passed.

## Browser and integration QA

Used `jac browse` sessions at 1280×720 (web) and 390×844 (mobile bundle).

- Registered two accounts, signed out and back in, and rejected a wrong password.
- Posted from the web, mobile, and CLI; all clients displayed the same three posts.
- Liked and unliked a post; reputation changed from 10 to 13 to 10.
- A second user's like raised the author's score; a subsequent CLI post produced
  the expected total of 23 points.
- Verified that another author's posts have no Delete control. Created and
  deleted an owned post from the mobile UI.
- Verified the empty composer disables submission and the mobile page has no
  horizontal overflow. Inspected screenshots at both viewport sizes.
- Reloaded signed-in web, mobile, and packaged desktop pages: all retained their
  sessions and loaded their scores without a render error. A simulated network
  failure showed an error; a successful Refresh cleared it and restored the feed.
- Confirmed feed and scoring return different process IDs in fleet mode.
- Terminated scoring: the feed retained all posts and displayed
  “Score temporarily unavailable,” with no feed error.
- Restarted the entire fleet: persisted post IDs, contents, likes, ownership,
  authentication, and the 23-point score survived. Both service PIDs changed.

Local evidence is under `.jac/qa/`: `web.png`, `mobile.png`,
`native-desktop.png`, `desktop-browser.png`, `scoring-outage.png`, and before/after
restart responses. These generated files
are ignored by Git and contain local demo data.

The first request on the cold compiler cache temporarily blocked a service's
health checks. Registration completed, but joining returned 503; signing in after
the service recovered succeeded. Subsequent operations and the warm restart
completed normally.

The plain colocated `jac run web` does not mount `/api/feed/user/*` in this
checkout and uses a different identity store. This example therefore documents
`jac run --serve --fleet web` as its supported backend command.

## Platform scope

`TINY_JACYAC_URL=http://127.0.0.1:8127 jac build desktop` produced the Linux
native host `.jac/client/desktop/desktop/TinyJacYac`. Its local desktop broker
returned a healthy response; its packaged UI passed sign-in and shared-feed
checks through `jac browse`, displaying the same three posts and 23-point score.
Launched the final native WebKitGTK window and visually verified that it renders
the shared feed. Its generated navigation URL now includes the loopback port.
The local LLVM shim needed rebuilding with `zig build jacllvm` from `jac/`
because its old binary lacked `LLVMPY_InitializeAllAsmParsers`.

`jac build --platform android mobile` passed: a release APK was produced at
`.jac/mobile-rn/android/app/build/outputs/apk/release/app-release.apk`.
Verified that it contains `assets/index.android.bundle`. The release requires
an HTTPS backend under Android's default network policy.

The mobile browser checks run the separately built mobile entry, including its
backend connection screen. No Android device is attached to this environment.
iOS packaging and execution require macOS/Xcode and were not tested here.
