# React Native Local Storage and Database Migrations

A practical guide for changing persisted mobile data safely across application upgrades, account changes, interrupted migrations, and older app versions.

## Treat local data as a versioned interface

Persisted data outlives the JavaScript process and often survives many releases. A change that works on a clean install may fail for a user upgrading through several historical versions.

Common persisted data includes:

- Key-value preferences and onboarding state.
- Authentication/session references.
- SQLite or object-database records.
- Offline mutations and synchronization metadata.
- Downloaded files, images, and generated documents.
- Feature flags and cached API responses.

Every durable format needs an owner, a version, an upgrade strategy, and a deletion policy.

## Inventory storage before changing it

Record:

| Storage | Contents | Sensitive? | Account-scoped? | Migration owner |
|---|---|---:|---:|---|
| Secure storage | Session material | Yes | Yes | Authentication |
| Key-value storage | Preferences | Sometimes | Maybe | App platform |
| Local database | Offline records | Often | Yes | Data/sync |
| File cache | Downloaded media | Maybe | Often | Feature team |

Also record encryption, backup behavior, maximum size, cleanup rules, and whether server data can rebuild it.

## Separate durable data from disposable cache

Classify each value as:

- **Source of truth:** cannot be discarded without data loss.
- **Pending work:** must be synchronized or deliberately abandoned.
- **Rebuildable cache:** may be removed and fetched again.
- **Preference:** may have a safe default.
- **Credential:** requires protected storage and special lifecycle handling.

Do not solve a migration failure by clearing all storage unless product requirements explicitly allow that data loss.

## Version schemas explicitly

Use a monotonically increasing schema/data version. Keep migration code deterministic and reviewable.

```ts
type Migration = {
  fromVersion: number;
  toVersion: number;
  run: () => Promise<void>;
};

const migrations: Migration[] = [
  {fromVersion: 1, toVersion: 2, run: migrateFrom1To2},
  {fromVersion: 2, toVersion: 3, run: migrateFrom2To3},
];
```

Store the version with the data it describes. Do not infer it only from the current application version; multiple app releases may share or skip a storage schema version.

## Prefer small forward migrations

A safe migration usually:

1. Reads the current schema version.
2. Validates that the version is supported.
3. Applies one ordered transition at a time.
4. Writes data and the new version atomically where supported.
5. Verifies important invariants.
6. Records a sanitized success or failure event.

Keep transitions such as `1 → 2` even after adding `2 → 3` when supported users can still upgrade from version 1.

## Make migrations restartable

The OS can terminate an app during migration. Battery loss, low storage, a crash, or a user force-close can also interrupt it.

Design for one of these approaches:

- Use a database transaction that fully commits or rolls back.
- Write new data to a temporary location, validate it, then switch atomically.
- Record migration checkpoints when work cannot be transactional.
- Make each step idempotent so running it again produces the same result.

Never increment the stored version before the corresponding data change is safely complete.

## Use transactions for database changes

The exact API depends on the database library, but the intended behavior is:

```ts
await database.transaction(async transaction => {
  await transaction.execute('ALTER TABLE tasks ADD COLUMN priority INTEGER');
  await transaction.execute('UPDATE tasks SET priority = 0 WHERE priority IS NULL');
  await transaction.setSchemaVersion(2);
});
```

This is illustrative pseudocode. Use parameterized statements and the transaction API provided by the selected database.

For large tables, measure migration duration and memory use on low-end devices. A single enormous blocking migration may cause startup hangs or OS termination.

## Avoid destructive schema shortcuts

> [!WARNING]
> Dropping a table, deleting a database, clearing application data, or reinstalling the app permanently removes local user data that may not exist on the server.

Before destructive transformation:

- Confirm whether any record is unsynchronized or locally unique.
- Export or copy required fields into the replacement schema.
- Verify counts and important relationships.
- Define recovery behavior for insufficient storage.
- Obtain the appropriate product/security review for sensitive data.

Prefer additive changes first: add a nullable field or new table, populate it, move readers, and remove the old representation in a later release.

## Migrate key-value data deliberately

Key renames and shape changes need the same care as database schemas.

```ts
const oldValue = await storage.getItem('user_settings');

if (oldValue !== null) {
  const parsed = safelyParseOldSettings(oldValue);
  await storage.setItem('settings_v2', JSON.stringify(convertSettings(parsed)));
  await storage.removeItem('user_settings');
}
```

Handle invalid JSON, unexpected historical values, missing keys, and partial writes. Use safe defaults only for fields where data loss is acceptable.

