# github/github-mcp-server — Evaluation Findings

**Prepared for:** Friday architecture review — whether to standardize the team's GitHub integrations on the official GitHub MCP server
**Evaluated repository:** `github/github-mcp-server` ("GitHub's official MCP Server")
**Research date:** 2026-10-20

## 1. Latest release

- **Release name:** GitHub MCP Server 2.0.2
- **Release tag:** `v2.0.2` (annotated tag object `503622d154b505a9cebff296a13bc6c9259ea993`)
- **Published:** 2026-10-08T11:33:15Z
- **Release URL:** https://github.com/github/github-mcp-server/releases/tag/v2.0.2
- **Tag points to commit:** `eb47a99ddb866ca2b8a162920e6bda9521f33ebb`
  - **Headline message:** `fix(inventory): retain typed tool header validation (#3454)`
  - Body summary: Preserve advertised parameter header bindings in the SDK registration schema so Streamable HTTP rejects header/body mismatches before receiving middleware. Keep full body validation and defaults behind tool guards, with unchanged advertised schemas and OAuth finalization.
  - Author: @SamMorrowDrums (merge commit by web-flow, co-authored by Copilot App), 2026-10-08T11:25:50Z
  - Commit stats: +490 / −3 lines (493 total changed)
  - **Files touched: 6**
    1. `docs/streamable-http.md` (modified, +16)
    2. `internal/ghmcp/oauth_typed_test.go` (modified, +141)
    3. `pkg/inventory/header_params_test.go` (added, +266)
    4. `pkg/inventory/server_tool.go` (modified, +3/−2)
    5. `pkg/inventory/typed_input_test.go` (modified, +14)
    6. `pkg/inventory/typed_schema.go` (modified, +50/−1)

## 2. What the project is / what it needs to run (README at the v2.0.2 commit)

- **What it is:** The GitHub MCP Server connects AI tools (agents, assistants, chatbots) directly to GitHub's platform over the Model Context Protocol. Use cases: repository management (browse/query code, analyze commits), issue & PR automation, CI/CD & workflow intelligence (monitor Actions runs, analyze failures, manage releases), code analysis (code scanning, Dependabot alerts), and team collaboration (discussions, notifications, teams).
- **Two ways to run it:**
  - **Remote server** (hosted by GitHub): `https://api.githubcopilot.com/mcp/` — easiest path; requires an MCP host with remote-server support (VS Code 1.101+, Claude Desktop, Cursor, Windsurf, etc.) plus any applicable policies; auth via OAuth or PAT. An "Insiders" channel (`/mcp/insiders` or `X-MCP-Insiders: true`) offers early-access features. Only VS Code currently has full remote OAuth support; most other hosts work remote+PAT.
  - **Local server:** Docker image `ghcr.io/github/github-mcp-server` (recommended and most common), pre-built release binary, or build from source.
- **Prerequisites:** a GitHub Personal Access Token (for local/PAT flows); Docker (for Docker installs) or a Go toolchain to build from source; a supported MCP host app. Installation guides cover 15+ hosts (VS Code, JetBrains, Visual Studio, Claude Desktop/Code, Cursor, Codex, Windsurf, Zed, OpenCode, Gemini CLI, Cline, Roo Code, Antigravity, Rovo Dev CLI, Xcode, Eclipse).
- **Note:** the legacy npm package `@modelcontextprotocol/server-github` is no longer supported (since April 2025). Configuration supports toolsets, read-only mode, and lockdown mode.

## 3. Language / runtime (manifest at the release commit)

- **Language: Go.** `go.mod` at the v2.0.2 commit declares module `github.com/github/github-mcp-server/v2`, `go 1.26.8`.
- Key direct dependencies: `modelcontextprotocol/go-sdk v1.8.0` (official MCP Go SDK), `google/go-github/v92` (REST), `shurcooL/githubv4` (GraphQL), `go-chi/chi/v5`, `spf13/cobra` + `spf13/pflag` + `spf13/viper` (CLI/config), `golang.org/x/oauth2`, `google/jsonschema-go`.
- Docs note Go 1.24+ for building from source. Go library consumers must import via the `/v2` module path (`github.com/github/github-mcp-server/v2/pkg/...`); the `v2.0.0` tag predates the module-path correction and cannot be used as a Go module.

