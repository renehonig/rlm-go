# Security Review – rlm-go

Date: 2026-01-08

## Scope and Methodology
- Reviewed source under `cmd/`, `pkg/`, `examples/`, install script, and Go module manifests (`go.mod`, `go.sum`).
- Ran `go test ./...` to establish a baseline; sandbox container tests failed locally because a container runtime (Podman/Docker) is not available in the environment.
- Looked for unexpected network calls, telemetry, credential collection, or obfuscated logic.

## Findings
1. **Malicious code / behavior**: No malicious or obfuscated code observed. Functionality aligns with documented purpose (LLM orchestration, Yaegi-based execution, optional sandboxing).
2. **Execution model**: By default, LLM-generated code is executed in-process via Yaegi. This is expected but means untrusted code should only be run with the sandbox backend enabled to prevent host access.
3. **Dependencies**: Minimal, well-known dependencies (`github.com/google/uuid`, `github.com/traefik/yaegi`, `golang.org/x/term`, `golang.org/x/sys`). No suspicious or typosquatted packages detected.
4. **Supply chain**: `install.sh` downloads release binaries directly from GitHub without checksum/signature verification. This is not malicious, but users should prefer `go install` or verify release artifacts to mitigate tampering risk.
5. **Data handling**: No built-in telemetry or credential exfiltration detected. API keys are read from environment variables by user-provided clients; callers should manage secrets securely.

## Recommendations
- Use the sandbox backend (Podman/Docker with network disabled by default) when running untrusted or user-supplied code.
- Consider adding checksum verification to `install.sh` for release binary downloads.
- Continue to keep dependencies up to date and monitor for CVEs.

## Test Baseline
- `go test ./...` — **fails** only for `pkg/sandbox` container tests due to missing container runtime. Other packages pass.
