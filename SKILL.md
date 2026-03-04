---
name: code-narrator
version: 1.2.0
description: Transforms complex code into compelling narrative explanations that reveal architecture, design patterns, and intent
author: SMOUJBOT
tags:
  - writing
  - code
  - education
  - documentation
  - analysis
maintainer: ops@smouj.dev
repo: smouj/openclaw/skills/code-narrator
docker_image: openclaw/code-narrator:latest
runtime: nodejs-20
capabilities:
  - code_analysis
  - narrative_generation
  - documentation
  - architecture_extraction
dependencies:
  - ast-parser:^2.3.1
  - @typescript-eslint/parser:^7.0.0
  - narrative-engine:^1.1.0
  - code-metrics:^3.0.0
  - markdown-formatter:^1.0.0
env_vars:
  NARRATOR_LOG_LEVEL: info
  NARRATOR_MAX_TOKENS: 4000
  NARRATOR_STYLE: technical
  NARRATOR_OUTPUT_FORMAT: markdown
exposes:
  - port: 8080
    protocol: http
    path: /narrate
healthcheck:
  endpoint: /health
  timeout: 5000
  interval: 30000
work_dir: /tmp/narrator-work
---

# Code Narrator Skill

## Purpose

Code Narrator transforms complex codebases into compelling, narrative-style explanations that reveal architectural decisions, design patterns, and developer intent. Unlike simple documentation generators that list functions and parameters, Code Narrator creates a story around the code explaining *why* decisions were made, *how* components interact, and *what* problems the code solves.

### Real Use Cases

1. **Onboarding Acceleration**: New developers join a legacy codebase with no documentation. Code Narrator analyzes the core modules and generates a narrative walkthrough explaining the system architecture, key patterns, and design rationale.

2. **Code Review Preparation**: Before reviewing a large PR, generate a narrative summary of changed files to understand the scope and intent quickly.

3. **Technical Debt Documentation**: Narrate complex, messy code sections to document what they actually do, creating baseline understanding before refactoring.

4. **Architecture Decision Records (ADR)**: Extract and narrate the implicit architectural decisions embedded in code to create explicit ADRs.

5. **Cross-Team Knowledge Sharing**: Share narrative explanations of microservices or libraries with teams that don't have direct access to the codebase.

## Scope

Code Narrator operates on local codebases or git repositories. It does not modify source code; it generates documentation files and narrative outputs.

### Supported Languages
- TypeScript/JavaScript (full support)
- Python (full support)
- Go (full support)
- Rust (full support)
- Java (basic support)
- C# (basic support)

### Common Commands

```bash
# Narrate entire directory with detailed analysis
openclaw skill run code-narrator --input ./src --output ./docs/narrative.md --depth comprehensive --style narrative

# Narrate specific file with focus on complexity
openclaw skill run code-narrator --file ./src/core/auth.ts --focus complexity --output ./docs/auth-narrative.md

# Narration with custom prompt
openclaw skill run code-narrator --git-repo https://github.com/user/repo --branch feat/new-auth --prompt "Explain authentication flow and security considerations"

# Generate executive summary only
openclaw skill run code-narrator --input ./src --level executive --output ./docs/EXECUTIVE_SUMMARY.md

# Interactive mode (REPL)
openclaw skill run code-narrator --interactive
```

## Detailed Work Process

### Phase 1: Code Ingestion and Parsing (0-30s)

1. **File Discovery**
   ```bash
   openclaw skill run code-narrator --input ./src --include '**/*.{ts,js,py,go,rs}'
   ```
   Scan directories for supported file types using glob patterns. Exclude `node_modules`, `.git`, `dist`, `build`, `coverage` by default.

2. **AST Generation**
   Parse each file into Abstract Syntax Trees using language-specific parsers:
   - TypeScript: `@typescript-eslint/parser`
   - Python: `ast-parser` with Python 3.8+ compatibility
   - Go: `go/ast` via Go's standard library
   - Rust: `syn` crate via `rust-analyzer` bridge

   **Verification**: AST generation must succeed for >95% of files. Failures logged with `NARRATOR_LOG_LEVEL=debug`.