## 4. Changes from the previous release (v2.0.1 → v2.0.2)

- **v2.0.1** — published 2026-10-07T08:13:24Z, one commit: `fix(go)!: use the v2 module path for Go consumers (#3445)` (commit `55edd58d5e1127fe7ea3c036bc14dca66f7e41c4`). **BREAKING for Go importers** — published the proper `/v2` module path; migrated internal imports, build linker symbols, installation docs, and generated license attribution.
- **v2.0.2** — published 2026-10-08T11:33:15Z, one commit: `fix(inventory): retain typed tool header validation (#3454)` (commit `eb47a99ddb866ca2b8a162920e6bda9521f33ebb`, +490/−3 across 6 files). Restores typed-tool HTTP header validation (advertised parameter header bindings in the SDK registration schema) so Streamable HTTP rejects header/body mismatches before receiving middleware.
- **Net:** two consecutive bugfix releases, one day apart, one commit each — v2.0.2 restores header-validation behavior that the v2.0.1 module-path migration had dropped. Full changelog: https://github.com/github/github-mcp-server/compare/v2.0.1...v2.0.2
- Both releases ship the same GoReleaser platform binaries (Darwin arm64/x86_64, Linux arm64/x86_64/i386 + checksums; e.g. `github-mcp-server_Darwin_arm64.tar.gz` ≈ 8.85 MB) plus a GHCR container image.

## 5. Health snapshot (as of 2026-10-20)

- **Open issues: 192**
- **Open pull requests: 187**
- Activity signals: back-to-back releases Oct 7–8; default-branch commits continuing through mid-October; ~29 active workflows including CI, golangci-lint, CodeQL (+Advanced), Docker publish, GoReleaser, Dependabot, Copilot code review, and AI moderation — a mature, high-velocity project.

## 6. CI build artifacts

- **Release run behind v2.0.2** — "GoReleaser Release" run #97 (`goreleaser.yml`, run id 37770029299, head SHA `eb47a99ddb866ca2b8a162920e6bda9521f33ebb`, event: push of tag `v2.0.2`, completed successfully 2026-10-08): **0 workflow-run artifacts.** GoReleaser uploads directly to Release assets instead, so binaries appear on the release page rather than as Actions artifacts.
- **Most recent completed run on the default branch** — "Docker" run #7328 (run id 39351138308, head SHA `e3809fe91c7546633241006bf0b91f23b2595e28`, completed successfully 2026-10-20): **1 artifact:**
  - **Name:** `github~github-mcp-server~Q18PFQ.dockerbuild`
  - **Size:** 124,923 bytes (~122 KiB)
  - Created 2026-10-20T01:04:17Z; expires 2027-01-18T01:02:03Z; not expired
  - Digest: sha256:28b040ee4c3fd3185b70a870acaaac6aa64416f14a3d9856253fab74a9d40fad
  - Download URL: https://api.github.com/repos/github/github-mcp-server/actions/artifacts/12233462409/zip
  - Caveat: `dockerbuild` artifacts are build/debug logs, not deployable binaries — the deployable outputs are the Release assets and the `ghcr.io/github/github-mcp-server` image.
- **Build-and-test run #11664** (`go.yml`, run id 38808623916, same head SHA, success): 0 artifacts.

## 7. Takeaways for the architecture review

- Official, first-party project from GitHub with an active release cadence, very broad MCP-host support, and mature CI/CD (GoReleaser multi-platform binaries, GHCR image, CodeQL, Dependabot).
- Recommended consumption for our team: the remote hosted server (OAuth) where the host supports it, otherwise the official Docker image; avoid the deprecated npm package.
- Self-hosting from source means a Go 1.26 toolchain and the `/v2` module path.