# Paysprint_test
Paysprint_Excode_test
# AGENTS.md — Relimoney Automation

This file documents the project for AI agents (Claude Code, Copilot, etc.) and human contributors. Read it before making any changes.

---

## Project Overview

**Playwright E2E automation** for the Relimoney Admin portal (`https://uat.relimoney.com`).

- **Stack:** Playwright 1.50.1 · TypeScript 5.8.3 · Node.js 20+
- **Pattern:** Page Object Model (POM)
- **Browser:** Chromium (headless by default)
- **Execution:** Sequential (`workers: 1`, `fullyParallel: false`)
- **Target app:** Angular 17+ with Angular Material components

---

## Directory Structure

```
Relimoney_Automation/
├── .github/workflows/test.yml              # CI pipeline (GitHub Actions)
├── .vscode/tasks.json                      # VSCode run tasks
├── changelog/
│   └── CHANGELOG.md                        # Human-authored change log
├── docs/
│   ├── Order Details Page — QA Guide.txt   # Manual QA reference
│   └── order_details.md                    # Order Details test documentation
├── fixtures/
│   └── users.ts                            # Test user credentials
├── pages/
│   ├── login.page.ts                       # LoginPage POM
│   ├── otp.page.ts                         # OtpPage POM
│   ├── new_order.page.ts                   # NewOrderPage POM
│   └── order_details.page.ts              # OrderDetailsPage POM
├── reporters/
│   ├── yaml-spec.reporter.ts               # Custom YAML + run-history reporter
│   ├── html-reporter.ts                    # Custom HTML reporter
│   ├── report-types.ts                     # Shared reporter type definitions
│   └── template/                           # HTML report templates
├── reports/                                # Generated test artifacts (git-ignored)
│   ├── html/                               # Main Playwright HTML report
│   ├── login/                              # Login module reports
│   ├── new-order/                          # New Order module reports
│   ├── order-details/                      # Order Details flow module reports
│   ├── order-details-page/                 # Order Details page module reports
│   ├── spec-definition.yaml                # YAML spec snapshot (auto-generated)
│   └── run-history.md                      # Markdown changelog (auto-appended)
├── tests/
│   ├── login/
│   │   └── login.spec.ts                   # 9 login tests (7 positive + 2 negative)
│   ├── new_order/
│   │   └── new_order.spec.ts               # 13 new order tests (5 positive + 8 negative)
│   └── processing-team-a/
│       ├── order_details_flow.spec.ts      # 21 order details flow tests (8P + 7N + 6V)
│       ├── order_detailes_page.spec.ts     # Full order-details page suite (typo in filename — do not rename)
│       ├── order_details.spec.ts           # Stub test file (all suites skipped via describe.skip)
│       └── full_suite.spec.ts              # Combined full suite runner
├── utils/
│   └── report-utils.ts                     # Shared report helper utilities
├── .env                                    # Local credentials (never commit)
├── package.json
├── playwright.config.ts                    # Default config (all tests); retries: 1 locally, 2 in CI
├── playwright.login.config.ts              # Login module config
├── playwright.new-order.config.ts          # New Order module config
├── playwright.order-details-flow.config.ts # Order Details flow config
├── playwright.order-details-page.config.ts # Order Details page config; storageState: state.json
├── sessionStorage.json                     # Pre-saved sessionStorage for fast auth injection
├── state.json                              # Pre-saved browser storage state (used by order-details-page config)
├── tsconfig.json
├── REPORTS-README.md                       # Guide to reading generated reports
├── Run-All-Tests.bat                       # Windows: login → new order (interactive menu)
├── Run-Login-Tests.bat                     # Windows: login only (interactive menu)
├── Run-New-Order-Tests.bat                 # Windows: new order only (interactive menu)
└── Run-Order-Details-Tests.bat             # Windows: order details only (interactive menu)
```

---

## Running Tests

### From terminal (recommended)