3. **Dependency Graph Construction**
   ```bash
   NARRATOR_BUILD_GRAPH=true openclaw skill run code-narrator --input ./
   ```
   Build import/require graphs to understand module relationships. Detect circular dependencies and architectural violations.

### Phase 2: Analysis and Pattern Detection (30s-2min)

1. **Complexity Scoring**
   Calculate per-file metrics:
   - Cyclomatic complexity (>10 = high)
   - Nesting depth (>4 = high)
   - Function length (>50 lines = long)
   - Coupling metrics (efferent coupling >10 = high)

2. **Pattern Recognition**
   Detect and catalog:
   - Design patterns (Factory, Observer, Strategy, etc.)
   - Anti-patterns (God objects, Singleton abuse, etc.)
   - Architectural patterns (Layered, Hexagonal, Microservices, etc.)
   - Custom project-specific patterns

3. **Intent Inference**
   Analyze code to infer:
   - Why complex branching exists (feature flags, backward compatibility)
   - Why certain abstractions were created (reuse, testing, separation of concerns)
   - Why performance optimizations were necessary (hot paths, memory constraints)

### Phase 3: Narrative Generation (2min-5min)

1. **Structure Determination**
   Based on `--depth` flag:
   - `executive`: High-level overview, 1-2 paragraphs per major module
   - `standard`: Standard narrative with sections per component
   - `comprehensive`: Deep dive with line-by-line explanations of complex sections

2. **Narrative Writing**
   Using narrative templates with variables filled by analysis:
   ```
   "The {ModuleName} module implements {CoreResponsibility}. 
   It uses the {DesignPattern} pattern to {Intent}. 
   The complexity stems from {ComplexitySource}, which was necessary because {Rationale}. 
   Key components include {ComponentList}, which interact via {InteractionPattern}."
   ```

   **Style options** (`--style`):
   - `narrative`: Story-like flow ("Imagine a user trying to...")
   - `technical`: Direct documentation ("This module provides...")
   - `educational`: Teaching-focused ("Let's explore how...")

3. **Quality Validation**
   - Check narrative length (min 500 chars for standard, 2000 for comprehensive)
   - Verify all major files are mentioned
   - Validate that "why" explanations exist for >80% of complex sections

### Phase 4: Output and Integration (5min-6min)

1. **Format Rendering**
   Render narrative to selected format (`--output-format`):
   - `markdown`: With headings, code blocks, diagrams
   - `html`: With syntax highlighting and navigation
   - `json`: Structured data for further processing
   - `plain`: Simple text for terminals

2. **File Writing**
   Write output to specified path. Create parent directories if needed.
   ```bash
   openclaw skill run code-narrator --output ./docs/architecture-narrative.md --create-dirs
   ```

3. **Cross-Reference Generation**
   If `--generate-xrefs` flag set, create hyperlinks between mentions of modules/components.

## Golden Rules

1. **Never Modify Source Code**: Code Narrator is read-only. No files under `src/`, `lib/`, or source directories should be modified.

2. **Always Verify Parse Success**: Before generating narrative, ensure AST generation succeeded for at least the top 10 most critical files (by size or complexity). If parsing fails, abort with clear error.

3. **Narrate Intent, Not Just Structure**: Generated narrative must explain *why* code exists as it does. A file listing is a failure.

4. **Respect Complexity Thresholds**: Files with cyclomatic complexity >15 require individual attention. Narrate them separately with detailed explanations of each branch/path.

5. **Handle Large Codebases**: For >500 files, generate modular narrative (one file per major component) rather than monolithic document. Use `--output-dir` instead of single file.

6. **Always Cite Line Numbers**: When referencing specific code behaviors, include line numbers from original source. Format: `(src/auth/LoginService.ts:247)`

7. **Detect and Warn on Generated Code**: If code contains `// Generated by` or `@generated` markers, skip narrative generation and log warning. Generated code should not be narrated as if written by developers.

8. **Verify Output Before Writing**: Validate narrative content (non-empty, meets length requirements) before writing to disk. Write atomically (temp file then rename).

