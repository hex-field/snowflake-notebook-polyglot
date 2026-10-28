# Changelog

All notable changes to the `snowflake-notebook-polyglot` toolkit are documented here.

## [0.2.0] -- 2026-10-28

Fork and rename. This release marks the project's move from Snowflake Labs to
independent maintenance by Hex Field.

Forked from
[Snowflake-Labs/snowflake-notebook-multilang](https://github.com/Snowflake-Labs/snowflake-notebook-multilang)
at commit `497875fdab0c73132e4908e76f02796b85e92a06` (12 June 2026). The original
work is Copyright 2026 Snowflake Inc., licensed under Apache 2.0; that copyright
is retained. Files have been modified from their original form as set out below.

Version jumps from 0.1.3 to 0.2.0 to signal the discontinuity. The original was
never published to PyPI, so there is no released version being superseded.

### Changed -- BREAKING

- **Project renamed** `snowflake-notebook-multilang` → `snowflake-notebook-polyglot`.
- **Python package renamed** `sfnb_multilang` → `sfnb_polyglot`. All imports must
  be updated: `from sfnb_multilang.installer import ...` becomes
  `from sfnb_polyglot.installer import ...`.
- **Distribution name** is now `snowflake-notebook-polyglot`. The distribution name and import
  name differ deliberately: the former is descriptive for discovery, the latter
  short for everyday use.
- **Config files renamed** `cre_multilang_r.yaml` → `cre_polyglot_r.yaml`,
  `cre_multilang_ml.yaml` → `cre_polyglot_ml.yaml`.
- **Container assets renamed** `Dockerfile.multilang-r` → `Dockerfile.polyglot-r`;
  CRE image name `sfnb-multilang-r` → `sfnb-polyglot-r`; build artefact
  `sfnb-multilang.zip` → `sfnb-polyglot.zip`.
- **Build argument renamed** `SFNB_MULTILANG_REF` → `SFNB_POLYGLOT_REF`.
- Documentation and prose now say "polyglot" rather than "multi-language".

### Unchanged (deliberately)

- **Snowflake object names.** `MULTILANG_NOTEBOOK_EAI` (external access
  integration) and `MULTILANG_NOTEBOOK_EGRESS` (network rule) keep their current
  names. Renaming them would require every existing deployment to re-create
  Snowflake objects, so it is held back as a separate decision rather than
  bundled into this rename.
- **`sfnb-setup` CLI command** and the `sfnb_setup.py` helper keep their names.

### Install source

- **Pinned to release tags.** The `sfnb_setup.py` bootstrap now installs the
  toolkit from the `v0.2.0` release tag on GitHub instead of the tip of
  `main`, so every notebook gets the same code. Set `SFNB_POLYGLOT_REF` to
  override (e.g. `main` for development). The CRE image build uses the same
  variable and default, and now accepts tags as well as branch names.
- **Not on PyPI.** Installation docs (README, quickstart, CLI) now give the
  pinned GitHub URL rather than `pip install snowflake-notebook-polyglot`,
  which pointed at a PyPI project that doesn't exist.
- Derived `sfnb_setup.py` copies in `notebooks/` and `examples/` re-synced
  with the canonical source.
- `sfnb_polyglot.__version__` now matches the package version (0.2.0).

### Removed -- BREAKING

- **Account endpoint in generated EAI rules.** The network rule generator no
  longer adds `<account>.snowflakecomputing.com`. Workspace containers reach
  Snowflake over the internal SPCS network (`SNOWFLAKE_HOST` + session token),
  so the public endpoint was never needed and widened egress unnecessarily.
  The installer no longer auto-detects the account for this purpose. The
  `--account` CLI flag, the `account` argument to `generate_eai_sql()`,
  `apply_eai()` and `ensure_eai()`, and `network_rule.account` in YAML are
  removed (an `account` key in existing YAML files is ignored).

### Synced from upstream

Changes made in Snowflake-Labs/snowflake-notebook-multilang after the fork
(Murray Stokely, Aug–Sep 2026, upstream PRs #1 and #2), applied to the renamed
files:

- **CRE base image path** moved from
  `st_plat/runtime/x86/runtime_image/snowbooks` to
  `container_runtime/cpu_x86_64`, fixing builds in customer accounts.
  `SNOWBOOKS_TAG` bumped 2.5.0 → 2.9.0 (latest Container Runtime release).
- **CRE R image:** R 4.5.2 → 4.5.3; snowflakeR v0.2.0 and RSnowflake v0.2.2
  tarballs; zoo, cowplot, scales, patchwork and arrow baked in; `R_HOME` and
  `PATH` exported so the rpy2 pip install can build against the conda R.
- **Follow-up (Hex Field):** the same 2.9.0 / 4.5.3 defaults applied to
  `build_cre.sh`, `render_cre_profile.py` and `sfnb_ml.example.yaml`, which
  upstream left on the old values.

### Migration

Existing users need only update imports and the install source. Apart from the
upstream CRE changes above, no functional behaviour has changed in this release.

```diff
- !pip install "sfnb-multilang @ git+https://github.com/Snowflake-Labs/snowflake-notebook-multilang"
+ !pip install "snowflake-notebook-polyglot @ https://github.com/hex-field/snowflake-notebook-polyglot/archive/v0.2.0.zip"

- from sfnb_multilang.installer import install
+ from sfnb_polyglot.installer import install
```

## [Unreleased]

### Added

- **ML CRE profile (`sfnb_ml.example.yaml`):** second image tag `ml-v1` /
  `cre@sfnb_polyglot_ml` with tidymodels, xgboost, ranger, forecast, and
  foreach baked in; `configs/cre_polyglot_ml.yaml` runtime preset for Posit /
  credit-risk E2E demos. Keeps lean `sfnb_polyglot_r` v1 unchanged.
- **CRE `runtime_config`:** profile YAML selects which config file is copied
  into `/opt/sfnb/config/` at image build time.

### Fixed

- **CRE micromamba path:** `setup_notebook()` on a custom runtime no longer
  downloads micromamba to `/root/micromamba` when the image bakes R at
  `/home/jupyter/micromamba`. Honors `SFNB_MICROMAMBA_ROOT`, sets
  `MAMBA_ROOT_PREFIX` on micromamba subprocesses, and documents the path in
  `cre_polyglot_r.yaml`.
- **`%%R` `-i` / `-o` comma lists:** `-o p, mtcars` (space after comma) no longer
  leaves an orphan token that rpy2 injects into the R cell (`mtcarslibrary(...)`).
  `SafeRMagics` normalizes IO lists to `-o p,mtcars` before parsing.

## [0.1.3] - 2026-04-28

### Fixed

- **Mirror URL authentication:** `urlunparse` was silently decoding
  percent-encoded characters (`%40` -> `@`) in credentials, breaking
  basic-auth URLs for Artifactory/Nexus. Replaced with manual string
  construction in `inject_auth_into_url` and `mask_url_credentials`
  (affects `installer.py` and `sfnb_setup.py`).
- **Secret path case sensitivity:** `_normalize_secret_path` now
  lowercases the path, matching Snowflake's case-insensitive object
  names. Previously `MYDB.MYSCHEMA.SECRET` would fail to resolve via
  the container mount at `/secrets/mydb/myschema/secret`.
- **urllib stripping inline credentials:** `urllib.request` ignores
  `user:pass@host` in URLs. Micromamba and tarball downloads now
  extract credentials and pass them via an `Authorization: Basic`
  header instead (affects `micromamba.py` and `sfnb_setup.py`).
- **CRAN auth missing:** `install.packages()` was called without
  authentication when `auth_secret` was configured. The R plugin now
  reads mirror credentials and passes a `Basic` auth header via R's
  `headers` parameter (R >= 4.1.0).
- **Tarball auth not injected:** R package tarball URLs from the
  `tarballs` config section were not receiving mirror credentials.
  `install_r_packages` now injects auth before download.
- **Network rule parsing:** `DESCRIBE NETWORK RULE` returns a
  `VALUE_LIST` column (not a property row) for `PRIVATE_HOST_PORT`
  rules. `_get_rule_domains` now checks for this column first before
  falling back to property-based parsing (affects `network_rules.py`
  and `sfnb_setup.py`).

### Added

- **`pypi_extra_index`** mirror config option: supports dual-index pip
  installs (`--index-url` + `--extra-index-url`). Useful when packages
  are split across a primary Artifactory PyPI repo and a secondary
  internal index.
- **Pip install verification:** after installing pip packages, each
  package is import-tested and the result logged (installed OK / import
  FAILED).

### Improved

- **CRAN install logging:** reports total/already-installed/missing
  package counts, post-install success/failure per package, and
  forwards R stderr to the Python logger.

### Removed

- Unused helper functions `_rpy2_available` and `_rscript_path` from
  `sfnb_setup.py`.

## [0.1.2] - 2026-04-10

- Initial public release with R, Scala/Java, and Julia support.
- Custom mirror configuration (Artifactory, Nexus).
- EAI auto-generation and domain management.
- `setup_notebook()` single-cell bootstrap.