```powershell
# Change into project directory first
cd C:\Organization\RNFI\Relimoney_Automation

# Run login → new order in sequence (one command)
npm run test:all
npm run test:all:headed          # with visible browser

# Run individual modules
npm run test:login               # login tests only (headless)
npm run test:login:headed        # login tests with browser visible
npm run test:login:positive      # positive login cases only
npm run test:login:negative      # negative login cases only

npm run test:new-order               # new order tests only (headless)
npm run test:new-order:headed        # new order tests with browser visible
npm run test:new-order:positive      # grep "P-" prefix
npm run test:new-order:negative      # grep "N-" prefix

npm run test:order-details-flow      # order details flow tests (headless)
npm run test:order-details-flow:headed  # order details flow with browser visible
npm run test:order-details-page      # order details page tests (headless)
npm run test:order-details-page:headed  # order details page with browser visible

# Reports
npm run report                          # open main HTML report
npm run report:login                    # open login HTML report
npm run report:new-order                # open new order HTML report
npm run report:order-details-flow       # open order details flow HTML report
npm run report:order-details-page       # open order details page HTML report
npm run history                         # view last 60 lines of run-history.md
```

### From Windows Explorer (batch files)

Double-click or run in cmd:
- `Run-All-Tests.bat` — runs login first, then new order; menu to choose headless/headed
- `Run-Login-Tests.bat` — login only; menu with all/positive/negative/headed/report options
- `Run-New-Order-Tests.bat` — new order only; same menu options
- `Run-Order-Details-Tests.bat` — order details only; same menu options

### From VSCode

Open Command Palette → `Tasks: Run Task` → pick from the 6 predefined login tasks.

---

## Environment Variables

Stored in `.env` (local only, never commit). Used via `process.env['VAR']` in configs.

| Variable         | Purpose                              | Example value               |
|-----------------|--------------------------------------|-----------------------------|
| `BASE_URL`      | Admin portal base URL                | `https://uat.relimoney.com` |
| `ADMIN_EMAIL`   | Login email                          | `tarun@gmail.com`           |
| `ADMIN_PASSWORD`| Login password                       | `12345678`                  |
| `ADMIN_TYPE`    | User type shown in dropdown          | `Processing Team A user`    |
| `ADMIN_OTP`     | OTP value used in tests              | `1234`                      |

In CI (GitHub Actions), these are stored as repository secrets.

---

## Page Objects

### `pages/login.page.ts` — LoginPage

Handles the `/admin/sign-in` page.

| Method | Description |
|--------|-------------|
| `goto()` | Navigate to login page; waits for form + 1.5s for geolocation API |
| `selectUserType(adminTypeName)` | Click mat-select dropdown, pick option by text |
| `fillEmail(email)` | Clear + type into email input |
| `fillPassword(password)` | Clear + type into password input |
| `clickSignIn()` | Click "Sign in" button |
| `clickSendOtp()` | Click "Send OTP" button (OTP login mode) |
| `loginWithPassword(adminType, email, password)` | Composite: all fields + sign in |

**Key locators:** `userTypeSelect` (combobox "Select User Type"), `emailInput` (placeholder "Enter Email Id"), `passwordInput` (placeholder "Enter Password"), `signInButton`.

---

### `pages/otp.page.ts` — OtpPage

Handles the OTP verification modal (`.register-modal` inside `.cdk-overlay-container`).

| Method | Description |
|--------|-------------|
| `waitForModal(timeout)` | Returns `true` if OTP modal appeared, `false` if login was direct (200 response) |
| `enterOtp(otp)` | Loops through 4 `.input-wrap.otp input` fields, fills each digit |
| `clickVerify()` | Click "Verify & Proceed" button |
| `submitOtp(otp)` | Composite: wait + enter + verify |

**Conditional OTP flow:** If the API returns HTTP 202, the OTP modal appears. If it returns 200, login is direct. `waitForModal()` handles this — always check its return value before calling `enterOtp`.

---

### `pages/new_order.page.ts` — NewOrderPage

Handles the Order History page and New Order dialog.

