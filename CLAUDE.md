# BirdNET-Go (tphakala/birdnet-go)

## 프로젝트 개요
마이크를 켜두면 창밖에서 들려오는 새소리와 박쥐 소리를 인공지능이 실시간으로 듣고 어떤 종인지 척척 맞춰주는 "자연 소리 실시간 생태 분석기"
숲속이나 정원에서 들리는 신비로운 자연의 소리를 기록하고 계절별, 시간별로 어떤 야생동물이 다녀갔는지 데이터로 축적
자연을 사랑하는 탐조 애호가와 생태 연구자에게 야생의 숨결을 과학적으로 들려주는 감동적인 생태계 모니터링 도구

## 핵심 특징 & 추천 분야
- 실시간새소리인식
- 자연생태소리분석
- 야생동물탐조도구
- 음향AI생태모니터링
- 친환경감성기술

---
*이 문서는 오픈소스 큐레이터(Curator-Agent)에 의해 자동 생성된 가이드 문서입니다.*


---
## 기존 CLAUDE.md 내용

# BirdNET-Go Development Guidelines

## Project Overview

BirdNET-Go: Go implementation of BirdNET for real-time bird sound identification aimed for non serious birders and home users. Open source project for fun.

## Quick Navigation

- **Frontend**: See `frontend/CLAUDE.md` for Svelte 5, TypeScript, UI
- **Backend**: See `internal/CLAUDE.md` for Go standards, testing
- **API v2**: See `internal/api/v2/CLAUDE.md` for endpoints
- **Testing**: See `TESTING.md` for test patterns, testify usage, shared helpers

**IMPORTANT**: Always read the relevant CLAUDE.md file before working on code:

- Working on Go code? Read `internal/CLAUDE.md` first
- Working on frontend? Read `frontend/CLAUDE.md` first
- Working on API v2? Read `internal/api/v2/CLAUDE.md` first
- Writing tests? Read `TESTING.md` first - all tests MUST use testify
- Working with Sentry issues or support dumps? Load the `sentry` skill first

## Universal Rules

### Critical Constraints

- **NEVER expand API v1** - All new endpoints in `internal/api/v2/`
- **Always lint before commit**: `golangci-lint run -v` (Go), `npm run check:all` (Frontend)
- **Branch from updated main**: `git pull origin main && git checkout -b feature-name`
- **No magic numbers/strings** - Use named constants with descriptive names
- **Settings must support hot-reload** - All settings changed via the UI must take effect immediately without requiring a server restart. Use per-request checks (e.g., dynamic middleware) instead of startup-time branching.

### Project Structure

| Path         | Purpose            |
| ------------ | ------------------ |
| `/cmd/`      | Viper CLI commands |
| `/internal/` | Private packages   |
| `/pkg/`      | Public packages    |
| `/frontend/` | Svelte 5 UI        |

## Code Search & Refactoring

**Use ast-grep instead of grep/sed for code operations** - it's more reliable and syntax-aware.

### Search Operations

```bash
# ❌ Avoid grep for code patterns
grep -r "function.*async" src/

# ✅ Use ast-grep - understands syntax
ast-grep --pattern "async function $NAME($$$) { $$$ }" src/

# ❌ Complex grep with regex
grep -r "console\.\(log\|warn\|error\)" src/

# ✅ Structural pattern matching
ast-grep --pattern "console.$METHOD($$$)" src/
```

### Refactoring Operations

```bash
# ❌ Avoid sed for code changes
sed 's/let \([a-zA-Z]*\) =/const \1 =/g' file.js

# ✅ Use ast-grep rewrite - syntax-safe
ast-grep --pattern "let $VAR = $VALUE" --rewrite "const $VAR = $VALUE" src/

# ✅ Complex refactoring example
ast-grep --pattern "export let $PROP" --rewrite "let { $PROP } = \$props()" --lang svelte src/
```

### Why ast-grep?

