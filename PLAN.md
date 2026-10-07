# Task Management Application Plan

## 1. Final Goal and Scope

Build a small Laravel application to learn how to execute work using Sol and subagents. The application will display tasks, create a task, edit its title, delete it, and mark it as complete or incomplete, with data stored in SQLite.

The interface has only two pages: a task list containing the creation form, and a page for editing a task title. No authentication, API, association between tasks and users, or new packages. No descriptions, priorities, deadlines, search, filters, or notifications. Use the existing tools without adding service or repository layers or a separate JavaScript frontend.

**This stage is planning only.** None of the implementation tasks below have started; creating this file does not authorize implementation to begin now.

## 2. Current Project State

The project was inspected on 2026-10-07 at `/workspace/Task-Manager`, at commit `73e0777`. The Git working tree was clean before this plan was added.

The inspection covered the project file inventory, its instructions, all application source in `app/`, `routes/`, `database/`, `bootstrap/`, `config/`, `resources/`, and `tests/`, entry-point files in `public/`, and package, build, and documentation files. Libraries in `vendor/` and `node_modules/` were not reviewed file by file; their presence and versions were checked. Secrets in `.env` and user data were not read; the SQLite schema and migration history were inspected in read-only mode.

| Area | Verified State |
|---|---|
| Laravel | `composer.json` requires `^13.17`; the locked and installed version is `13.35.0`. The project requires PHP `^8.3`. |
| Application | The base `Controller` is empty, and `User` and `AppServiceProvider` are the defaults. There is no task model or controller. |
| Routes | `GET /` renders `welcome`; the `/up` health route exists. There are no task, API, or authentication routes. JSON exception handling in bootstrap does not create an API. |
| Database | A local SQLite database exists, and the three default migrations for users, sessions, cache, and queues have run. There is no `tasks` table. The default user model, factory, and seeder are not needed for this work. |
| Configuration | `.env.example` and the default configuration use SQLite and database-backed sessions and cache. A local `.env` exists, but its values were not read; the local runtime database must be confirmed during implementation. |
| Interface | Only the Laravel welcome page exists; CSS has the Tailwind setup, and JavaScript is effectively empty. Conditional login links in the welcome template do not indicate an actual authentication system. |
| Build | Vite `8.3.3` and Tailwind `4.3.3` are installed. `vite.config.js` loads the Instrument Sans font from Bunny. `public/build` exists locally and is ignored by Git; the old build cannot establish acceptance of the new interface. |
| Tests | PHPUnit `12.5.38`; one example unit test and one feature test expecting `200` from `/`. `phpunit.xml` configures SQLite `:memory:` and `array` sessions and cache. There are no task tests. |
| Dependencies | `vendor/` and `node_modules/` exist; `composer.lock` exists, and the project has no `package-lock.json`. Package installation is not needed in the current environment. |
| Tools | After environment activation: PHP `8.4.26`, Composer `2.8.12`, Node `24.19.0`, npm `11.9.0`, and the `pdo_sqlite` and `sqlite3` extensions are available. |
| Planning Verification | `composer check-platform-reqs --no-interaction` passed. Application tests, builds, and migrations were not run during planning; their success has not been established. |

PHP and Composer are not on the default PATH in the current environment. Run the following before implementation commands in each new shell:

```bash
cd /workspace/Task-Manager
source /workspace/.task-manager-setup/activate.sh
```

The activation path is specific to this environment; elsewhere, tools compatible with the package requirements are sufficient. `AGENTS.md` and `CLAUDE.md` instruct agents to install Laravel Boost before modifying the application. It will not be added here because the user's constraint prohibits unnecessary packages and the current task is planning only. Instruction and dependency files remain unchanged.

## 3. Architecture and Shared Contract

The request flow is simple: browser → `routes/web.php` → `TaskController` → the `Task` model and SQLite, followed by a Blade response or redirect. Use Laravel's existing web middleware, sessions, and CSRF protection.

### Data

The only required database addition is the `tasks` table:

| Field | Type and Behavior |
|---|---|
| `id` | The default auto-incrementing identifier. |
| `title` | Required text, up to 255 characters. |
| `is_completed` | Boolean with a database default of `false`, cast to boolean in the model. |
| `created_at`, `updated_at` | Standard timestamps. |

The model allows saving `title` and `is_completed`, but each HTTP action passes only its permitted fields. No Task factory or seeder is needed; tests create their data directly.

### Routes Fixed Before Delegation