| Method | Description |
|--------|-------------|
| `gotoOrderHistory()` | Navigate to `/admin/dashboard/order-transaction` |
| `gotoNewOrder()` | Navigate to `/admin/dashboard/new-order` (auto-opens MatDialog) |
| `expandSidebarToNewOrder()` | Smart expand: toggle collapsed sidebar → expand Order accordion |
| `clickNewOrder()` | Click sidebar "New Order" link; falls back to direct navigation if not visible |
| `fillName(name)` | Clear + fill name input in dialog |
| `fillPhone(phone)` | Clear + fill phone input in dialog |
| `selectCity(cityName)` | Type city → wait for `mat-option` → click first match |
| `blurField(locator)` | Press Tab to trigger Angular on-blur validation |
| `clickSubmit()` | Click `.primary-btn` |
| `fillAndSubmit(name, phone, city)` | Composite: all fields + submit |

**Dialog locator:** `.register-modal` (MatDialog `panelClass`). Always scope sub-locators to `this.dialog` to avoid stale DOM matches.

---

### `pages/order_details.page.ts` — OrderDetailsPage

Handles the `/admin/dashboard/order/order-detail` page (product row editing table).

**Locator groups:**

| Group | Locators |
|-------|----------|
| Table rows | `allRows` (`table tbody tr`), `cancelledRows` (`tr.isCancelled`) |
| View-mode chips | `productTypeChip` (`.type-btn`), `productTypeBuy` (`.type-btn.buy`), `productTypeSell` (`.type-btn.sell`) |
| Edit-mode form | `productSelect`, `productTypeSelect`, `currencySelect`, `forexAmtInput`, `inrAmtInput`, `proposeRateInput`, `relationSelect`, `travellerNameInput` |
| Action icons | `editIcon`, `deleteIcon`, `saveIcon`, `closeIcon`, `refundIcon`, `revertIcon` |
| Header buttons | `addFriendButton`, `addProductButton`, `actionMenuButton` |
| Validation | `inlineErrors`, `matErrors`, `proposerateRateError` |
| Dialogs & overlays | `popupCard`, `snackbar`, `swalContainer`, `swalConfirmButton`, `swalCancelButton`, `cancelReasonDialog`, `cancelReasonSubmitButton` |
| Loader | `skeletonRows` |

**Row-scoped helpers (index-based):**

| Helper | Description |
|--------|-------------|
| `row(n)` | Locator for nth `tbody tr` |
| `editIconInRow(n)` | Edit button inside row n |
| `deleteIconInRow(n)` | Delete button inside row n |
| `saveIconInRow(n)` | Save button inside row n |
| `closeIconInRow(n)` | Close/cancel button inside row n |
| `forexAmtInRow(n)` | FX amount input inside row n |
| `inrAmtInRow(n)` | INR amount input inside row n |

**Methods:**

| Method | Description |
|--------|-------------|
| `gotoOrderList()` | Navigate to `/admin/dashboard/order-transaction` |
| `openOrderByIndex(n)` | Click nth row in the order list to open it |
| `selectMatOption(text)` | Pick a mat-option from the CDK overlay by partial text |
| `blurField(locator)` | Press Tab to trigger Angular on-blur validation |
| `waitForSnackbar(timeout)` | Wait for snackbar to become visible |
| `snackbarText()` | Returns snackbar inner text |
| `waitForTableData(timeout)` | Waits until non-skeleton rows are present |

**Form control selectors:** All edit-mode inputs use Angular `formcontrolname` attributes (`product_id`, `product_type_id`, `currency_id`, `forex_amt`, `inr_amt`, `propose_rate`, `relation_id`, `traveller_name`).

**Critical locator notes:**
- `cancelReasonDialog`: `.cdk-overlay-container mat-dialog-container` — MatDialog opens without a custom `panelClass`, so it is NOT `.register-modal`.
- `cancelReasonSubmitButton`: scoped to the dialog, matches `button.btn-success` — the button has `type="button"`, not `type="submit"`.
- `waitForTableData()`: waits on `form .item` — the skeleton `<ng-template>` renders `.item` only when the reactive-form template is active. Do NOT use `tr:not(.skeleton)` (skeleton `<tr>` elements have no `.skeleton` class and will match prematurely).
- `noProduct` state: the "no product" indicator class is on `<td class="noProduct">`, not on the `<tr>`. Assert `.noProduct` visibility, not row count.
- `propose_rate`: renders as `<input>` in edit mode and as `<span>` in view/saved mode. Call `waitFor({ state: 'visible' })` before interacting with it.
- Rate column: `getRateValue()` reads from `<span class="rate">` (cell text), not from an `<input>`. Use this method instead of directly locating an input.