## Keep account data isolated

Namespace user-specific data with a stable internal account identifier, not an email address or display name.

On logout or account switch:

- Stop synchronization for the old account.
- Resolve, discard, or quarantine pending mutations according to product rules.
- Clear sensitive in-memory state.
- Prevent cached queries and media from appearing under the next account.
- Remove credentials independently from ordinary preferences.

Treat cross-account data exposure as a security and privacy incident.

## Plan for downgrade and rollback

Once a new app writes a new schema, an older binary may not understand it. Store rollback does not guarantee that every device returns to an earlier data format.

Choose a strategy:

- Keep changes backward-readable for a defined window.
- Use additive fields that older versions ignore.
- Block unsafe downgrade with a clear recovery path.
- Keep risky features behind a remotely controlled flag while migration health is observed.
- Restore server behavior without requiring the client to reverse its local schema.

Reverse migrations are often riskier than forward fixes. Test them only when they are an explicit supported requirement.

## Handle encryption and key changes

- Keep encryption keys separate from encrypted data.
- Use platform-protected key storage where appropriate.
- Define what happens when keys are invalidated or unavailable.
- Rotate keys using a recoverable, staged process.
- Verify backup/restore behavior on Android and iOS.
- Never log raw keys, decrypted records, or complete database files.

Encryption does not replace account isolation, access control, retention, or secure deletion decisions.

## Avoid blocking application startup unnecessarily

Show an intentional migration state instead of a frozen splash screen. For long work:

- Measure progress only when it is accurate and useful.
- Keep the migration resumable.
- Prevent normal readers/writers from using a partially migrated store.
- Allow safe retry after recoverable failures.
- Provide a support path for unrecoverable data.

Do not let two application processes or tasks start the same migration concurrently.

## Test real upgrade paths

For every supported prior version:

1. Install the old released build.
2. Create realistic data, including edge cases and pending offline work.
3. Upgrade without uninstalling or clearing data.
4. Launch under low-storage and offline conditions where practical.
5. Verify record counts, relationships, user-visible data, and sync behavior.
6. Restart the app during or immediately after migration.
7. Test logout, account switch, backup/restore, and the next upgrade.

Do not rely only on databases generated by the current test suite. Keep sanitized fixtures representing important historical schemas.

## Diagnose migration failures

Capture only safe diagnostics:

```text
App version/build
Platform and OS version
Previous and target schema versions
Migration step identifier
Database/library error category
Available storage category
Duration and record counts (when non-sensitive)
Recovery result
```

Never upload a user's entire local database as an automatic crash attachment.

Common causes include:

- Version updated before data commit.
- Historical schema not covered by the migration chain.
- Non-null field added before values exist.
- Duplicate or invalid data violating a new constraint.
- Migration runs twice and is not idempotent.
- Disk is full during a copy or index build.
- Encryption key is unavailable after device restore.
- New account reads a previous account's shared storage.

## Release migration changes safely

- Test on both clean install and upgrade.
- Include migration duration and failure rate in release monitoring.
- Use staged rollout for meaningful schema changes.
- Preserve the previous release fixture and reproduction instructions.
- Keep backend APIs compatible with older clients.
- Avoid combining a risky migration with unrelated dependency upgrades.
- Define pause and support thresholds before rollout.

## Quick migration checklist

- [ ] Durable data and disposable caches are identified.
- [ ] Schema/data version is explicit.
- [ ] Every supported old version has an ordered upgrade path.
- [ ] Migration is transactional or safely restartable.
- [ ] Interrupted migration is tested.
- [ ] Unsynchronized data is preserved.
- [ ] Account switching cannot expose old data.
- [ ] Low-storage and corrupted-data behavior is defined.
- [ ] Diagnostics contain no sensitive records.
- [ ] Staged rollout and recovery owners are assigned.

## Related guides

- [Offline Mode and Network Retry Strategies](offline-mode-and-network-retry-strategies.md)
- [Mobile Security and Sensitive Data Handling](mobile-security-and-sensitive-data-handling.md)
- [App Store Submission and Release Management](app-store-submission-and-release-management.md)
- [Crash Reporting and Symbolication](crash-reporting-and-symbolication.md)

## Official references

- [Android: Save Data in a Local Database Using Room](https://developer.android.com/training/data-storage/room)
- [Android: Data and File Storage Overview](https://developer.android.com/training/data-storage)
- [Apple: Core Data](https://developer.apple.com/documentation/coredata)
- [React Native Security](https://reactnative.dev/docs/security)
