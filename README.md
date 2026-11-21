# CodexTrace

CodexTrace is a read-only .NET 10 command-line analyzer of local Codex execution traces. It reads Codex JSONL rollouts and, when available, the Codex SQLite state database to recover agent hierarchy, event timelines, tool activity, token usage and evidence gaps. It also understands IDD Factory runs, roles, batches and completion evidence.

The core source and test suite are self-contained. There is no build-time dependency on Intent-Driven-Development or generated IDD plugins. The existing executable and C# namespaces remain `idd-factory-report` and `Idd.Factory.Report` for now, to keep this repository extraction behavior-preserving.

## Prerequisites

- [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0).
- Git (for locating the repository to analyze).
- Local Codex trace files (only needed to analyze real runs; not needed for tests).

## Build and test

```sh
dotnet restore CodexTrace.slnx
dotnet build CodexTrace.slnx --no-restore
dotnet test CodexTrace.slnx --no-build
```

GitHub Actions runs the same solution on Ubuntu, Windows and macOS, and smoke-tests the CLI. All test fixtures are included in `tests/Idd.Factory.Report.Tests/Fixtures/`; tests do not require Codex execution, tokens, secrets, credentials, or an IDD checkout.

## CLI

```sh
dotnet run --project src/Idd.Factory.Report -- --help
dotnet run --project src/Idd.Factory.Report -- latest --repo /path/to/project
dotnet run --project src/Idd.Factory.Report -- list --repo /path/to/project
dotnet run --project src/Idd.Factory.Report -- <thread-id> --repo /path/to/project
dotnet run --project src/Idd.Factory.Report -- <thread-id> --run 2 --repo /path/to/project
dotnet run --project src/Idd.Factory.Report -- latest --repo /path/to/project --codex-home /path/to/.codex --json report.json --markdown report.md --verbose
```

The default output is a console report. `--json` and `--markdown` write additional output files; `--verbose` adds diagnostics and trace-path details. `latest` and `list` currently select recognized IDD Factory runs, while Codex event parsing is reusable independently. See [detailed behavior, wire formats, limitations and metrics](docs/factory-report.md).

The analyzer only reads the input; it does not edit sessions or workspaces. Traces and reports may contain prompts, tool output, local paths or other sensitive information. Restrict access accordingly. An analyzed project's `.idd/` files are optional *input data*, not a source dependency.

## Code organization

- `src/Idd.Factory.Report/Codex/`: Codex JSONL/SQLite readers, normalized events and trace models; no dependency on IDD Factory semantics.
- `src/Idd.Factory.Report/Factory/`, `FactoryBatchTrace.cs` and `FinalVerificationEvidence.cs`: IDD-specific run, role, batch, evidence and verification interpretation built on top of generic events.
- `src/Idd.Factory.Report/Cli/`: command-line handling and console, JSON, Markdown writers.
- `tests/Idd.Factory.Report.Tests/`: deterministic tests of nested and parallel agents, sequential runs, token deduplication, format compatibility, damaged or ambiguous traces, and Factory completion.

The separation is logical rather than multi-assembly. No dynamic adapter system or new platform support was introduced.

## Integration with IDD

[Intent-Driven-Development](https://github.com/DimonSmart/Intent-Driven-Development) consumes this repo as a Git submodule at `agent-trace/`, pinned to a tested commit. Its root solution, LiveTests and diagnostic script refer to files under that directory.

Clone IDD with `git clone --recurse-submodules`, or initialize an existing checkout with `git submodule update --init --recursive`. The IDD CI checks out submodules automatically. Updating CodexTrace does *not* silently update the version pinned by IDD; IDD must explicitly advance and test its submodule pointer.

This project builds and tests without IDD. Changes made here should be validated in CodexTrace CI first, then propagated to IDD's pinned integration. No NuGet/.NET Tool publishing pipeline is configured yet.

## License

MIT. See [LICENSE](LICENSE).