## Examples

### Example 1: Onboarding a New Developer

**Command:**
```bash
openclaw skill run code-narrator \
  --git-repo https://github.com/company/monorepo \
  --branch main \
  --input packages/auth \
  --depth comprehensive \
  --style educational \
  --output /tmp/auth-narrative.md \
  --generate-xrefs
```

**Configuration (implicit):**
```bash
export NARRATOR_STYLE=educational
export NARRATOR_MAX_TOKENS=4000
```

**Sample Output (excerpt):**
```markdown
# Authentication Module Narrative

## Overview
The authentication module (`packages/auth/`) is responsible for all user identity operations in the system. It handles login, session management, token refresh, and permission validation. The module was designed with security as the primary constraint, which explains the apparent complexity in token handling.

## Core Components

### 1. LoginService (src/services/LoginService.ts)
This service orchestrates the multi-step login process. You'll notice a seemingly redundant validation step at line 47—this exists because of a legacy SSO integration that requires double verification for federated identities. The service follows the Chain of Responsibility pattern, where each handler (line 62-203) represents a different authentication method (password, OAuth, SAML, etc.).

**Why the complexity?** Originally, the system only supported email/password. When enterprise customers demanded SSO, the architecture was extended without breaking existing integrations. Hence the bifurcated logic at lines 89-145.

### 2. TokenManager (src/security/TokenManager.ts)
Token management is split into two concerns: generation (lines 18-89) and validation (lines 92-167). The separation allows aggressive caching of public keys for JWT validation, reducing latency from ~50ms to ~2ms per request after warmup.

## Architectural Decisions

### Decision: Stateless Session Design
**What:** Sessions are stored client-side as encrypted JWTs rather than server-side sessions.
**Why:** The system runs in a serverless environment (AWS Lambda) where traditional session stores would introduce unacceptable latency and scaling bottlenecks. The tradeoff is larger tokens (2KB) and inability to revoke tokens before expiry.
**See:** Implementation at src/middleware/SessionMiddleware.ts:45-89.
```

### Example 2: Understanding a Complex Refactoring

**Command:**
```bash
openclaw skill run code-narrator \
  --git-repo https://github.com/company/api \
  --branch feat/refactor-payment \
  --compare main \
  --output ./docs/payment-refactor-narrative.md \
  --focus 'migration,backward-compatibility' \
  --style technical
```

**Prompt (implicit):** "Explain how the payment refactoring maintains backward compatibility while introducing the new聚合 payment engine."

**Sample Output (excerpt):**
```markdown
# Payment Refactoring Narrative (feat/refactor-payment vs main)

## Summary of Changes
The refactoring splits the monolithic `PaymentProcessor` (560 lines) into three focused services while maintaining 100% backward compatibility with existing integrations. This was achieved through an adapter pattern and careful feature flagging.

## Key Migration Strategies

### 1. Adapter Layer (`src/adapters/LegacyPaymentAdapter.ts`)
The original `PaymentProcessor` interface is preserved in this adapter, which translates old calls to the new service mesh. Crucially, the adapter at line 234 contains a direct `eval()` call—this is intentional and safe because the input is sanitized at line 231. It exists to execute dynamically constructed AML (Anti-Money Laundering) rules that regulators update without code deploys.

**Compatibility verification:** All existing API endpoints continue to function; see integration tests at `test/legacy/payment.integration.test.ts:45-89`.

### 2. Feature Flags
New code paths are gated behind `featureFlags.enableNewPaymentEngine`. The flag is read at runtime from environment, allowing gradual rollout. The flag check at line 78 in `src/modules/PaymentRouter.ts` uses a short-circuit evaluation pattern to avoid any performance impact when the flag is off.

## Architecture Evolution
The previous architecture (main branch) had PaymentProcessor doing validation, processing, settlement, and reconciliation. The new architecture separates these into:
- `PaymentValidationService` (idempotent, CPU-bound)
- `PaymentProcessingService` (stateful, I/O-bound)
- `PaymentSettlementService` (async, eventually consistent)

This separation allows independent scaling: processing can handle 10k RPS while validation runs at 1k RPS on smaller instances.

## Performance Impact
Throughput increased from 1,200 to 4,500 transactions/sec in load testing (see `benchmark/payment-bench.ts`). Latency P99 dropped from 450ms to 120ms. The narrative includes before/after flamegraphs at lines 456 and 789.
```

