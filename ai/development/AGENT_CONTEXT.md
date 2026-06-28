# Agent Context — Stellar Hacks: Real-World ZK

> Arquivo de configuração de contexto para agentes de IA que vão construir neste hackathon.
> Aponte qualquer agente (Claude Code, Cursor, Codex) para este arquivo **antes** de começar a codar.
> Fonte: aba *Resources* + *Detail* + *Ideas* de https://dorahacks.io/hackathon/stellar-hacks-zk

---

## 0. Bootstrap rápido do agente

Ordem recomendada de leitura/ingestão para qualquer agente novo:

1. Ler `https://skills.stellar.org/` (skills agent-readable da Stellar).
2. Ingerir `https://developers.stellar.org/llms.txt` (digest dos docs em formato LLM).
3. Ler os 2 docs oficiais: **ZK Proofs on Stellar** e **Privacy on Stellar** (links na §3).
4. Instalar a skill `stellar-dev` + a skill `zk-proofs` (§2).
5. Estudar 1 verifier de referência conforme o path escolhido (§5).

Frase pronta para colar no agente:
> "Read skills.stellar.org and developers.stellar.org/llms.txt before you start building on Stellar. We are verifying ZK proofs inside a Soroban smart contract. Use the latest Soroban SDK (Protocol 26 / Yardstick support)."

---

## 1. Fatos do hackathon (constraints duras)

| Campo | Valor |
|---|---|
| Nome | Stellar Hacks: Real-World ZK |
| Org | Stellar Development Foundation (DoraHacks) |
| Prêmio | $10.000 XLM — 1º $5.000 · 2º $2.000 · 3º $1.250 · 4º $1.000 · 5º $750 |
| Track | Single open innovation track (sem sub-tracks) |
| Abertura | 15 jun 2026, 00:00 PST |
| **Deadline** | **29 jun 2026, 12:00 PST** (estendido — ver nota) |
| Repo público | **Obrigatório** (GitHub/GitLab/Bitbucket + README claro) |
| Vídeo demo | **Obrigatório** (2–3 min, mostrar funcionando + papel do ZK) |
| Regra core | ZK deve ser *load-bearing* (poder real, não só citado no README) + tocar Stellar (verificar proof em contrato, ou integrar testnet/mainnet) |
| Suporte | Discord `#zk-chat` (discord.gg/stellardev) · Telegram (t.me/+e898qibDUVExODkx) |
| Nota BR | Form pergunta sobre Stellar Builder House em São Paulo — possível networking presencial |

Critério de vitória implícito: projeto *sharp* e bem executado ganha mesmo sendo "mild". ZK essencial + doc clara > demo polida e vazia.

---

## 2. Skills de IA para instalar (núcleo do pedido)

Estes são os pacotes de contexto agent-readable. **Instalar antes de codar.**

### Stellar Skills (hub oficial — começar aqui)
- Hub: https://skills.stellar.org/ — skills dedicadas: Soroban, dApps/wallets, assets, data/APIs, agentic payments, **ZK Proofs**. Funciona com qualquer agente.
- **ZK Proofs skill (direto):** https://skills.stellar.org/skills/zk-proofs/SKILL.md — verificar proofs Groth16 com BLS12-381, BN254 e Poseidon.

### stellar-dev-skill (repo open-source que alimenta o hub)
- Repo: https://github.com/stellar/stellar-dev-skill — Soroban, SDKs, RPC, wallet, passkeys, padrões de segurança.
- **Claude Code:** `/plugin marketplace add stellar/stellar-dev-skill` → `/plugin install stellar-dev@stellar-dev`
- **Cursor:** add `stellar/stellar-dev-skill`
- **Codex:** `git clone https://github.com/stellar/stellar-dev-skill ~/.codex/skills/stellar-dev-skill`

### stellar-build (jornada completa)
- https://github.com/kaankacar/stellar-build — instalador de 42 skills, da ideia ao deploy em mainnet + submissão SCF grant. Inclui 6 agentes persona DevRel.

