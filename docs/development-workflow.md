# Development workflow

Each student works in their own fork of the course repository. The mentor repository is the source of project specifications, API contracts, and lesson updates.

## Repository ownership

Mentor-owned paths:

- root `README.md`;
- `docs/`;
- `lessons/`.

Student-owned paths:

- `services/order/` for the Order Service track;
- `services/restaurant/` for the Restaurant Service track.

Students should not include changes to mentor-owned files in lesson submissions. This convention avoids most conflicts when course material is updated. Automated checks and review can detect accidental changes, but forks cannot be fully protected by settings in the mentor repository.

## Fork and update flow

1. Fork the mentor repository and clone the fork.
2. Keep the fork as the `origin` remote.
3. Add the mentor repository as the `upstream` remote.
4. Keep the fork's `main` branch synchronized with `upstream/main`.
5. Create a separate branch from `main` for each lesson.
6. Open the lesson pull request against `main` in the student's fork.
7. After review, merge the lesson and update `main` before starting the next one.

Example update sequence:

```bash
git switch main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
git switch -c lesson-01
```

Replace `upstream` if a different remote name is used.

## Specification updates

- The specification and API contract are frozen while a lesson is active.
- Corrections are recorded in [the changelog](changelog.md).
- Breaking contract changes are introduced between lessons rather than during one.
- A lesson release may be tagged, for example `lesson-01-v1`, so students can always recover the exact material used for their assignment.

If a student has not modified mentor-owned files, regular upstream updates should normally merge without conflicts.

## Working as two service teams

Order Service development uses a deterministic Restaurant Service substitute until real service communication is introduced. Restaurant Service development does not depend on a running Order Service. Both implementations follow the same shared contract and are connected in a later lesson.