**Auto-created row:** After a new order is submitted and the app navigates to the order-detail page, the first empty row is automatically opened in edit mode. Wait for `productSelect` to be visible as the ready signal — do not proceed before this.

---

## Test Files

### `tests/login/login.spec.ts`

Suite: **"Processing Team A – Login"**

| ID   | Type     | Name / What it covers |
|------|----------|-----------------------|
| P-01 | Positive | Login page renders with all required fields |
| P-02 | Positive | Password login → optional OTP modal → dashboard redirect |
| P-03 | Positive | Empty form submission shows "required" error on email |
| P-04 | Positive | Invalid credentials show error snackbar/toast |
| P-05 | Positive | OTP mode toggle switches UI (password ↔ OTP fields) |
| P-06 | Positive | OTP login flow: email → Send OTP → enter OTP → dashboard |
| P-07 | Positive | Mode switching back from OTP to password mode |
| N-01 | Negative | Verify button disabled until all 4 OTP digits entered |
| N-02 | Negative | OTP modal close button dismisses without submitting |

**Test data:** Credentials from `PROCESSING_TEAM_A` fixture (`fixtures/users.ts`).

**Note:** `afterAll` must NOT call `browser.close()` — doing so closes the entire Playwright browser and breaks test files that run after this one.

---

### `tests/new_order/new_order.spec.ts`

Suite: **"New Order – Order History"**

Each test's `beforeEach` runs a full login (`adminLogin` helper) and navigates to Order History. Tests are independent.

| ID   | Type     | Name / What it covers |
|------|----------|-----------------------|
| P-01 | Positive | "New Order" link visible in sidebar after expansion |
| P-02 | Positive | New Order dialog opens with name, phone, city fields |
| P-03 | Positive | Submit button disabled until all fields valid |
| P-04 | Positive | Valid form submission shows success toast or navigates |
| P-05 | Positive | Successful submission navigates to order-detail page |
| N-01 | Negative | Empty form — submit button disabled |
| N-02 | Negative | Empty name — required error shown, button disabled |
| N-03 | Negative | Empty phone — required error shown, button disabled |
| N-04 | Negative | Phone < 10 digits — pattern error shown |
| N-05 | Negative | Phone starting with 0–5 — pattern error (regex `^[6789][0-9]{9}`) |
| N-06 | Negative | Alphabets in phone stripped by `appInputRestriction="integer"` → required error |
| N-07 | Negative | City typed but not selected from autocomplete → form invalid |
| N-08 | Negative | No errors shown until fields are touched (Angular on-blur) |

**Test data constants (top of file):**
```typescript
const VALID_NAME  = 'John Doe';
const VALID_PHONE = '9876543210';  // starts with 9
const VALID_CITY  = 'Delhi';       // must exist in autocomplete dropdown
```
Update `VALID_CITY` if the dropdown no longer contains "Delhi".

---

### `tests/processing-team-a/order_details.spec.ts`

Contains **stub describe blocks** for future test implementation. All three `describe` blocks use `test.describe.skip()` so they are skipped entirely (including their `beforeEach`). Do not change these to `test.describe()` until the tests are implemented — running a `beforeEach` that does a full login for 48+ empty stubs wastes time and shows misleading skip counts in reports.

---

### `tests/processing-team-a/order_details_flow.spec.ts`

Three suites, all sharing the same `beforeEach`: login as `PROCESSING_TEAM_A`, navigate to Order History, click New Order, fill and submit the new-order dialog, wait for the order-detail URL, and wait for `productSelect` to confirm the auto-created first row is in edit mode.

**Shared helpers (top-level functions):**

| Helper | Description |
|--------|-------------|
| `adminLogin` | Login + optional OTP modal |
| `loginAndCreateOrder` | Full `beforeEach` sequence ending at a ready order-detail page |
| `fillAndSaveCEBuyUSD` | Fill the open row as Currency Exchange / Buy / USD and save it |

