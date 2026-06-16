# raven-data

Downloadable **data artefacts and external-tool binaries** for
[RAVEN](https://github.com/SysBioChalmers/RAVEN) (MATLAB) and
[raven-toolbox](https://github.com/SysBioChalmers/raven-toolbox) (Python).

This repository hosts the large files those tools fetch on demand — KEGG reference
data, prebuilt HMM libraries, and the BLAST+/DIAMOND/HMMER binaries — as **GitHub
Release assets**, indexed by a single [`manifest.json`](manifest.json). Keeping them
here (instead of attached to the code repositories) keeps the code releases clean,
isolates the assets' upstream licenses, and gives both the MATLAB and Python tools
one shared, versioned source of truth.

> The HMM libraries are ~135–155 MB each — past GitHub's 100 MB per-file git limit —
> so they can only live as release assets, never committed into a repo's tree.

## What's here

| Kind | Tags | Contents |
|---|---|---|
| **KEGG data** | `kegg118`, … | reference-model + tables (`*_core.tar.gz`), `*_taxonomy.gz`, prokaryote/eukaryote HMM libraries (`*.hmm.gz`) |
| **Binaries** | `blast-<v>`, `diamond-<v>`, `hmmer-<v>` | per-platform ZIPs `<bundle>-<version>-<os>-<arch>.zip` (executables + license + any runtime DLL) |
| **Manifest** | `manifest-v<N>` (+ [`manifest.json`](manifest.json) on `main`) | the index: every asset's URL + SHA256 + size |

## How the versioning works (no duplication)

Each artefact has its **own immutable release tag**, versioned by its *upstream*
version — `blast-2.17.0`, `diamond-2.1.17`, `hmmer-3.4.0`, `hmmer-3.3.2` (native
Windows), `kegg118`. An asset is uploaded **once** under its tag and **never
re-uploaded**. When a tool bumps (say DIAMOND → 2.1.18) you publish a *new* tag
`diamond-2.1.18` with only that tool's ZIPs; every other tag is untouched.

There is **no single "raven-data version"**. The role of "a snapshot of the whole
set" is played by **`manifest.json`** (tiny text), versioned as `manifest-v1`,
`manifest-v2`, … — bumped only when the *pointer set* changes. The binaries
themselves are stored once and merely re-referenced across manifest versions.

`manifest_version` inside the JSON is the **schema** version (see
[`manifest.schema.json`](manifest.schema.json)), independent of the content snapshot.

## How consumers use it

Both tools read the same `manifest.json` and verify every file's SHA256 after
download:

- **raven-toolbox (Python)** bakes a pinned snapshot of the manifest into its
  default registries (`raven_toolbox.data` / `raven_toolbox.binaries`), so a given
  code release always fetches the exact, checksum-verified assets it was tested
  against. `RAVEN_PYTHON_MANIFEST=<url|path>` overrides it with a newer manifest.
- **RAVEN (MATLAB)** downloads the KEGG HMM libraries from the corresponding
  `kegg<NNN>` release.

Pinning is by `(tag, asset, sha256)`, so the two tools coordinate **through the
manifest**, not by matching version numbers.

## Publishing (maintainers)

Assets are produced and published from **raven-toolbox**'s scripts:

```bash
# in the raven-toolbox checkout:
python scripts/build_binary_bundles.py        # -> dist/binaries/*.zip (from RAVEN's vetted binaries)
python scripts/build_kegg_artefacts.py ...     # -> kegg<NNN>_*.{tar.gz,hmm.gz,gz}

# upload to this repo's releases (idempotent; skips assets already present):
python scripts/publish_to_raven_data.py binaries --dir dist/binaries
python scripts/publish_to_raven_data.py release --tag kegg118 --dir <kegg dir>
python scripts/publish_to_raven_data.py release --tag manifest-v1 data/manifest.json
```

See `docs/maintenance/maintaining_binaries.md` and `maintaining_kegg_data.md` in
raven-toolbox for the full procedure. Per-asset upstream sources and checksums are
in [`PROVENANCE-binaries.txt`](PROVENANCE-binaries.txt).

## Licensing

This repo's own text (README, manifest) is provided as-is. **Each asset carries its
upstream license**, included inside the ZIP / recorded in the manifest: BLAST+
(public domain, NCBI), DIAMOND (GPL-3.0-only), HMMER (BSD-3-Clause). KEGG-derived
data is redistributed under permission from KEGG and is subject to KEGG's terms.
