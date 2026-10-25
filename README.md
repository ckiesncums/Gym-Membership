# FORM — Gym Membership Management System

An admin-only PHP application for member registration, membership subscriptions, payments, printable QR cards, webcam attendance, expiration monitoring, and reports. All ten project phases are implemented together at the user's request. Members have no login accounts.

## Open the application

1. Open **C:\xampp\xampp-control.exe** and start **Apache** and **MySQL**.
2. Open **http://localhost/gym-membership/** in Brave.
3. Sign in with the administrator account provided at handoff. The generated temporary password must be changed at the first sign-in. Passwords must be 12–72 bytes long.

The htdocs directory `C:\xampp\htdocs\gym-membership` is a Windows directory junction to `C:\Users\Ken\Documents\PHP`. Both paths refer to the same files. The application uses the **C-drive** database, not the old E-drive XAMPP.

The seven seeded demonstration members are still present, alongside all subsequently registered members. The `demo_fixture_admin` account is disabled. There is no shared default administrator password and no public sign-up page.

## Features

- Password-hashed admin login/logout, session renewal, idle/absolute expiry, password changes, and persistent failed-login throttling.
- Dashboard counts for registered/new/active/expiring/expired members, daily check-ins, recent registrations, and selected-month payment collections.
- Member registration, optional first subscription/payment, editable personal details, search/status filters, pagination, profiles, archive/restore, and histories.
- Configurable plans, exact fees, calendar-month expiration dates, renewals, immutable price snapshots, partial/full payments, payment corrections by voiding, and outstanding balances.
- Cryptographically random member QR tokens, PHP-generated SVG QR images, printable cards, downloads, and replacement of lost cards.
- Webcam QR scanning and local QR-image scanning, server validation, clear rejection messages, daily duplicate prevention, and immediate attendance totals.
- Attendance search/date filters; membership, attendance, payment, and expiring-member reports; printable pages and CSV exports.
- Club settings and administrative password changes. The established charcoal/ivory/orange design is preserved and uses Bootstrap 5 with custom CSS.

## Existing installation

This machine already has the schema, defaults, demo records, dependencies, dedicated application database account, and additive application migration installed. **Do not re-import `schema.sql` into the populated database.** `config/local.php` contains a generated database password and is ignored by version control. Do not publish it.

The verified runtime is PHP 8.5.12 and MariaDB 10.11.16 from C-drive XAMPP. PHP 8.2+ with PDO MySQL and mbstring is supported. The database contains nine InnoDB tables and two read views; `operation_receipts` adds transactional idempotency to the eight Phase 2 tables.

## Fresh installation

Install XAMPP, Composer, and the source under an htdocs folder named `gym-membership`. Run `composer install` using PHP 8.2+; the exact dependency versions are pinned in `composer.lock`. On this machine the verified Composer PHAR is also available locally in `.tools/composer.phar`:

```powershell
Set-Location -LiteralPath 'C:\xampp\htdocs\gym-membership'
& 'C:\xampp\php\php.exe' -d extension=zip '.\.tools\composer.phar' install --no-interaction

& 'C:\xampp\mysql\bin\mysql.exe' --no-defaults --host=127.0.0.1 --user=root --abort-source-on-error --execute='source C:/xampp/htdocs/gym-membership/database/create-database.sql'
& 'C:\xampp\mysql\bin\mysql.exe' --no-defaults --host=127.0.0.1 --user=root --database=gym_membership --abort-source-on-error --execute='source C:/xampp/htdocs/gym-membership/database/schema.sql'
& 'C:\xampp\mysql\bin\mysql.exe' --no-defaults --host=127.0.0.1 --user=root --database=gym_membership --abort-source-on-error --execute='source C:/xampp/htdocs/gym-membership/database/seeds/plans.sql'

& 'C:\xampp\php\php.exe' '.\scripts\configure-local.php'
'{"username":"owner","name":"Gym owner"}' | & 'C:\xampp\php\php.exe' '.\scripts\create-admin.php'
```

The admin provisioner reads JSON from standard input. Without a password field it generates a temporary password, displays it once in the command output, and requires a change at sign-in. A `password` field can instead be supplied through stdin; do not place passwords in shell arguments or source files. `configure-local.php` is for a fresh database account; it refuses to overwrite an existing account or private configuration.