**Static waits removed (2026-05-22):** All `waitForTimeout()` calls for rate/INR auto-calculation were replaced with `expect.poll()` polling on actual field values (e.g., rate > 0, INR > 0, INR matches expected). If auto-calculation tests regress, check `expect.poll()` timeouts rather than re-adding static waits.

**Test data constants:**
```typescript
const VALID_NAME  = 'Test Customer';
const VALID_PHONE = '9876543210';
const VALID_CITY  = 'Delhi';
const FX_AMOUNT   = '1000';
```

#### Positive Scenarios

| ID | What it covers |
|----|----------------|
| TC-OD-FLOW-P-01 | Page loads on order-detail URL with empty first row in edit mode |
| TC-OD-FLOW-P-02 | Fill CE Buy USD row: save succeeds, row switches to view mode (edit icon visible) |
| TC-OD-FLOW-P-03 | Fill FC New Card EUR row: save succeeds |
| TC-OD-FLOW-P-04 | Select Self relation: traveller name auto-fills and field is disabled |
| TC-OD-FLOW-P-05 | Rate auto-fills when currency is selected |
| TC-OD-FLOW-P-06 | INR Amount auto-calculates when FX Amount is entered |
| TC-OD-FLOW-P-07 | Edit saved CE row: change FX amount, INR recalculates, re-save succeeds |
| TC-OD-FLOW-P-08 | Add second row after saving first: CE Sell USD row saves; table has ≥ 2 rows |

#### Negative Scenarios

| ID | What it covers |
|----|----------------|
| TC-OD-FLOW-N-01 | Save empty row: Product field shows required `mat-error` |
| TC-OD-FLOW-N-02 | Click "Add Product" while unsaved row is open: snackbar with save-first message |
| TC-OD-FLOW-N-03 | Duplicate CE Buy USD: second save blocked with duplicate-product snackbar |
| TC-OD-FLOW-N-04 | Edit lock: clicking Edit on second row while first is open shows snackbar |
| TC-OD-FLOW-N-05 | FC New Card + second currency: blocked with single-currency snackbar |
| TC-OD-FLOW-N-06 | Delete only row: blocked via SweetAlert2 or snackbar with minimum-order message |
| TC-OD-FLOW-N-07 | Cancel edit (close icon on second row): changes not persisted; row reverts to view mode |

#### Field Validations

`beforeEach` for this suite pre-fills the open row to CE / Buy / USD state before each test.

| ID | What it covers |
|----|----------------|
| TC-OD-FLOW-V-01 | Rate auto-fills when currency is selected |
| TC-OD-FLOW-V-02 | INR Amount updates when FX Amount is entered |
| TC-OD-FLOW-V-03 | FX Amount set to 0: INR Amount resets to `0` or blank |
| TC-OD-FLOW-V-04 | Self relation: traveller name auto-fills and is disabled |
| TC-OD-FLOW-V-05 | Friend relation: traveller name field is enabled and editable |
| TC-OD-FLOW-V-06 | INR Amount input is read-only (cannot be manually edited) |

---

### `tests/processing-team-a/order_detailes_page.spec.ts`

> **Filename typo is intentional** — do not rename this file. The playwright config `testMatch` references it by name.

Full order-details page test suite. Uses `setupOrderDetailPage()` in every `beforeEach`, which calls `ensureAuthenticated()`.

**Authentication pattern (`ensureAuthenticated`):**
1. **Fast path:** If `sessionStorage.json` exists, inject it and navigate to `/admin/order-transaction`. If the URL is correct, session is valid — skip login.
2. **Slow path:** Full login flow (credentials + optional OTP) → writes a fresh `state.json` and `sessionStorage.json`.

This prevents silent auth failures where the app redirects to the login page without throwing an error, causing all assertions to fail against the wrong page.

**Uses `storageState: state.json`** (configured in `playwright.order-details-page.config.ts`) so Playwright injects saved browser storage before each test context starts.

---

## Delete Flow Patterns

These are critical — the delete behavior varies based on row state. Getting it wrong causes tests to look for the wrong dialog.