| HTTP | Path | Name | Action |
|---|---|---|---|
| GET | `/` | — | Redirect to `tasks.index`. |
| GET | `/tasks` | `tasks.index` | `index`: the list and its inline creation form. |
| POST | `/tasks` | `tasks.store` | `store`: create using a title only, initially incomplete. |
| GET | `/tasks/{task}/edit` | `tasks.edit` | `edit`: the title editing form. |
| PUT | `/tasks/{task}` | `tasks.update` | `update`: change the title only. |
| DELETE | `/tasks/{task}` | `tasks.destroy` | `destroy`: delete the task. |
| PATCH | `/tasks/{task}/completion` | `tasks.completion` | `completion`: set the requested completion state. |

### Behavior All Agents Rely On

- `index` passes `$tasks` to `tasks.index`, ordered by `id` descending, with a clear empty-list message. Pagination is unnecessary for this small scope.
- `edit` passes `$task` to `tasks.edit`, showing its current title and a link back to the list.
- Creation and editing validate `title` using `required|string|max:255`. Display validation errors and previous input using standard Laravel and Blade mechanisms.
- A new task remains incomplete even if the client submits an extra completion field. Editing its title does not change its completion state.
- The `completion` action accepts `is_completed` with `required|boolean` validation. The interface sends the target value, `0` or `1`, rather than a blind toggle; submitting the same value again does not reverse the state.
- Successful operations redirect to `tasks.index` with a short message under the `success` session key. Use route model binding to return `404` when a task is missing.
- HTML forms use `@csrf` and `@method` as needed, and titles use the default `{{ }}` escaping. Do not disable CSRF protection.
- A simple Arabic interface with RTL direction, clear field labels, and text buttons. Use the existing Blade and Tailwind setup without new JavaScript or a UI library.
- Keep the default user files, migrations, and unused welcome page; removing them does not serve the goal.

## 4. Tasks and File Ownership

All paths below are relative to the project root. Ownership means exclusive write access during the task; everyone may read files. An agent must not create files outside its list. Sol manages generated outputs ignored by Git according to the verification rules below.

| Task | Specific Work | Dependencies | Owned Files | Suggested Model |
|---|---|---|---|---|
| T0 — Prepare and Fix the Contract | Activate tools, record the baseline, confirm the local SQLite database and in-memory test configuration, then send the contract and file ownership to agents. | Subsequent authorization to begin implementation. | `PLAN.md` for status updates only; local `.env` if needed for runtime setup, without exposing secrets. | Sol, implementation lead. |
| T1 — Task Data | Create the Task model and tasks migration according to the contract, and run the additional migration only against the confirmed local development database. | T0 passes. | `app/Models/Task.php`, `database/migrations/2026_10_07_000001_create_tasks_table.php`. | Luna, a small and clearly defined task. |
| T2 — Web Requests | Implement the six actions in one controller, connect routes, redirect `/`, and validate input. | T1 is complete and the contract is fixed; behavioral acceptance waits for T3 and T4. | `app/Http/Controllers/TaskController.php`, `routes/web.php`. | Sol. |
| T3 — Blade Interface | Create the layout, list, inline creation form, editing page, deletion and completion buttons, and success and error messages. | T1 is complete and the contract is fixed; testing actual rendering waits for T2. | `resources/views/layouts/app.blade.php`, `resources/views/tasks/index.blade.php`, `resources/views/tasks/edit.blade.php`, `resources/css/app.css` if needed, and `vite.config.js` only if removing the external font dependency is necessary to fix the build. | Luna. |
| T4 — Behavior Tests | Write the required feature tests and update the `/` test to expect a redirect. | T1 is complete and the contract is fixed before writing; final execution waits for T2 and T3. | `tests/Feature/TaskManagementTest.php`, `tests/Feature/ExampleTest.php`. | Sol. |
| T5 — Integration and Delivery | Review the change scope, run sequential verification, exercise the interface, document startup, and fill in the completion report. | T2, T3, and T4 finish writing; acceptance requires every verification check to pass. | `README.md`, `PLAN.md` after T0 relinquishes ownership; read other files and return fixes to their owners. | Sol, implementation lead. |

Terra is not required for this plan; the current complexity suits Sol and Luna. These names are recommendations for later implementation. If Luna is unavailable, the lead uses Sol for the same task without changing its scope or assuming an unavailable model exists.

## 5. Parallel Work and Subagent Execution

Sequence: **T0 → T1 → T2, T3, and T4 in parallel → T5**.