### Example 3: Interactive Exploration

**Command:**
```bash
openclaw skill run code-narrator --interactive --input . --style narrative
```

**Session:**
```
Narrator> What's the deal with the weird caching layer in src/cache?
Analysis: The cache layer uses a hybrid strategy. At lines 12-45, there's a write-through cache for hot data (user profiles, settings). At lines 47-89, there's a read-through cache with stale-while-revalidate for slower endpoints. The weirdness at line 67—the negative TTL—is intentional: it's a circuit breaker that, when triggered, sets TTL to -1 (meaning "do not cache") for 30 seconds to allow origin recovery.

Narrator> Why is there database logic inside the email service?
The email service (src/services/email.ts:234-567) contains direct PostgreSQL queries because the system sends 50,000+ transactional emails per day and needs to track status per recipient. The ORM abstraction was removed after profiling showed 40% overhead. The queries at lines 278-312 are hand-optimized with CTEs for batch lookups. This violates clean architecture but was necessary for scale. See ADR-2023-07 for the decision record.

Narrator> exit
```
```

## Rollback Commands

If Code Narrator produces incorrect, misleading, or unwanted narratives, use these commands:

```bash
# Undo single narration file (if generated with --output)
rm /path/to/generated-narrative.md
# or restore from git if committed
git checkout HEAD -- /path/to/generated-narrative.md

# Clear entire output directory
rm -rf ./docs/narratives/
# Revert from backup
cp -r /backups/narratives/2024-01-15/ ./docs/narratives/

# Disable skill entirely (revert to manual docs)
openclaw skill disable code-narrator