The optional `database/seeds/demo.sql` adds only educational fixtures and a disabled admin. Its predictable QR values must not be reused for real members. Run it only in a development database after the default plans.

For an **existing Phase 2 database**, apply the additive migration instead of importing the full schema:

```powershell
& 'C:\xampp\php\php.exe' '.\scripts\migrate.php'
```

Migration defaults to local root with no password. Set `GYM_MIGRATION_USER` and `GYM_MIGRATION_PASSWORD` in the temporary process environment if the administrative account differs. The web application always uses the limited `gym_app` account.

## Demonstrate the complete workflow

1. Sign in and change the temporary password.
2. Choose **Register member**, enter a name/contact, and select the three-month plan at PHP 800/month.
3. Verify the preview is **PHP 2,400**. Enter **2400.00** as the initial payment and save.
4. Open the member profile and print/download the QR card. Subscription, payment, and QR are saved together; repeating the original submission does not create a second purchase.
5. Open **QR check-in**, select **Start camera**, and allow camera access in Brave. Hold a printed QR card or a QR displayed on another device in front of the webcam. A member's name, member code, and check-in time appear after the server commits attendance.
6. Select **Scan next member** and scan the same card again. The result says it was already checked in and retains the first timestamp.
7. View **Attendance** and **Overview**. The daily total updates across open tabs and on navigation. Attendance records are retained permanently.
8. Create a partially paid membership and verify its QR is rejected until the balance is paid. The seeded expired/scheduled/archived members support additional demonstrations.
9. Renew a member. A new non-overlapping term is added, previous prices/payments remain, and the same QR card continues working.
10. Open **Reports**, choose a report, apply relevant dates, and print or export CSV.

For a camera-free check, download a QR SVG and use **Use a QR image instead** on the scanner page. This is still authenticated and uses the same backend eligibility/uniqueness rules. Camera hardware and permission dialogs require a manual browser check on the demonstration machine.

QR downloads have explicit square dimensions and an opaque white border. Image uploads, including older dimensionless SVG downloads, are converted locally to a complete PNG before decoding. The preview fits the full card while a separate hidden canvas retains decoding resolution. If an old scanner tab is still open after an update, use Ctrl+F5 before retrying the upload.

## Business rules

- Amounts use DECIMAL in SQL and integer centavos in PHP. JavaScript only previews prices; the server recalculates from the selected plan.
- A term covers `starts_on <= gym_local_today < ends_on_exclusive`. The displayed expiration is the previous day. Calendar-month addition clamps to the target month's last day: October 25 + three months ends exclusively January 25; the last valid date is January 24. January 31 + one month in 2027 ends exclusively February 28, so its last valid date is February 27.
- Early renewals start at the latest existing term's exclusive boundary. After a lapse they default to today. A later manually chosen start is allowed; overlapping terms are rejected.
- Entry requires a non-archived member and one current, non-voided, fully paid subscription. Expiring-soon members may still enter. Future, expired, unpaid, archived, malformed, replaced, and unknown QR values are rejected.
- One check-in per member per gym-local day. Eligibility is checked again even when an earlier attendance row exists. A later archive/payment void can therefore reject another scan while preserving the first visit.
- New registrations are a separate rolling-calendar-date measure. Active includes expiring-soon; dashboard measures overlap.
- Registration, purchase, and payment submission keys are bound to their action, admin, normalized payload, and member. Receipts commit in the same transaction. A different payload using the same key is rejected.
- Payment amounts may not exceed the outstanding balance. Incorrect payments are voided with a reason. Void their valid payments before voiding a subscription. Archiving never deletes history.
- The gym timezone initially remains `Asia/Shanghai` (UTC+08), and currency is PHP. Historical business dates must not be silently reinterpreted by changing the timezone.

## Security and files

The root `.htaccess` routes requests to `public/`, restricts this local development project to local requests, and blocks private directories and credentials. Admin authentication also applies to all application pages, QR endpoints, reports, CSV exports, and scanner APIs. Every state-changing form/API requires CSRF validation. Sessions use HttpOnly/SameSite cookies, strict session IDs, a 30-minute idle limit and eight-hour absolute limit; HTTPS enables Secure cookies. Password changes revoke other sessions.

