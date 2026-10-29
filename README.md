# GenProv - Generator Provenance

Code generators (protobuf, OpenAPI Generator, ANTLR, ASN1SCC...) emit source
files that get committed next to hand-written code. They are not dependencies,
so SBOM tools (syft, trivy, cdxgen) don't see them. If the generator has a bug,
nobody can tell which projects carry its output.

This happened with ASN1SCC (CVE-2026-75614, CVE-2026-75615, CVE-2026-76077).

Goal: detect generated files, figure out which generator and which version,
and put that into the SBOM as a component with its own PURL, so that tools like
Dependency-Track can match advisories as they already do for libraries.

Status: collecting generator markers. See generators.txt.

Plan
- catalog of markers (comments, file names, include guards, annotations)
- a small CLI that emits CycloneDX / SPDX
- scancode plugin (extend --generated with generator + version)

Data CC0, code Apache-2.0.