### OpenZeppelin Skills (contratos seguros)
- https://github.com/OpenZeppelin/openzeppelin-skills — skills Claude Code p/ contratos Stellar seguros.
- Install: `/plugin marketplace add OpenZeppelin/openzeppelin-skills` → `/plugin install openzeppelin-skills`

### Recursos de contexto LLM
- **Building with AI (docs):** https://developers.stellar.org/docs/build/building-with-ai
- **llms.txt:** https://developers.stellar.org/llms.txt — digest machine-readable dos docs.

---

## 3. Docs oficiais — começar pelo ZK & Privacy

- **ZK Proofs on Stellar:** https://developers.stellar.org/docs/build/apps/zk — referência core. Host functions BN254 e Poseidon/Poseidon2, como funciona verificação de proof, exemplos de código.
- **Privacy on Stellar:** https://developers.stellar.org/docs/build/apps/privacy — mapa do stack de privacidade: Privacy Pools, Confidential Tokens, verifiers ZK on-chain, primitivas crypto.
- **Stellar X-Ray (Protocol 25):** https://stellar.org/blog/developers/announcing-stellar-x-ray-protocol-25 — por que as primitivas foram adicionadas.
- **Yardstick (Protocol 26) upgrade guide:** https://stellar.org/blog/foundation-news/stellar-yardstick-protocol-26-upgrade-guide — o que P26 adicionou p/ ZK e por que verificação ficou mais barata.

### Base técnica (resumo)
- **Protocol 25 "X-Ray":** host functions nativos p/ BN254 (curva elíptica) + Poseidon/Poseidon2 (hash ZK-friendly).
- **Protocol 26 "Yardstick":** +9 host functions BN254 (multi-scalar mult, aritmética em scalar field, curve-membership). Move math pesada p/ host layer → verificação (incl. NoirLang) mais barata.
- **+ BLS12-381** de protocolos anteriores.
- Modelo: gera proof **off-chain** (Noir/Circom/RISC Zero) → deploy verifier contract na Stellar → contrato checa a proof. Primitivas são *building blocks*, não produto pronto.

---

## 4. Escolha do path ZK (3 opções provadas)

| Path | Linguagem | Proof system | Trade-off | Quando usar |
|---|---|---|---|---|
| **RISC Zero** | Rust comum (zkVM) | Groth16 | computa muito off-chain, prova execução | computação geral pesada provável |
| **Noir** (Aztec) | DSL tipo Rust | UltraHonk | fácil ler/escrever; proofs maiores, verify mais caro (P26 baixou custo) | circuits legíveis, dev rápido |
| **Circom** | DSL constraint-based | Groth16 | mais baixo nível, difícil; verify mais barato | máxima eficiência on-chain |

### Tutoriais E2E (James Bachini)
- RISC Zero: https://jamesbachini.com/stellar-risc-zero-games/
- Circom: https://jamesbachini.com/circom-on-stellar/
- Noir: https://jamesbachini.com/noir-on-stellar/

### Docs das tools de circuito
- Noir: https://noir-lang.org/docs/
- RISC Zero: https://dev.risczero.com/
- Circom: https://docs.circom.io/

---

## 5. Verifiers de referência (starter code — fork/estudar)

- **RISC Zero (Groth16) verifier** (Nethermind): https://github.com/NethermindEth/stellar-risc0-verifier — companion: https://stellar.org/blog/developers/risc-zero-verifier
- **UltraHonk verifier (Noir/Barretenberg):**
  - https://github.com/yugocabrio/rs-soroban-ultrahonk
  - https://github.com/indextree/ultrahonk_soroban_contract
- **Groth16 Verifier Contracts (Soroban examples):** https://github.com/stellar/soroban-examples/tree/main/groth16_verifier
- **Stellar Private Payments (Privacy Pools PoC, Nethermind):** https://github.com/NethermindEth/stellar-private-payments — Circom + Groth16 + Soroban. Pool contract + verifier Groth16 + ASP membership/non-membership. Proofs geradas client-side no browser via WASM (segredos não saem do device). Docs: https://nethermindeth.github.io/stellar-private-payments/
  - ⚠️ protótipo de pesquisa, não auditado. Não usar com ativos reais.