| Scenario | Trigger | What happens |
|----------|---------|--------------|
| **Single remaining row** | Click delete on the only row | `_toster.error('Minimum One order is required')` fires immediately. No SweetAlert, no dialog. |
| **Fresh/inline-saved row** (not reloaded) | Click delete, 2+ rows present | SweetAlert2 "Are you sure?" → Confirm → row is removed. |
| **Persisted/reloaded row** (after `page.reload()` + `waitForTableData()`) | Click delete, 2+ rows present | CancelReasonDialog opens **directly** as `mat-dialog-container` (no SweetAlert step). |

**Rules for cancel-reason dialog tests:**
- MUST call `page.reload()` then `waitForTableData()` before deleting.
- MUST have 2+ rows — delete the non-first row (index ≥ 1) to bypass the minimum-order guard.
- Do NOT add a SweetAlert confirm step before the dialog.
- Dialog locator: `.cdk-overlay-container mat-dialog-container`
- Submit button locator: `button.btn-success` (it has `type="button"`, not `type="submit"`)

---

## Playwright Configs

| Config file | testMatch | Report output | Notes |
|-------------|-----------|---------------|-------|
| `playwright.config.ts` | all tests | `reports/html/` | `retries: 1` locally, `2` in CI |
| `playwright.login.config.ts` | `tests/login/login.spec.ts` | `reports/login/html/` | |
| `playwright.new-order.config.ts` | `tests/new_order/new_order.spec.ts` | `reports/new-order/html/` | |
| `playwright.order-details-flow.config.ts` | `order_details_flow.spec.ts` | `reports/order-details/html/` | |
| `playwright.order-details-page.config.ts` | `order_detailes_page.spec.ts` | `reports/order-details-page/html/` | `storageState: state.json`, `viewport: null` |

All configs share the same base settings:
- `baseURL`: from `BASE_URL` env var or `https://uat.relimoney.com`
- `headless: true`
- `geolocation: { latitude: 28.6139, longitude: 77.2090 }` (Delhi)
- `trace: 'on-first-retry'`, `screenshot: 'only-on-failure'`, `video: 'on-first-retry'`
- `retries: 2` in CI, `1` locally (changed from 0 on 2026-05-22 to catch flaky-test false failures)

**`playwright.order-details-page.config.ts` differences:** `storageState: 'state.json'` (pre-injects auth), `viewport: null` with `--window-size=1280,800` launch arg, `timeout: 120000`.

---

## Custom Reporter (`reporters/yaml-spec.reporter.ts`)

Runs on every test execution alongside the standard `list` and `html` reporters.

**Outputs two files:**

1. **`reports/<module>/spec-definition.yaml`** — snapshot of this run:
   ```yaml
   suite: "Test Suite Name"
   generated: "2025-01-01T00:00:00.000Z"
   overall_status: passed
   summary:
     total: 9
     passed: 9
     failed: 0
     skipped: 0
   tests:
     - id: 1
       name: "P-01: ..."
       type: positive        # inferred from test name keywords
       status: passed
       duration_ms: 1234
   ```

2. **`reports/run-history.md`** — appended after each run (never overwritten):
   - Timestamp in IST (Asia/Kolkata)
   - ✅ PASSED / ❌ FAILED status badge
   - Summary table
   - List of failed/skipped tests with error messages

**Type inference logic:** A test is `negative` if its name contains any of: `invalid`, `error`, `empty`, `fail`, `wrong`, `disabled`, `close`, `dismiss`, `validation`. Otherwise `positive`.

---

## CI/CD (GitHub Actions)

File: `.github/workflows/test.yml`

**Triggers:** push to `main`/`develop`/`master`, pull request, manual dispatch.

**Two sequential jobs:**

```
login-tests  →  new-order-tests (needs: login-tests)
```

Both jobs:
1. Checkout → Node 20 → `npm ci`
2. `npx playwright install chromium --with-deps`
3. Run tests with `CI=true` (enables 2 retries, forbids `.only`)
4. Upload HTML report as artifact (30-day retention)
5. Upload failure artifacts on test failure (14-day retention)

**Secrets required** (set in repo Settings → Secrets):
`BASE_URL`, `ADMIN_EMAIL`, `ADMIN_PASSWORD`, `ADMIN_TYPE`, `ADMIN_OTP`

---

## Fixtures (`fixtures/users.ts`)