`app/Services/` owns transactions and business rules; `app/Repositories/` owns prepared SQL; `app/Views/` contains escaped templates; `app/Application.php` dispatches allowlisted routes. `bootstrap/app.php` configures UTF-8, UTC/strict SQL, sessions, headers, and errors. Private error logs live in `storage/logs/`; QR tokens/passwords are not written to request logs.

Member locks are acquired before subscription/payment/attendance decisions. Locking reads, unique daily attendance, unique submission keys, receipt hashes, and bounded deadlock retries protect concurrent requests. The webcam decodes locally and sends only its opaque token, not video frames or member personal data.

## Tests

From the project directory:

```powershell
& 'C:\xampp\php\php.exe' '.\tests\database\verify.php'
& 'C:\xampp\php\php.exe' '.\tests\app\verify.php'
& 'C:\xampp\php\php.exe' '.\tests\http\verify.php'
```

Verified results on October 25, 2026: **72 database assertions, 73 service assertions, and 61 HTTP assertions** (206 total), plus 18 checks against the real C-drive Apache/PHP server. Concurrent registration, scan, renewal, and payment workers passed. A separate recovery test restored a backup and matched all nine tables exactly, with both views present.

The suites create random `gym_test_*` databases and remove only their own verified test databases. The HTTP suite starts and stops a separate loopback PHP server with isolated session/configuration files. The application database is not reset. Tests cover SQL constraints, exact fees/dates, full registration/payment/check-in, invalid membership states, idempotency, real concurrent registrations/scans/renewals/payments, authentication, first-password change, cross-session revocation, login throttling, CSRF, escaping, QR output, reports, and logout.

The test account defaults to local root without a password and needs CREATE/DROP DATABASE rights. Override `GYM_TEST_DB_HOST`, `GYM_TEST_DB_PORT`, `GYM_TEST_DB_USER`, and `GYM_TEST_DB_PASSWORD` as needed. The HTTP suite needs the CLI curl extension and PHP `proc_open` enabled. Tests do not grant browser camera permissions or verify physical webcam focus.

## Backup and recovery

```powershell
& 'C:\xampp\php\php.exe' '.\scripts\backup.php'
```

This writes a consistent SQL backup under the ignored, web-blocked `storage/backups/` directory. Set temporary `GYM_BACKUP_DB_USER` and `GYM_BACKUP_DB_PASSWORD` environment values if the administrative database account differs from local root with no password. Passwords are passed to the dump process through its environment, never command arguments. Backups contain personal details, password hashes, and QR identifiers; keep them private. Restore only to a deliberately selected empty recovery database with the XAMPP client, verify row counts/constraints, then update private configuration if a recovery is needed. Never automatically reset the live database.

Run `& 'C:\xampp\php\php.exe' '.\tests\recovery\verify.php'` to create a backup, restore it to a fresh isolated database, compare every table's data, and remove only the recovery database. The private SQL backup is retained.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Localhost refuses the connection | Start C-drive Apache. Check whether another program owns port 80. |
| Database unavailable | Start C-drive MySQL and verify the private connection's host/port/account. Do not start E-drive MySQL on the same port. |
| A protected page returns login | Sign in again; sessions expire or may be revoked by a password change. |
| Form expired | Reload the form, sign in if needed, and submit with its current CSRF/form key. |
| Payment rejected | Use at most the outstanding balance; future-dated payments and nonpositive values are rejected. |
| Subscription rejected | Use the next available start date and an active plan. Overlaps and archived members are rejected. |
| Camera permission denied | Allow the camera in Brave site settings and Windows camera privacy settings; close other apps using it. Use localhost or HTTPS. |
| QR cannot be read | Include the white border, improve lighting/focus, and keep the full code in frame; try the QR-image fallback. |
| Already checked in | The original daily visit is retained. This is expected, including retries after a lost network response. |
| Report dates look different | Business filters use the gym timezone; exported event columns explicitly labeled UTC contain UTC timestamps. |
| Dependency class missing | Run Composer install from the lock file with PHP 8.2+ and mbstring enabled. |

See [the completion guide](docs/completed-system.md) for phase-to-file mapping and acceptance results. The old phase documents record earlier milestones; this README describes the current application.
