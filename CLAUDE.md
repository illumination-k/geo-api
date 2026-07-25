# AGENTS Guideline

This repository is pre-alpha and under active development. The API is not stable and may change without a major version bump, so backwards compatibility is not guaranteed at this stage.
So developers of this repository DO NOT need to worry about breaking changes or maintaining backwards compatibility. We prefer to iterate quickly and make breaking changes as needed, rather than trying to maintain backwards compatibility.

## Policy

Follow the YANGI, SOLID, DRY, and KISS principles in all code and documentation. Prioritize simplicity, readability, and maintainability over cleverness or optimization. Avoid premature optimization and over-engineering. Strive for clear and concise code that is easy to understand and modify.

## Development Process

Run `mise install` first to install the toolchain and project tools.

At the end of a session, run `mise run ci` and make sure it passes. Use the narrower tasks while iterating:

```bash
mise run fmt      # Format
mise run lint     # Lint and policy checks
mise run test     # Tests
mise run ci       # Full required verification
```

## Commands

Run `mise install` first to install all tools.

```bash
mise run ci    # Run all ci:* tasks
mise run fmt   # Run all fmt:* tasks
mise run lint  # Run all lint:* tasks
mise run test  # Run all test:* tasks
```

## Tools

All tools are managed by mise. Run `mise install` to install them.

| Tool           | Purpose                                 |
| -------------- | --------------------------------------- |
| uv             | Python package manager                  |
| dprint         | Code formatter                          |
| prek           | Pre-commit hook runner                  |
| shfmt          | Shell script formatter                  |
| actionlint     | GitHub Actions linter                   |
| zizmor         | GitHub Actions security linter          |
| shellcheck     | Shell script linter                     |
| ghalint        | GitHub Actions linter                   |
| pinact         | Pin GitHub Actions versions to SHAs     |
| rust           | Rust toolchain                          |
| cargo-binstall |                                         |
| cargo-nextest  | Fast Rust test runner                   |
| cargo-deny     | Dependency license and advisory checker |
| cargo-audit    | Security advisory checker for Rust      |
| cargo-mutants  |                                         |
| cargo-llvm-cov |                                         |
| python         | Python runtime                          |

## Purpose

NCBI GEO (Gene Expression Omnibus) にアクセスするための Rust ライブラリ。
GEO のアクセッション（GSE / GSM / GPL / GDS）を起点に、
E-utilities を使った検索・メタデータ取得、SOFT / MINiML フォーマットのパースを提供する。
まず Rust 実装を作り、そのうえで pyo3 / maturin による Python binding を追加して、
Python からも同じ機能を使えるようにするのが目的。

## Architecture

### Workspace 構成

Rust workspace を中心に、Python binding を上に載せる構成：

- `crates/geo-api/` — コアの Rust ライブラリ。GEO へのアクセスロジックはすべてここに集約する
- Python binding — pyo3 + maturin で後から追加する（`crates/geo-api-py/` などを想定）。
  binding は薄いラッパーに留め、ドメインロジックは `crates/geo-api/` 側に置く

### コアライブラリ (`crates/geo-api/`)

想定するモジュール構成：

- `client` — `reqwest` ベースの HTTP クライアント。NCBI E-utilities のエンドポイントを叩く
- `accession` — GSE (Series) / GSM (Sample) / GPL (Platform) / GDS (Dataset) の型とパース
- `eutils` — `esearch` / `esummary` / `efetch` のラッパー（db=gds など）
- `parser` — SOFT / MINiML フォーマットのパーサ
- `model` — Series / Sample / Platform / Dataset のデータ構造

### Python binding の方針

テンプレートで生成される `pyproject.toml` は純 Python パッケージ用の足場のため、
binding を実装する際に maturin ビルドバックエンド（pyo3）に置き換える。
API は Rust 側と対応させ、Python 側で薄いラッパー・型変換のみを持つ。

## Coding guidelines

- NCBI のレート制限に配慮する（APIキーなしで 3 req/s、ありで 10 req/s）。
  クライアント側でレート制御・リトライを持たせる
- 非同期は `tokio` + `reqwest` を基本とする
- エラーは `thiserror` で型付けし、ネットワーク・パース・アクセッション不正を区別する
- ネットワークに依存するテストはモック（レスポンス固定）で書き、実 API 叩きは最小限にする
