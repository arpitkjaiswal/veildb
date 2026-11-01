# veildb

Field-level encryption for Python applications using PostgreSQL.

veildb explores a simple idea: a database can store sensitive information without
being trusted to read it. Protection happens inside the application, before
selected values reach PostgreSQL. A manifest describes the fields to protect,
and the ORM integration connects that policy to application queries and writes.

**Stage:** experimental research preview. Production security qualification is
still outstanding.

## Where it fits

An application may need a customer's email to send a message, while its database
only needs to store that email and find matching records. veildb separates those
responsibilities: the application handles plaintext and keys; PostgreSQL stores
encrypted values and explicitly enabled search terms.

```text
Application + key provider
          |
          | protect selected fields
          v
SQLAlchemy integration
          |
          | encrypted values + declared search terms
          v
      PostgreSQL
```

SQLAlchemy is the main integration point, with an optional SQLModel adapter.
Query support is bounded by the declared policy and the adapter's admission
checks. The host application remains responsible for authenticating users and
the tenant scope passed into a session.

## Inside the design

| Piece | Purpose |
| --- | --- |
| Protection manifest | Names protected models and fields, stable identifiers, tenant scope, and requested query capabilities. |
| Value encryption | CF1 uses AES-256-GCM-SIV, fresh random nonces, and HKDF-derived keys, with authenticated tenant, field, and record context. |
| Exact search | Separate HMAC-SHA-256 terms support declared equality queries. |
| Key providers | Separate key custody from database storage; adapters include AWS KMS, Google Cloud KMS, and OpenBao. |
| Lifecycle tooling | Code for planning protection, changing policies, rotating keys, and reversing protection of current data. |

Search carries tradeoffs. Equality terms reveal matching values; opting into
order queries reveals ordering. These choices need explicit review rather than
being enabled as invisible defaults. Unsupported operations must fail explicitly,
without plaintext fallback.

## Work on the project

The selected development stack is Python **3.12 or 3.13**, **uv**, PostgreSQL
**16**, SQLAlchemy, and psycopg. Dependency versions are recorded in
[pyproject.toml](pyproject.toml) and [uv.lock](uv.lock).

From the repository root, install the locked dependencies and optional adapters:

```sh
uv sync --frozen --all-extras
```

Read [AGENTS.md](AGENTS.md) and the [engineering playbook](ENGINEERING_PLAYBOOK.md)
before changing code. PostgreSQL checks must go through
[scripts/test_postgres.py](scripts/test_postgres.py), using private role files and
an explicitly authorized disposable database. Keep test credentials out of the
checkout and use synthetic data for development key custody.

Follow the [local development guide](docs/local-development.md) to provision an
isolated PostgreSQL 16 cluster and run the suite. The [application guide](docs/application.md)
covers configuration and usage; the [compatibility matrix](docs/compatibility.md)
defines the admitted operations and their limits.

## Find your way around

| Location | Contents |
| --- | --- |
| [src/](src/) | Encryption, manifests, ORM adapters, and lifecycle implementation. |
| [tests/](tests/) | Database integration, cryptography, and lifecycle checks. |
| [docs/](docs/README.md) | Usage, design contracts, development instructions, and recorded evidence. |
| [scripts/](scripts/) | PostgreSQL verification, qualification, and release helpers. |
| [qualification/](qualification/) | Dependency inputs for isolated qualification workflows. |
| [spikes/](spikes/) | Research experiments, separate from the package's supported behavior. |
| [tools/](tools/) | Research and scanning utilities. |
| [CHANGELOG.md](CHANGELOG.md) | Preview version notes and scope. |

## Boundaries

The code is a prototype, not evidence of a completed security review. Provider
adapters have their own verification limits, and local checks do not qualify
production identity, backup custody, or key destruction. Broad SQL compatibility,
production readiness, and performance advantages are not claimed here.

Decrypting current data does not erase historical backups or previously disclosed
information. Recovery, retention, and removal need their own operational review.

## License

Maintained by [arpitkjaiswal](https://github.com/arpitkjaiswal).

Distributed under the [MIT License](LICENSE). See that file for the copyright
notice and license terms.