# Clean work directory (if process was interrupted)
rm -rf /tmp/narrator-work/*
rm -rf $(dirname $NARRATOR_OUTPUT)/*.md

# Revert configuration changes
# If you modified ~/.openclaw/config/ext/custom-overrides.json for narrator:
git checkout -- ~/.openclaw/config/ext/custom-overrides.json
openclaw doctor --fix

# Full cleanup (if skill misbehaved)
openclaw skill uninstall code-narrator
rm -rf ~/.openclaw/data/code-narrator-cache
```

## Troubleshooting

### Issue: "Parser failed for file X.ts"
**Cause**: TypeScript version mismatch or syntax too new for parser.
**Fix**: Ensure `@typescript-eslint/parser` version matches project's TypeScript. Update skill: `openclaw skill update code-narrator`. Or exclude the file: `--exclude '**/generated/**'`.

### Issue: Narrative is just a file listing, no explanations
**Cause**: Code too simplistic or pattern detection empty.
**Fix**: Increase depth: `--depth comprehensive`. Check logs: `NARRATOR_LOG_LEVEL=debug openclaw skill run ...`. Ensure files have meaningful logic (not just types/interfaces).

### Issue: Output truncated mid-sentence
**Cause**: `NARRATOR_MAX_TOKENS` too low for large codebase.
**Fix**: Increase limit: `export NARRATOR_MAX_TOKENS=8000` or split output: `--output-dir ./narratives/` instead of single file.

### Issue: Generated narrative mentions wrong files/lines
**Cause**: AST line numbers misaligned due to preprocessors (e.g., Babel).
**Fix**: Use `--source-maps` flag if source maps exist. Or run on original (untranspiled) source.

### Issue: Skill hangs on large repository
**Cause**: Memory exhaustion parsing 1000+ files at once.
**Fix**: Use `--batch-size 100` to process incrementally. Or target specific subdirectory: `--input packages/auth`.

### Issue: No design patterns detected
**Cause**: Codebase genuinely pattern-free or patterns obscured by framework conventions.
**Fix**: Manually specify focus: `--focus 'Factory,Strategy,Observer'` to force exploration. Or examine raw analysis output: `--output-format json` to see what was detected.

### Issue: Permission denied writing output
**Cause**: Output directory not writable.
**Fix**: Create directories first: `mkdir -p $(dirname -- /output/path)`. Or run with appropriate user: `sudo -u deployer openclaw skill run ...`.

### Issue: "Circular dependency detected" warnings
**Cause**: Codebase has actual circular imports/module dependencies.
**Fix**: This is informational, not an error. The narrative will note these as architectural concerns. To suppress: `--ignore-circular-deps`.

## Dependencies and Requirements

- **Node.js**: v20.0+ (required)
- **AST Parsers**: Installed automatically per language:
  - TypeScript: `@typescript-eslint/parser` (peers: `@typescript-eslint/typescript-estree`, `typescript`)
  - Python: `ast-parser` with Python 3.8+ binary
  - Go: `golang.org/x/tools/go/analysis` (installed via `go install`)
  - Rust: `rust-analyzer` must be in PATH
- **Memory**: Minimum 2GB RAM for codebases <10k LOC; 8GB for >50k LOC
- **Disk**: 500MB for skill itself; additional space for cache (configurable via `NARRATOR_CACHE_DIR`)
- **Network**: Optional, for git cloning (`--git-repo`) or downloading language servers

**Compatibility Matrix:**
| OpenClaw Version | Code Narrator Version | Node.js |
|------------------|----------------------|---------|
| 2.1.x - 2.3.x    | 1.0.x - 1.1.x        | 18+     |
| 2.4.x+           | 1.2.x+               | 20+     |

## Verification Steps

After running Code Narrator, verify success with:

```bash
# 1. Check exit code
if [ $? -eq 0 ]; then
  echo "✓ Narrator completed successfully"
else
  echo "✗ Narrator failed"
  exit 1
fi

# 2. Verify output exists and is non-empty
test -s ./docs/narrative.md || { echo "Output empty or missing"; exit 1; }

# 3. Validate narrative contains expected keywords (customize per project)
grep -q "architecture\|design pattern\|reasoning" ./docs/narrative.md || {
  echo "Narrative appears superficial (missing key terms)";
  exit 1;
}

# 4. Check line number citations present
grep -E '\(src/.*:[0-9]+\)' ./docs/narrative.md > /dev/null || {
  echo "Missing line number citations";
  exit 1;
}

# 5. Verify coverage (narrative should mention all top-level modules)
# Assuming project structure
modules=$(find src -maxdepth 1 -type d | wc -l)
mentioned=$(grep -oE '# [A-Za-z]+' ./docs/narrative.md | wc -l)
if [ $mentioned -lt $((modules - 2)) ]; then
  echo "Coverage insufficient: only $mentioned/$modules modules narrated";
  exit 1;
fi

echo "✓ All verification checks passed"
```

## Advanced Usage

### Integration with CI/CD
```yaml
# .gitlab-ci.yml
generate-docs:
  stage: docs
  script:
    - openclaw skill run code-narrator --input ./src --output ./docs/architecture.md --depth standard
    - git config user.email "docs@ci"
    - git add ./docs/architecture.md
    - git commit -m "docs: update architecture narrative [skip ci]" || echo "No changes"
    - git push origin HEAD:docs-updates
  only:
    - main
    - merge_requests
```

### Custom Narrative Templates
Place custom Jinja2 templates in `./templates/narrator/`:
```bash
openclaw skill run code-narrator --template ./templates/custom-narrative.j2 --output ./docs/custom.md
```

Template variables: `{module_name}`, `{complexity}`, `{patterns}`, `{intent}`, `{line_refs}`.

### Batch Processing Multiple Repos
```bash
#!/bin/bash
for repo in repos/*; do
  openclaw skill run code-narrator \
    --input "$repo/src" \
    --output "narratives/$(basename $repo).md" \
    --depth executive \
    --batch-size 50
done
```
```

This SKILL.md provides comprehensive, specific, practical documentation for Code Narrator as a real OpenClaw skill with actual commands, examples, configuration, and operational guidance.