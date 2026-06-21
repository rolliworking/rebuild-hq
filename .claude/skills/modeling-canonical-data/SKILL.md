# Skill: modeling-canonical-data

## Purpose

Apply Rolliworks canonical data model principles to schema design. Closes the loop from architectural decisions (D-010, D-011, D-013) into actual table structures, ownership boundaries, and promotion lifecycles.

## When to invoke

- Designing or modifying any database table in the rebuild
- Adding a new entity to the ecosystem
- Resolving where a new piece of data should live (which schema, which app owns it)
- Deciding whether to denormalize or normalize
- Designing master tables, join tables, or cross-app contracts

## When NOT to invoke

- Documenting existing workflows (use `mapping-legacy-workflows`)
- Generating implementation tasks (use `generating-task-packets`)

## Inputs

- A proposed entity, table, column, or relationship
- The list of apps that will read/write the entity

## Process

1. **Classify the entity.** Is this a:
   - **Master** — authoritative shared data (customer, watch, reference, caliber, bracelet, dial, part)
   - **Operational** — app-specific transactional data (estimate, job, conversation)
   - **Derived** — computed from masters (analytics, embeddings, summaries)

2. **Assign the schema.**
   - Master → `shared.*`
   - App-specific operational → `rs.*` / `rw.*` / `rc.*`
   - AI module outputs → `ai_modules.<module>_*`
   - External system data (RolliTime, authenticator) → owns its own DB; reads from `shared.*`

3. **Define ownership.**
   - Who is the *write owner* for this entity? Exactly one app or role.
   - Who has read access? List all consumers.
   - For masters, who promotes from `stub` → `validated` (the human role)?

4. **Design the lifecycle.** For master tables, add status columns:
   - `status` (`stub`, `partial`, `complete`, `validated`, `authoritative`)
   - `validated_by` (FK to contributor)
   - `validated_at` (timestamp)
   - `confidence` (numeric, set by consensus engine when RolliCurator is active)

5. **Model relationships honestly.**
   - One-to-many → child table with FK
   - Many-to-many → join table (never comma-separated columns, never numbered slot columns like `bracelet1`, `bracelet2`)
   - Many-to-many with metadata → join table with extra columns (see D-013: `reference_caliber` with `era_start`, `is_primary`)

6. **Add audit columns to every table.**
   - `created_at`, `created_by`
   - `updated_at`, `updated_by`
   - Soft delete: `deleted_at`, `deleted_by` (never hard delete masters)

7. **Define the contract for cross-app reads/writes.**
   - App-to-app: which schema, which role, which auth pattern
   - External system: which API endpoint, which auth, which payload shape

## Output structure

For each entity, produce:

1. **Entity name and classification** (master / operational / derived)
2. **Schema placement** (`shared.*`, `rs.*`, etc.)
3. **Columns table** — name, type, nullable, default, FK, notes
4. **Lifecycle states** (if a master)
5. **Ownership** — write owner, read consumers, promotion owner
6. **Relationships** — explicit list of FKs in and out
7. **Audit columns** — confirm all present
8. **Cross-app contract** (if data crosses an app boundary)
9. **Open questions** — anything that requires another decision before implementation

## Anti-patterns

- ❌ Denormalized columns where a join table belongs (e.g. `bracelet1`, `bracelet2` instead of `reference_bracelets`)
- ❌ Free-text where an FK belongs (e.g. `caliber` text column when `caliber_id` FK exists)
- ❌ Multiple apps writing to the same master without a designated owner
- ❌ No audit columns (every change must be traceable)
- ❌ Hard deletes on master rows (always soft delete)
- ❌ Storing the same fact in two schemas (drift incoming)
- ❌ Comma-separated lists in a single column (forces parsing, kills queries)

## Mandatory checks before approving a schema

- [ ] Schema placement matches D-003 (correct schema for the entity type)
- [ ] Write ownership is exactly one app
- [ ] Audit columns present
- [ ] No denormalized many-valued columns
- [ ] FKs use stable codes, not display names
- [ ] If a master: lifecycle state column present, promotion owner defined
- [ ] If cross-app: contract documented in target app's integration file

## Output destination

- Schema proposals: `documentation/schema/<entity>.md`
- Cross-app contracts: `documentation/integrations/<system>-INTEGRATION.md`

## Related skills

- `mapping-legacy-workflows` — feeds the "what data exists" question
- `generating-task-packets` — turns schema decisions into migration tasks
- `reviewing-prs-governance` — verifies PRs follow these rules

## Related decisions

- D-002 (RS is source of truth for caliber/reference data)
- D-003 (shared Supabase, separate schemas)
- D-010 (canonical ecosystem data model)
- D-011 (living masters with promotion workflows)
- D-013 (reference and caliber mated via join table)
