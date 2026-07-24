# Day 4 status (Task 3)

## Supply chain snapshot
- `./mvnw dependency:tree` reports **93** dependency entries (direct + transitive).
- Traced transitive example 1: `org.apache.tomcat.embed:tomcat-embed-el:10.1.30` <- `org.springframework.boot:spring-boot-starter-validation:3.3.4`.
- Traced transitive example 2: `com.github.docker-java:docker-java-transport-zerodep:3.4.2` <- `org.testcontainers:testcontainers:1.21.4`.
- CVE lookup sample: `org.springframework:spring-web:6.1.13` currently maps to medium advisories (for example `CVE-2024-38820`, `CVE-2025-41234`); fixed line is 6.1.21+ per advisory metadata.

## Tested at which altitude
- Unit (`com.fx.api.ConversionServiceTest`): conversion rounding, fee/net math, unknown pair exception.
- Web slice (`com.fx.api.web.RateControllerTest`): latest rates endpoint, 404 unknown pair, valid conversion 201, invalid amount 400.
- Integration (`com.fx.api.repo.RateRepositoryIT`): real MySQL container + Liquibase migration path, `findLatest()` size 10, EUR/USD 1.0818, unknown pair empty.

## What CI guards
- `.github/workflows/ci.yml` runs `mvn -B verify` in `build`, `mvn -B validate` in `lint`, and OWASP scan in `dependency-check`.
- `build` includes surefire (`*Test`) and failsafe (`*IT`) so all three test altitudes are covered.
- Branch protection is expected to require `build` before merge.

## What CI does not guard
- No full end-to-end user-flow check that boots full compose stack and validates API from the outside.
- No frontend verification yet (Week 3 scope).
- Security scan is non-blocking (`continue-on-error: true`), so findings still need human triage.

## Honesty checks run today
- Fast tier check (`./mvnw test`): `Tests run: 29, Failures: 0, Errors: 0, Skipped: 1` (green).
- Docker-off check (`./mvnw verify` after stopping Docker backend processes): `RateRepositoryIT` showed `Tests run: 3, Skipped: 3` and build still reported SUCCESS.
- Docker restored and `./mvnw verify` rerun: `RateRepositoryIT` back to `Skipped: 0`.
- Local MySQL Windows service stop (`MySQL80`) was blocked by OS permissions (`Access denied`), so service-level shutdown needs elevated terminal when repeating the MySQL-off proof.

## Dependabot rule
- Team rule: Dependabot PRs are reviewed within 1 working day; security-related updates are prioritized for same-day merge if `build` is green.
- Repo setting action required: enable Dependabot alerts and security updates in GitHub `Settings -> Code security and analysis`.

## Definition of Done (DoD)
A change is done only when all of these hold:

1. From `fx-app-spring/`, `./mvnw verify` is green for unit + slice + integration, with nothing wrongly skipped (read summary lines), and `docker compose up` serves `/api/rates` with 10 rows.
2. New behavior has tests at the right altitude: endpoint changes have slice tests (happy + failure), calculation changes have unit boundary tests, SQL changes have `*IT` coverage.
3. A teammate reviewed the PR, checked out the branch, ran `./mvnw verify`, and approved.
4. Merge to `main` happens only via PR (no direct push).
5. `main` stays green after merge, and a fresh clone can build and verify.

Not done: failing tests, wrongly skipped tests, "works only on my machine", or unreviewed direct merges.

