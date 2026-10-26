# genprov — provenance of generated code in SBOMs

Generated source files and copied generator runtimes end up in repositories
without any package dependency. Dependency-based SBOM tools (Syft, Trivy,
cdxgen) therefore record them as first-party code: no generator, no version,
no link to the specification they came from.

When a generator ships a bug or a CVE in its *output* — not in the tool, in
the code it emits — nobody can tell which applications carry that output.
Two documented cases:

- **OpenAPI Generator < 5.1.0** (CVE-2021-21430): generated Java and Scala
  clients create temporary files with insecure permissions. Clients generated
  once stay vulnerable until regenerated. The generator is usually not a
  runtime dependency of the project, so it never appears in its SBOM.
- **ASN1SCC** (ESA): the compiler emits the encoders *and* copies its runtime
  (`asn1crt.c`, `asn1crt.h`, `asn1crt_encoding_uper.c`) next to them. The
  runtime is compiled into the application with no package record and no
  version.

What exists today: GitHub Linguist and ScanCode's `--generated` flag answer
"is this file generated?" with yes/no, to hide diffs or skip license scans.
Neither says *by what* or *which version*, and no SBOM tool writes the answer
down.

## What this repository is

1. **An open catalog of signatures** (`signatures/`): for each generator, the
   deterministic markers that identify its output — header comments, file
   name patterns, include guards, annotations — with a regex for the version
   where the generator writes one, and the generator's Package URL. Only
   markers that generators actually emit; no heuristics, no guessing.
2. (planned) a small scanner that applies the catalog to a repository and
   emits the result as CycloneDX / SPDX relationships (`GENERATED_FROM`),
   with "version unknown" stated explicitly when the marker carries none.
3. (planned) an integration for an existing scanner (ScanCode), upstream.

Rules for the catalog:

- a signature must be reproducible from the generator's own source or
  templates (link it);
- `version_regex` may be absent: then the generator is identified, the version
  is reported as unknown;
- a copied runtime is a separate entry with `kind: runtime`, matched by file
  name and include guard, not by comment.
