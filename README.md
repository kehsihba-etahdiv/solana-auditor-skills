# Solana Auditor Skills

> The ultimate Claude Code skill for Solana smart contract security auditing — 105 attack vectors with concrete code detection patterns, 6 parallel agents, DeFi protocol checklists, and adversarial reasoning.

Built in the style of [pashov/skills](https://github.com/pashov/skills) (Solidity) but rebuilt from scratch for **Rust/SVM/Solana**. Aggregates knowledge from 10+ open-source audit and development skill repositories.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Architecture

The same proven architecture as pashov/skills — adapted for Solana's account model, CPI trust boundaries, PDA security, and the Anchor/Native Rust/Pinocchio ecosystem.

### 4-Turn Orchestration

1. **Discover** — find all in-scope `.rs` files, resolve reference paths
2. **Prepare** — bundle codebase + attack vectors + judging rules into per-agent files
3. **Spawn** — launch 4–6 agents in parallel (vector scan, adversarial reasoning, protocol analysis)
4. **Report** — merge, deduplicate by root cause, sort by confidence, format

### 105 Attack Vectors (5 reference files)

| File | Vectors | Focus Areas |
| --- | --- | --- |
| [attack-vectors-1](solana-auditor/references/attack-vectors/attack-vectors-1.md) | V1–V25 | Signer checks, ownership, discriminators, account constraints, data matching, reinitialization, writable flags, Token-2022 mint authority, sysvar spoofing, instruction introspection, Ed25519 bypass, validation chains (Cashio $48M), Pump Science state bugs |
| [attack-vectors-2](solana-auditor/references/attack-vectors/attack-vectors-2.md) | V26–V50 | PDA derivation (bumps, sharing, collisions, seeds), CPI safety (arbitrary CPI, signer privilege forwarding, stale data, return values), invoke_signed, Token-2022 (permanent delegate, transfer fee, non-transferable, mint close authority), cross-program reentrancy, security dependency chains |
| [attack-vectors-3](solana-auditor/references/attack-vectors/attack-vectors-3.md) | V51–V75 | Integer overflow/underflow, precision loss (Neodyme $2.6B), rounding direction, first-depositor inflation, fee bypass, dust poisoning, token decimals, state lifecycle (close, realloc), coupled fields, time units, Token-2022 closable/interest, floating-point, lamport denomination |
| [attack-vectors-4](solana-auditor/references/attack-vectors/attack-vectors-4.md) | V76–V100 | Oracle manipulation (staleness, confidence, fake accounts, Mango $115M), staking reward gaming, flash stake, reward dilution, cooldown griefing, vault inflation, compute/heap DoS, signature replay, on-chain randomness, rent in bonding curves (Pump Science), transfer hook validation, upgrade authority |
| [attack-vectors-5](solana-auditor/references/attack-vectors/attack-vectors-5.md) | V101–V105 | CPI ownership reassignment, unsafe deserialization without input length validation, orphan account lifecycle, partial discriminator matching, dynamic Token-2022 account sizing |

### 3 Specialized Agent Types

| Agent | Mode | Model | Approach |
| --- | --- | --- | --- |
| Vector Scan (x4) | Default + Deep | Sonnet | Systematic triage of 25 vectors each against full codebase |
| Adversarial Reasoning | Deep only | Opus | Free-form exploit hunting with Feynman questioning, state inconsistency analysis, invariant hunting |
| Solana Protocol | Deep only | Opus | Domain-specific checklists for lending, AMM, vaults, staking, bridges, governance, proxies, session keys |

### Quality Controls

- **FP Gate:** 3-check filter (concrete path, reachable entry, no existing guard)
- **Confidence Scoring:** Base 100 with deductions for privileged callers (-25), partial paths (-20), self-contained impact (-15), token assumptions (-10), external preconditions (-10)
- **Threshold:** Findings below 75 confidence reported without fix suggestions
- **Framework-aware:** Works with Anchor, native Rust, and Pinocchio

---

## Install & Run

Works with **Claude Code CLI**, the **VS Code Claude extension**, and **Cursor**.

**Claude Code CLI:**

```bash
git clone https://github.com/sanbir/solana-auditor-skills.git && mkdir -p ~/.claude/commands && cp -r solana-auditor-skills/solana-auditor ~/.claude/commands/solana-auditor
```

**Cursor:**

```bash
git clone https://github.com/sanbir/solana-auditor-skills.git && mkdir -p ~/.cursor/skills && cp -r solana-auditor-skills/solana-auditor ~/.cursor/skills/solana-auditor
```

The skill is then invocable as `/solana-auditor`. See the [skill README](solana-auditor/README.md) for usage.

**Update to latest:** `cd` into the cloned repo and run:

```bash
git pull
# Claude Code CLI:
cp -r solana-auditor/ ~/.claude/commands/solana-auditor
# Cursor:
cp -r solana-auditor/ ~/.cursor/skills/solana-auditor
```

---

## Skills

| Skill | Description |
| --- | --- |
| [solana-auditor](solana-auditor/) | 105-vector security audit with 4–6 parallel agents, DeFi protocol checklists, and adversarial reasoning |

---

## What's Included

### 105 Attack Vectors (5 reference files)

Every vector includes: **Detect** (grep-able code patterns), **Vulnerable** (concrete Rust code snippet), **Exploit** (how attacker uses it, with real-world references), **Secure** (correct code pattern). Organized by attack surface:

**Account Validation & Authorization (V1–V25):** Missing signer checks, ownership spoofing, type cosplay, discriminator bypass, reinitialization, `init_if_needed` frontrunning, `has_one` constraint gaps, writable flag abuse, `UncheckedAccount` without manual validation, `remaining_accounts` injection, account revival, sysvar spoofing (Wormhole $320M), instruction introspection, predictable PDA initialization, validation chain bypass (Cashio $48M), Ed25519 signature verification bypass, missing state update (Pump Science H-02).

**PDA, CPI & Cross-Program Security (V26–V50):** Non-canonical bumps, PDA sharing, seed concatenation collisions, arbitrary CPI, signer privilege forwarding, stale data after CPI (missing `reload()`), CPI return values ignored, Token-2022 permanent delegate (silent vault drain), transfer fee accounting mismatch, non-transferable tokens, mint close authority reinitialization bypass, CPI ordering violations, security dependency chains, dangling references after CPI close.

**Arithmetic, Tokens & State Management (V51–V75):** Integer overflow/underflow, division-before-multiplication (Neodyme $2.6B), unsafe `as` casting (narrowing + signed-to-unsigned), rounding direction exploitation, first-depositor vault inflation, saturating math misuse, slippage not enforced, fee bypass on alternate paths, pre/post-fee confusion (Pump Science M-01), token decimals mismatch, coupled field inconsistency, time unit mismatch, Token-2022 transfer fee blocks account close, floating-point in financial logic, lamport/SOL denomination confusion.

**Oracle, DeFi & Platform-Level (V76–V100):** Stale oracle price, confidence interval, fake oracle account, on-chain spot price manipulation (Mango Markets $115M), staking reward index bugs, flash stake/unstake, reward dilution via direct transfer, cooldown griefing, self-liquidation profit, vault share inflation, compute budget DoS, heap exhaustion (32KB), signature replay without nonce, on-chain randomness manipulation, rent in bonding curves (Pump Science M-02), transfer hook validation, `.unwrap()` panics, upgrade authority, interest during pause.

**Additional Vectors (V101–V105):** CPI ownership reassignment, unsafe deserialization without input length validation, orphan account from parent-child lifecycle, partial discriminator matching, dynamic Token-2022 account sizing.

### Protocol Checklists (74 items across 8 domains)

| Domain | Items | Key Checks |
| --- | --- | --- |
| Lending/Borrowing | 14 | Health factor includes accrued interest, liquidation incentive covers gas, self-liquidation not profitable, bad debt socialization |
| AMM/DEX | 10 | Slippage from calldata not on-chain, deadline enforced, multi-hop protection, LP value not from raw balance |
| Vault/Token Accounting | 10 | First-depositor mitigated, rounding direction correct, round-trip not profitable, share price not manipulable |
| Staking/Rewards | 10 | Reward accumulator updated before balance change, no flash stake capture, precision doesn't zero small stakers |
| Bridge/Cross-Chain | 9 | Message replay protection, source validation, rate limits, decimal conversion, supply invariant |
| Governance | 6 | Vote weight from past slot, timelock, quorum, no double-voting via transfer |
| Proxy/Upgradeable | 8 | Multi-sig upgrade authority, timelock, storage append-only, verifiable build |
| Session Keys/AA | 7 | Bounded permissions, revocable, replay protection, no self-escalation |

---

## Attributions

This skill aggregates knowledge from the following open-source repositories. We are grateful to all contributors.

### Architecture Inspiration

| Repository | Author | Contribution |
| --- | --- | --- |
| [pashov/skills](https://github.com/pashov/skills) | Pashov Audit Group | Parallelized agent orchestration pattern, FP gate, confidence scoring, vector-scan and adversarial-reasoning agent design, report formatting — adapted from Solidity to Solana |

### Solana Security Knowledge

| Repository | Author | Contribution |
| --- | --- | --- |
| [ciphernova-skills/safe-solana-builder](https://github.com/ciphernova-skills/safe-solana-builder) | CipherNova / Frank Castle | 20 comprehensive security rules (account validation, PDA security, CPI safety, arithmetic, Token-2022, oracle, fees, state management, clock/timing), Anchor-specific and native Rust patterns, security checklist methodology |
| [trailofbits/building-secure-contracts](https://github.com/trailofbits/building-secure-contracts) | Trail of Bits | 6 critical Solana vulnerability patterns (arbitrary CPI, improper PDA validation, missing ownership/signer checks, sysvar spoofing, instruction introspection), Solana-specific lint rules, detection patterns with code examples |
| [tenequm/skills](https://github.com/tenequm/skills) | Tenequm | 15 detailed vulnerability patterns with exploit scenarios and secure alternatives (signer validation, overflow, PDA substitution, type cosplay, account reloading, closing, lamports, CPI, duplicates, bump canonicalization, precision loss, init_if_needed, stale oracle) |
| [nicholasgasior/solana-dev-skill](https://github.com/nicholasgasior/solana-dev-skill) | Nicholas Gasior | 9 vulnerability categories with Anchor/Pinocchio code examples, program-side checklist (39 items), client-side checklist (7 items), security review questions, framework-specific prevention patterns |
| [solana-claude-config](https://github.com/nicholasgasior/solana-claude-config) | Nicholas Gasior | 13-step audit workflow, Anchor architect agent (PDA architecture, token programs, CPI patterns), Anchor engineer agent (modern patterns, constraint patterns, testing), comprehensive Anchor rules (446 lines), security checklists |
| [aeither/solana-anchor-claude-skill](https://github.com/aeither/solana-anchor-claude-skill) | Aeither | Anchor development coding guidelines, platform terminology, Anchor version best practices, project structure conventions, PDA management patterns, space calculation methodology |
| [nicholasgasior/dot-context](https://github.com/nicholasgasior/dot-context) | Nicholas Gasior | 11 Solana/Anchor audit check categories (account constraints, PDA safety, CPI safety, deserialization, error handling, token operations, system accounts, type cosplay, closing accounts), 10 detailed vulnerability knowledge base files |
| [exvulsec/exvul-solana-skill](https://github.com/exvulsec/exvul-solana-skill) | ExVulSec | 5 additional attack vectors (V101–V105): CPI ownership reassignment, unsafe deserialization without input length validation, orphan account lifecycle, partial discriminator matching, dynamic Token-2022 account sizing. Also informed V55 signed-to-unsigned casting improvement |

### Methodology & Agents

| Repository | Author | Contribution |
| --- | --- | --- |
| [sainikethan/nemesis-auditor](https://github.com/sainikethan/nemesis-auditor) | Nemesis | Feynman questioning strategy, state inconsistency analysis methodology — adapted for adversarial reasoning agent |
| [carni-ships/SolidSecs](https://github.com/carni-ships/SolidSecs) | SolidSecs | Protocol-specific checklist approach (lending, AMM, vault, staking, bridge, governance, proxy, account abstraction) — adapted for Solana protocol agent |
| [auditmos/skills](https://github.com/auditmos/skills) | Auditmos | Lending protocol vulnerability patterns, liquidation mechanics, staking reward edge cases — adapted for DeFi attack vectors |

### DeFi Protocol Knowledge

| Repository | Author | Contribution |
| --- | --- | --- |
| [sendaifun/skills](https://github.com/sendaifun/skills) | SendAI | Solana ecosystem protocol integration patterns (Jupiter, Drift, Kamino, Helius) |
| [ethskills.com](https://ethskills.com) | EthSkills | Cross-chain audit checklist methodology, protocol composability patterns |
| [kadenzipfel/scv-scan](https://github.com/kadenzipfel/scv-scan) | Kaden Zipfel | 4-phase systematic audit methodology (load → sweep → validate → report) |

---

## License

[MIT](LICENSE) — see individual attribution repos for their respective licenses.