- **Syntax-aware**: Won't break code structure
- **Language-specific**: Supports TypeScript, Svelte, Go, etc.
- **Reliable**: Matches semantic patterns, not just text
- **Safe refactoring**: Preserves code meaning

**Frontend**: See `frontend/doc/AST-GREP-SETUP.md` for complete ast-grep integration guide.

## Build Commands

| Command               | Purpose                             |
| --------------------- | ----------------------------------- |
| `task`                | Default build (auto-detects target) |
| `task dev_server`     | Development with hot reload         |
| `task frontend-build` | Frontend only                       |
| `task clean`          | Clean artifacts                     |
| `task linux_amd64`    | Cross-platform builds               |

## Pre-Commit Checklist

0. Run preflight quality gate: `/preflight` (or follow `.agents/skills/preflight/SKILL.md`)
1. Run linters: `golangci-lint run -v` / `npm run check:all`
2. Run tests: `go test -race -v` / `npm test`
3. Check open PRs to avoid conflicts
4. Format markdown with prettier
5. Document all exports

## QA Testing Framework

The BirdNET-Go QA system lives in `~/src/birdnet-go-qa/`. Always use it instead of ad-hoc test scripts.

### Key Paths

| Path | Purpose |
|------|---------|
| `~/src/birdnet-go-qa/e2e/tests/` | Playwright E2E test specs |
| `~/src/birdnet-go-qa/configs/` | Test runtime configs (mounted into containers) |
| `~/src/birdnet-go-qa/Taskfile.yml` | Task runner for build/deploy/test workflows |
| `~/src/birdnet-go-qa/Dockerfile.test` | Test container image definition |

### Running E2E Tests

```bash
# Build test image with latest binary
cp ~/src/birdnet-go/bin/birdnet-go ~/src/birdnet-go-qa/birdnet-go
cd ~/src/birdnet-go-qa && podman build -t birdnet-go:test -f Dockerfile.test .

# Deploy test container (dashboard config, port 8085, auth enabled)
podman run -d --name birdnet-go-test --network host \
  -v ~/src/birdnet-go-qa/configs/test-runtime-dashboard:/config \
  birdnet-go:test

# Run specific test suites
cd ~/src/birdnet-go-qa/e2e
npm run test:settings      # Settings CRUD round-trip (15 tests)
npm run test:fuzz          # Settings fuzzer
npm run test:integrity     # Config integrity
npm run test:alerts        # Alert rules
npm run test:eq-gain       # Audio EQ
```

### Config Management Tests

For config hot-reload QA, these are the relevant test files:
- `settings-roundtrip.spec.js` - PATCH/PUT persistence, CSRF, validation
- `settings-fuzzer.spec.js` - Fuzzing settings with random/boundary values
- `config-integrity.spec.js` - Config structure validation
- `hot-reload-comprehensive.sh` - Shell-based hot-reload tests
- `hot-reload-deep.sh` - Deep hot-reload edge cases
- `audio-eq-save.spec.js` - Audio equalizer save round-trip

### Forgejo QA Wiki

Full documentation in the birdnet-go-qa Forgejo wiki: `http://localhost:3000/tphakala/birdnet-go-qa/wiki/`

## PR Review Workflow

Automated code review (CodeRabbit, plus the repo's configured review checks) runs on new PRs automatically and checks for bugs, security issues, and best practices. After pushing fixes, you can request a fresh CodeRabbit pass:

```bash
# Re-request a CodeRabbit review
gh pr comment <PR_NUMBER> --body "@coderabbitai review"

# Or from current branch
gh pr comment $(gh pr view --json number -q .number) --body "@coderabbitai review"
```

### Handling PR Review Comments

When fetching and addressing code review comments from a PR, use the receiving-code-review skill:

```text
/superpowers:receiving-code-review
```

This skill ensures:

- Technical verification before implementing suggestions
- Appropriate pushback on incorrect feedback
- No performative agreement - just fix and move on
- Clarification of unclear items before partial implementation