```typescript
export interface TestUser {
  email: string;
  password: string;
  adminType: string;
  otp: string;
}

export const PROCESSING_TEAM_A: TestUser = {
  email: 'tarun@gmail.com',
  password: '12345678',
  adminType: 'Processing Team A user',
  otp: '1234',
};
```

To add a new user role, add a new exported constant here and import it in the relevant spec file.

---

## Angular-Specific Notes

These are common gotchas when writing or debugging tests in this project:

- **Mat-select:** Use `getByRole('combobox')` — the `<mat-select>` renders as an ARIA combobox. Do not target the internal `<div>` directly.
- **Mat-error:** Only renders after a field is touched. Call `blurField(locator)` (Tab keypress) before asserting error visibility.
- **Mat-option (autocomplete):** Typing into city input is not enough — the test must wait for `mat-option` to appear and click it. The form stays invalid if the user only types.
- **Mat-option exact matching:** Use `new RegExp('^' + text + '$')` to match mat-options — partial text can collide (e.g., "Buy" matches "Buy Back"). Prefer exact-text RegExp over `filter({ hasText })` for option lists with similar names.
- **AppInputRestriction directive:** The phone field strips non-integer characters on input. Typing letters will result in an empty field, not letters in the field.
- **CDK overlay:** The OTP modal lives inside `.cdk-overlay-container .register-modal`. MatDialog without a custom `panelClass` (e.g., the cancel-reason dialog) renders as `.cdk-overlay-container mat-dialog-container` — do not use `.register-modal` for it.
- **On-blur validation:** Angular Reactive Forms in this app use `updateOn: 'blur'`. Errors don't appear until a field loses focus. Call `blurField(locator)` (Tab keypress) before asserting errors.
- **Geolocation:** The login page requests browser geolocation. `LoginPage.goto()` waits for `signInButton` to be visible rather than using a static `waitForTimeout()` — do not reintroduce static waits.
- **Propose rate field:** Renders as `<input formcontrolname="propose_rate">` in edit mode and as `<span>` in view/saved mode. Always `waitFor({ state: 'visible' })` before interacting.
- **Rate value (saved row):** Read via `getRateValue()` which reads `<span class="rate">` text — there is no input in saved-row view mode.
- **Traveller name auto-fill:** After selecting "Self" relation, the name fills asynchronously. Use `expect.poll()` to wait rather than asserting immediately.
- **Skeleton loader:** The skeleton `<tr>` elements have no `.skeleton` class. Use `form .item` in `waitForTableData()` (only present in the reactive-form template, not the skeleton `<ng-template>`).
- **No-product state:** The empty-table indicator class is `noProduct` on a `<td>`, not the `<tr>`. Assert `td.noProduct` visibility, not row count.

---

## Adding New Tests

### New test in an existing module

1. Add the test to the relevant `*.spec.ts` file.
2. Follow the naming convention: `P-XX: description` for positive, `N-XX: description` for negative.
3. Reuse existing page object methods. Only add new POM methods if a selector or interaction is genuinely new.

### New module

1. Create `tests/processing-team-a/<module>.spec.ts`.
2. Create page objects in `pages/<module>.page.ts`.
3. Create `playwright.<module>.config.ts` (copy an existing one, change `testMatch` and report paths).
4. Add npm scripts to `package.json` (`test:<module>`, `test:<module>:headed`, `report:<module>`).
5. Create `Run-<Module>-Tests.bat` for Windows users.
6. Add a new job to `.github/workflows/test.yml`.
7. Update `test:all` script to include the new module.

---