1. Sol performs T0, then assigns T1 to one agent and waits for its result.
2. After T1, launch only three subagents: web requests, interface, and tests. An agent per CRUD operation is unnecessary because those operations share a controller and route file.
3. Send each agent its task ID, the contract from section 3, owned files, dependencies, verification commands, required evidence, and the prohibition on modifying others' work.
4. During parallel work, each agent writes its own files and performs only their local syntax checks. Agents send status updates to the lead instead of writing them in `PLAN.md`; the lead does not edit their files while they are working.
5. Once writing finishes, Sol alone runs builds, Artisan commands, tests, and Pint in check-only mode, sequentially. Do not run shared operations such as `view:cache`, `config:clear`, builds, or migrations concurrently, or run a broad formatter that writes to other agents' files.
6. Files are shared in the subagent environment; this small task does not require multiple branches or worktrees. The lead reviews changes without creating a commit, pull request, or deployment within this scope.

Task states: `Not started`, `In progress`, `Waiting for dependencies to verify`, `Complete`, and `Blocked`. Finishing test code before the controller is available is neither a failure nor accepted completion; the task remains waiting for integration verification.

## 6. Verification and Required Evidence for Each Task

The commands below are for later implementation from the project root after environment activation. T0 and T1 commands precede parallel work. Local `php -l` checks may run during parallel work; the lead runs the remaining acceptance checks after writing stops.

| Task | Verification Command | Required Acceptance Evidence |
|---|---|---|
| T0 | `php -v`, `composer -V`, `composer check-platform-reqs --no-interaction`, `node -v`, `npm -v`, `php artisan test`, `git status --short`. | Tool versions, passing platform requirements, actual baseline test results, confirmation of local SQLite and the `:memory:` test database without secrets, and a record of any existing user changes. |
| T1 | `php -l app/Models/Task.php`, `php -l database/migrations/2026_10_07_000001_create_tasks_table.php`, `php artisan migrate --no-interaction`, `php artisan migrate:status`. | Passing syntax checks, the new migration shown as executed, and review of field definitions, the default value, and the cast. T4 tests later establish actual persistence and the default creation state. |
| T2 | `php -l app/Http/Controllers/TaskController.php`, `php -l routes/web.php`; after integration: `php artisan route:list --except-vendor` and `php artisan test --filter=TaskManagementTest`. | Routes, methods, and names match the contract, and CRUD, validation, completion, and 404 tests pass, without new authentication or API routes. |
| T3 | After integration: `npm run build`, `php artisan view:cache`, then `php artisan view:clear`, and `php artisan test --filter=TaskManagementTest`. | Successful build and Blade compilation, passing list and editing-page rendering tests, and a record of visually exercising both pages and their buttons. Blade compilation alone does not prove routes or view variables are correct. |
| T4 | `php -l tests/Feature/TaskManagementTest.php`, `php -l tests/Feature/ExampleTest.php`; after integration: `php artisan test`. | Actual test and assertion counts, exit code zero, and database assertions after operations, without disabling tests or changing valid expectations to conceal a failure. |
| T5 | The acceptance sequence below, then `git diff --check` and `git status --short`. | Commands and results, files within the ownership boundaries, manual verification evidence, usable README instructions, and a final report with no required verification left pending. |

Each agent sends a short report to the lead: task ID and state, files created or changed, work completed, verification commands with exit codes and result summaries, and any dependency being awaited or reason for blockage. Saying "done" without this evidence is insufficient. New untracked files must be included in the report; `git diff` alone does not show their contents.

### T4 Test Scope

`TaskManagementTest` uses `RefreshDatabase` and the existing `phpunit.xml` configuration. It must not test against local `database/database.sqlite` and does not need a new test package.

- `/` redirects to `/tasks`; update `ExampleTest` accordingly without following the redirect.
- Display the empty list, stored tasks in the agreed order with their states, and the editing form for an existing task.
- Create a valid title and save an incomplete task, even when an extra completion field is submitted.
- Reject missing, empty, non-string, or over-255-character titles on creation and editing; accept the 255-character boundary, and do not create or change data when validation fails.
- Edit the title while preserving completion state, and actually delete the task from the database.
- Set completion to `1` and then `0`, repeat submission of the same state, and reject missing or invalid values without changing the state.
- Return `404` for a missing task during editing, updating, deletion, and completion changes.
- Render a title containing HTML tags as escaped text. The lead reviews forms for CSRF and method spoofing; ordinary Laravel tests, which bypass CSRF by default, do not establish that protection on their own.