- **Soroban P25 preview examples:** https://github.com/jayz22/soroban-examples/tree/p25-preview/p25-preview

---

## 6. SDK / primitivas — referência de API

- **Soroban SDK — BN254:** https://docs.rs/soroban-sdk/latest/soroban_sdk/_migrating/v25_bn254/index.html
- **Soroban SDK — Poseidon:** https://docs.rs/soroban-sdk/latest/soroban_sdk/_migrating/v25_poseidon/index.html
- **CAPs (deep cuts):**
  - BN254 — CAP-0074: https://github.com/stellar/stellar-protocol/blob/master/core/cap-0074.md
  - Poseidon/Poseidon2 — CAP-0075: https://github.com/stellar/stellar-protocol/blob/master/core/cap-0075.md
  - BLS12-381 — CAP-0059: https://github.com/stellar/stellar-protocol/blob/master/core/cap-0059.md

---

## 7. Tooling core Stellar (dev/deploy)

- Docs: https://developers.stellar.org/
- SDKs (usar versão mais nova p/ P26): https://developers.stellar.org/docs/tools/sdks
- Stellar CLI: https://developers.stellar.org/docs/tools/cli
- Lab (gerar/fundear conta testnet no browser): https://developers.stellar.org/docs/tools/lab
- Quickstart (rede local via Docker): https://developers.stellar.org/docs/tools/quickstart
- Scaffold Stellar (lifecycle completo): https://scaffoldstellar.org
- Stellar Wallets Kit: https://stellarwalletskit.dev/
- OpenZeppelin on Stellar (lib auditada, Wizard, MCP, detectors): https://www.openzeppelin.com/networks/stellar

### Smart contract building blocks (docs)
- Getting Started: https://developers.stellar.org/docs/build/smart-contracts/getting-started
- Authorization: https://developers.stellar.org/docs/build/guides/auth
- Storage: https://developers.stellar.org/docs/build/guides/storage
- Testing: https://developers.stellar.org/docs/build/guides/testing

---

## 8. Privacidade — contexto conceitual

- **Confidential Token Association** (SDF, Nethermind, OpenZeppelin, Zama): https://www.confidentialtoken.org/ — padrão de confidencialidade on-chain por encriptação, compatível com interfaces de token existentes. Vídeo: https://www.youtube.com/watch?v=6NnDqVQYOHM
- **Privacy Pools whitepaper** (Buterin, Illum, Nadler, Schär, Soleimani): https://privacypools.com/whitepaper.pdf — base de privacy pools compliant (deposit/withdraw visível, transfer interno privado, ASP allow/deny).

---

## 9. Recursos de comunidade

- Stellar Ecosystem Resources: https://github.com/stellar/ecosystem-resources/
- Stellar Hackathon FAQ: https://github.com/briwylde08/stellar-hackathon-faq
- Stellar Ecosystem DB (achar projeto existente antes de criar): https://github.com/lumenloop/stellar-ecosystem-db

---

## 10. Catálogo de ideias (mild → wild) — para escopo

**🟢 Mild:** proof-of-balance/funds (range proof) · age/eligibility check · private allowlist (Merkle membership) · verifiable off-chain compute · anonymous attestation.

**🟡 Medium:** private/shielded transfer (deposit-withdraw) · confidential payroll/invoicing · transfer privado com view key (selective disclosure) · private credential/reputation · sealed-bid auction/vote · proof-of-reserves.

**🟠 Spicy:** privacy pool compliant c/ ASP · confidential token · private RWA settlement · identity on-chain privacy-preserving (nullifier p/ Sybil) · private DAO governance.

**🔴 Wild:** wallet stablecoin totalmente shielded · corredor de remessa cross-border privado · DeFi confidencial · sistema UTXO-style privado · proof aggregation/recursive · bridge cross-chain privado (via compat BN254 ↔ EIP precompiles).

> Regra ao escolher: ZK genuinamente essencial + shippável no prazo + doc clara.

---

## 11. Mídia / primer

- Vídeo primer: https://www.youtube.com/watch?v=nl1uD1Kcmvw
- Twitter Space primer: https://x.com/i/spaces/1AGRnnLddNyGl