## Common Issues

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `VALID_CITY` not found in autocomplete | City not in backend data | Change `VALID_CITY` constant to a valid city |
| OTP modal never appears | API returned 200 (direct login) | Expected — `waitForModal()` returns false, test skips OTP |
| Submit button stays disabled | City typed but not selected from dropdown | Use `selectCity()` which clicks the mat-option |
| Mat-error not visible | Field not yet touched | Call `blurField()` before asserting error |
| Tests pass locally but fail in CI | Timing issue or missing env var | Check CI secrets; add `waitFor` if selector race |
| `reports/run-history.md` not created | First run or reports folder missing | Run any test once; folder is created automatically |
| order-details-page tests all fail with wrong-page assertions | Session expired, silent redirect to login | `ensureAuthenticated()` handles this — if it recurs, delete `state.json` and re-run to force a fresh login |
| Cancel-reason dialog not found | Test deleting without a prior `page.reload()` | Add `page.reload()` + `waitForTableData()` before the delete — persisted rows open the dialog; fresh rows open SweetAlert |
| SweetAlert step times out on persisted rows | Wrong delete-flow branch | See Delete Flow Patterns section — persisted rows skip SweetAlert entirely |
| `waitForTableData` resolves too early (skeleton still visible) | Old `tr:not(.skeleton)` selector | Use `form .item` — the current implementation is correct; revert if accidentally changed |
| Rate/INR field value stays 0 after currency selection | `waitForTimeout` removed, `expect.poll` timing | Increase `expect.poll` interval or check if the API call for rates is slower than expected |
| `mat-option` click hits wrong option | Partial text match collision | Use `new RegExp('^' + text + '$')` for exact matching |
| Large skip count in reporter (e.g., 48+ skipped) | Stub `describe` blocks running `beforeEach` | Ensure stub describe blocks use `test.describe.skip()`, not `test.describe()` |

---

## TypeScript Path Aliases

```json
"@fixtures/*"  →  fixtures/*
"@pages/*"     →  pages/*
```

Use these in new files:
```typescript
import { PROCESSING_TEAM_A } from '@fixtures/users';
import { LoginPage } from '@pages/login.page';
```

---

## Minor Purpose Matrix (SUITE 21, added 2026-06-09)

Business rule: a **minor traveller (age < 18)** may NOT use the **Business** or **Employment**
purposes on **Buy / New Card / Reload Card** (Sell/Unload show no DOB; Remittance has no DOB).

- Tests: `tests/traveller_details/traveller_details.spec.ts` → `TC-TD-MINOR-MTX-01..05`.
- Page object: `TravellerDetailsPage.purposeOptionLabels(scope?)` opens the Purpose `mat-select`
  and returns trimmed option labels.
- App enforcement: `admin_frontend/.../traveller-detail.component.ts → applyMinorPurposeRestriction()`
  (called from `updateFieldVisibility`) filters Business/Employment from each product's `purposeList`
  by name when age < 18.
- **Deployed-UAT status:** the deployed build does NOT yet enforce this — the 3 minor tests
  (`-02/-03/-05`) fail on UAT (documented defect); the 2 adult tests (`-01/-04`) pass. They go green
  after the frontend fix is deployed. Report: `reports/traveller-details/Minor-Purpose-Matrix-Report.md`.

## All-Purposes fill→save→continue (SUITE 22, added 2026-06-09)

`TC-TD-PURP-01..06` select each non-Personal purpose on an adult Indian-Resident CE Buy, fill its
dynamic fields, save, and Skip & Continue. Business → Company Type + Company PAN; Education →
Education Loan Yes(+amount)/No; Emigration/Medical/Employment → none. Helpers:
`TravellerDetailsPage.selectPurposeByName(scope,label)` (forces a purpose over a default) +
`fillPurposeExtraFields(scope,{companyType,companyPan,educationLoan,loanAmt})`.

- **Business save-blocker (DEF-TD-BUSINESS-DECL):** Business on CE Buy hides the LRS declaration but
  leaves `declared_fx_amt/declared_inr_amt` required → unsaveable. App fix staged in
  `traveller-detail.component.ts updateProductValidators()` (re-sync `applyDeclarationValidators`
  per traveller). `TC-TD-PURP-01` is red on deployed UAT until the fix ships; the other 5 pass.
- No PAN-category-vs-company-type validation exists (only required + PAN pattern).

**Full-suite run 2026-06-09:** 93 tests → 87 pass / 4 fail / 2 skip (56 min). The 4 fails are all
staged-fix defects: `TC-TD-MINOR-MTX-02/03/05` + `TC-TD-PURP-01`. Report:
`reports/traveller-details/All-Purposes-And-Full-Suite-Report.md`.

*Last updated: 2026-06-09*