### Final T5 Acceptance Sequence

The lead runs shared checks once and uses their results to accept dependent tasks rather than repeating checks without a reason:

```bash
php artisan route:list --except-vendor
npm run build
php artisan view:cache
php artisan view:clear
php artisan test
vendor/bin/pint --test app/Models/Task.php app/Http/Controllers/TaskController.php routes/web.php database/migrations/2026_10_07_000001_create_tasks_table.php tests/Feature/TaskManagementTest.php tests/Feature/ExampleTest.php
git diff --check
git status --short
```

Stop the sequence at the first failure and rerun affected verification after fixing it. Do not run `pint` in broad write mode during parallel work. This application does not require running queues or Redis or adding CI tooling.

For manual verification, the lead runs `php artisan serve --host=127.0.0.1 --port=8000` and opens the application in a browser: view the list, create a task, edit its title, mark it complete and then incomplete, reload to confirm persistence, and delete it. Also submit an empty title to inspect the error and retained input. Record actual results, check button clarity, and check for disruptive horizontal scrolling on a narrow screen. If browser access is unavailable, this check remains unverified; HTTP tests do not justify claiming visual verification.

Write `README.md` startup and testing instructions suitable for the current environment. For a fresh checkout, describe `composer install` using the existing lockfile, setting up `.env`, the key and SQLite file, running migrations, and using `npm install --ignore-scripts` and the build command. Do not recommend `npm ci` without an npm lockfile or regenerating an existing key. Do not run `composer setup` in the current environment because it bundles changes that are unnecessary here.

## 7. Failure Handling

1. **Identify the cause:** Record the command, concise error, and affected file, distinguishing an implementation error, an unfinished dependency, and an environment issue. Do not delete tests or add an arbitrary package to bypass the error.
2. **Failure within a task's scope:** The file owner fixes it and reruns the affected check. After two unsuccessful attempts, or if the contract is ambiguous, send evidence to Sol instead of broadening the change.
3. **Contract change or an unowned file:** Sol pauses affected tasks, decides the smallest change, and updates the contract and ownership before resuming. Never give two parallel tasks write access to the same file; independent tasks may continue.
4. **File conflict or out-of-scope modification:** Stop the responsible writer and review the diff with the file owner. Do not use `git reset --hard` or delete user changes. Return the fix to a single owner.
5. **Tool or build issue:** First activate PHP and Composer using the existing path. If downloading the Bunny font specifically fails, the T3 owner removes that unnecessary dependency by using system fonts and updating its owned `vite.config.js` and CSS files, then rebuilds. Do not change packages or lockfiles for this reason.
6. **Migration or data issue:** Do not use `migrate:fresh` or `db:wipe` on the development database, or run the default seeder unnecessarily. Inspect and fix the targeted migration while preserving data; only tests use the isolated in-memory database.
7. **Integration failure:** Sol returns work to the original owner and waits for the fix to finish before running shared checks. If a blocker remains, mark the task `Blocked`, record what is needed to resolve it, and do not declare the application complete. Ask the user for clarification only when a decision falls outside this scope.

## 8. Final Completion Report

### Outcome of the Current Planning Stage

- Project inspection, task breakdown, contracts, dependencies, and file ownership are complete.
- The required output of this stage is `PLAN.md` only. No application code was implemented, no packages were added, and the database was not modified.
- Verification performed: tool and Composer platform checks, and review of source, versions, and the database schema. Application tests, builds, and manual interface verification were not performed during planning.

### Implementation Report for Sol to Complete at the End of T5

| Task | Current State | Evidence After Implementation |
|---|---|---|
| T0 | Not started | — |
| T1 | Not started | — |
| T2 | Not started | — |
| T3 | Not started | — |
| T4 | Not started | — |
| T5 | Not started | — |

Sol replaces these entries with actual results and adds a concise report covering:

- Outcome: complete or incomplete, and the five functions actually exercised.
- Added and changed files, each group's owner, and any justified departure from the plan.
- Verification commands, exit codes, test and assertion counts, and build, migration, and manual verification results.
- Problems encountered, how they were resolved, and anything still blocked or unverified.
- Startup instructions from the README and confirmation that the scope remains free of authentication, API, and additional packages.

**Completion criteria:** All five functions work and data persists after reload; behavior tests, build, and formatting checks pass; the interface is visually reviewed; and changes remain within owned files. Do not declare completion when required evidence is missing.
