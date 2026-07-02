# ZK-Bounty: Fluxo de Plataforma

## O Que É

**Protocolo com UI.** Contratos Soroban são a base — ninguém controla. UI é camada de conveniência em cima dos contratos públicos.

Qualquer pessoa pode:
- Criar interface própria em cima dos contratos
- Interagir direto via CLI sem UI
- Auditar tudo on-chain publicamente

```
┌─────────────────────────────────────────────┐
│  PROTOCOLO (on-chain, ninguém controla)     │
│                                             │
│  Contratos Soroban:                         │
│  - BountyEscrow   (guarda USDC, verifica)   │
│  - Verifier       (valida prova ZK)         │
│  - Reputation     (score de pesquisadores)  │
└─────────────────────────────────────────────┘
           ▲                    ▲
           │                    │
┌──────────┴──────┐   ┌────────┴──────────┐
│  UI WEB         │   │  CLI              │
│  (qualquer um   │   │  (pesquisador     │
│   pode fazer)   │   │   gera prova)     │
└─────────────────┘   └───────────────────┘
```

---

## O Problema Que Resolve

Pesquisador encontra bug em contrato DeFi Soroban. Dilema clássico:

- **Revelar o bug primeiro** → empresa pode ignorar, não pagar, ou outra pessoa exploita antes
- **Não revelar** → empresa não tem como saber que você encontrou algo real, não paga

ZK-Bounty quebra esse dilema: pesquisador **prova matematicamente que conhece o bug sem mostrar o bug**. Contrato trava USDC automaticamente. Só depois o pesquisador revela com segurança.

```
Immunefi/HackerOne:
  Pesquisador → envia report completo → HUMANO AVALIA → empresa decide pagar (ou não)

ZK-Bounty:
  Pesquisador → envia prova matemática → CONTRATO VERIFICA → USDC trava automático → aí pesquisador revela
```

---

## Personas e Responsabilidades

### Protocol Owner (dono do protocolo Soroban)
- Qualquer desenvolvedor que deployou contrato Soroban (AMM, lending, stablecoin)
- **Responsabilidade:** definir invariante pública clara e depositar recompensa em USDC
- **Risco:** se alguém provar que a invariante quebra, perde o USDC depositado
- **Ganho:** segurança contínua pós-deploy sem pagar auditoria recorrente

### Security Researcher (pesquisador de segurança)
- Especialista que analisa contratos buscando vulnerabilidades
- **Responsabilidade:** encontrar witness privado, gerar prova, fazer disclosure responsável
- **Risco:** tempo gasto sem garantia de encontrar bug (mas se encontrar, pagamento é garantido)
- **Ganho:** USDC automático + reputação on-chain permanente

### Protocolo ZK-Bounty (contratos Soroban)
- Não é empresa. É código. Ninguém controla.
- **Responsabilidade:** verificar prova, custodiar USDC, registrar reputação
- **Não faz:** avaliar relatórios, decidir severidade, mediar disputas subjetivas

---

## Fluxo Detalhado — Passo a Passo

### FASE 1: Protocol Owner Cria o Bounty

**O que acontece:**

O dono do protocolo acessa a UI web, conecta wallet Stellar (Freighter), e preenche o formulário de criação de bounty.

**Dados que ele fornece:**
```
Contrato alvo:      G... (endereço do contrato AMM na Stellar)
WASM hash:          abc123... (hash do bytecode do contrato)
Invariante (texto): "O produto das reservas não pode diminuir após swap"
Invariant ID:       bytes32 que identifica o circuito ZK associado
Verification Key:   VK do circuito que vai verificar a prova (gerada off-chain)
Recompensa:         100 USDC
Prazo:              30 dias
```

**Por que o WASM hash?** Liga o bounty a uma versão específica do contrato. Se o protocolo fizer update, o bounty antigo não vale mais para o novo código.

**Por que a Verification Key?** O contrato precisa saber qual circuito ZK aceitar. VK é o "molde" da prova — define que tipo de prova é válida para aquela invariante específica.

**On-chain:**
```
BountyEscrow.create_bounty(
  owner:              G...,
  target_contract:    G... (AMM),
  target_wasm_hash:   bytes32,
  invariant_id:       bytes32,
  verifier:           G... (Verifier contract),
  reward_token:       USDC SAC address,
  reward_amount:      100_000_000 (100 USDC, 7 decimais),
  deadline:           timestamp
)

→ USDC transferido do owner para o escrow
→ Bounty salvo no ledger com status: OPEN
→ Evento BountyCreated emitido
```

**UI mostra para o mundo:**
```
┌────────────────────────────────────────────┐
│  Bounty #abc123                            │
│  Target: Vulnerable Soroban AMM            │
│  Invariant: new_x * new_y >= old_x * old_y │
│  Reward: 100 USDC                          │
│  Deadline: 2026-07-31                      │
│  Status: OPEN                              │
└────────────────────────────────────────────┘
```

---

### FASE 2: Pesquisador Analisa o Contrato (Off-chain, Privado)

**O que acontece:**

Pesquisador vê o bounty na UI, baixa o WASM do contrato alvo, analisa localmente.

**Ferramentas que usa:**
- `soroban contract fetch` → baixa bytecode
- Decompilador / análise manual do código Rust
- Fuzzer ou análise matemática para encontrar input que quebra a invariante

**Para o AMM do exemplo:**

O pesquisador descobre que a fórmula do contrato está errada:
```
Fórmula correta (uniswap):  dy = reserve_y * dx / (reserve_x + dx)
Fórmula bugada no contrato: dy = reserve_y * dx / reserve_x   ← paga mais do que deveria
```

Com `reserve_x = 1000, reserve_y = 2000`, qualquer `dx` positivo quebra a invariante.

Exemplo com `dx = 500`:
```
dy_buggy = 2000 * 500 / 1000 = 1000
new_x = 1000 + 500 = 1500
new_y = 2000 - 1000 = 1000
Produto: 1500 * 1000 = 1.500.000

Produto original: 1000 * 2000 = 2.000.000

1.500.000 < 2.000.000  ← invariante quebrada ✓
```

O pesquisador escolhe `dx = 500` como seu **witness privado**. Esse número fica só no computador dele.

---

### FASE 3: Geração da Prova ZK (Off-chain, ~15 segundos)

**O que acontece:**

CLI do pesquisador roda o circuito Circom/Noir localmente. O circuito:
1. Recebe o witness privado (`dx = 500`, `salt = random`)
2. Executa a matemática do AMM bugado
3. Verifica que a invariante quebra
4. Gera comprometimento criptográfico do witness: `commitment = Poseidon(dx, salt, bounty_id)`
5. Gera nullifier para evitar replay: `nullifier = Poseidon(commitment, bounty_id)`
6. Produz prova Groth16 (~200 bytes)

```bash
$ zkbounty prove \
    --bounty-id abc123 \
    --dx 500 \
    --reserve-x 1000 \
    --reserve-y 2000 \
    --salt 0xdeadbeef

Generating ZK proof...
  Private witness: dx=500 (stays local)
  Computing Poseidon commitments...
  Running Groth16 prover...
  
✓ Proof generated in 12.3s
  proof.bin          (203 bytes)
  public_inputs.json (visible to everyone):
    bounty_id:           abc123
    target_wasm_hash:    abc...
    invariant_id:        def...
    reserve_x:           1000
    reserve_y:           2000
    witness_commitment:  0x7f3a...  ← hash do dx, não o dx
    nullifier:           0x9b2c...
```

**O que `public_inputs.json` revela:** que existe um witness válido para aquelas reservas específicas. **Não revela:** qual é o `dx`.

---

### FASE 4: Submissão para Soroban (On-chain)

**O que acontece:**

CLI envia prova + public inputs para o contrato BountyEscrow.

```bash
$ zkbounty submit --bounty-id abc123

Submitting to Soroban...
  Transaction signed with: G...researcher
  Sending to network...
```

**Dentro do contrato BountyEscrow:**

```
1. Verifica se bounty existe e está OPEN
2. Verifica se nullifier já foi usado → se sim, rejeita (duplicata)
3. Verifica se public_inputs.bounty_id bate com o bounty
4. Verifica se public_inputs.target_wasm_hash bate com o contrato alvo
5. Chama Verifier.verify(proof, public_inputs) → Groth16 check via BN254

Se Verifier retorna TRUE:
  - nullifiers.insert(nullifier)          → bloqueia replay futuro
  - locked_rewards[researcher] = 100 USDC → trava dinheiro
  - bounty.status = PROOF_VERIFIED
  - bounty.winner = researcher_address
  - Emite ProofVerified, RewardLocked

Se Verifier retorna FALSE:
  - revert (tudo volta ao estado anterior)
  - Status permanece OPEN
  - Pesquisador perde apenas gas (~0.01 XLM)
```

```bash
# CLI recebe resposta:
✓ Proof verified on-chain.
✓ 100 USDC locked for G...researcher
  Transaction: TXHASH...
  Ledger: 1234567
```

**UI atualiza:**
```
Status: PROOF_VERIFIED
Winner: G...researcher (pending claim)
Reward: 100 USDC (locked)
```

---

### FASE 5: Disclosure Responsável (Off-chain + On-chain)

**O que acontece:**

Pesquisador agora tem garantia de pagamento. Pode revelar o bug com segurança.

**Off-chain:**
```
Pesquisador escreve report privado:
  - Descrição do bug
  - O exploit completo (dx=500 e por que quebra)
  - Impacto estimado
  - Sugestão de fix

Envia para o dono do protocolo (email, DM, Telegram)
Calcula: report_hash = sha256(report_content)
```

**On-chain:**
```
BountyEscrow.submit_report_hash(
  researcher:  G...researcher,
  bounty_id:   abc123,
  report_hash: 0x...  ← compromisso criptográfico com o report
)

→ Status: REPORT_SUBMITTED
→ Emite ReportSubmitted
```

O `report_hash` on-chain prova que o pesquisador se comprometeu com um report específico naquele momento. Se o protocolo alegar que não recebeu nada, o hash é evidência de que existia um report antes do claim.

**Janela de resposta do protocolo:**
```
- Protocol Owner tem N dias para responder
- Se aceitar: libera imediatamente
- Se não responder: timeout → pesquisador pode claim automaticamente
- Se disputar: (versão futura) staking de reputação pela comunidade
```

---

### FASE 6: Claim e Reputação (On-chain)

**O que acontece:**

```
BountyEscrow.claim_reward(
  researcher: G...researcher,
  bounty_id:  abc123
)

→ Transfere 100 USDC para G...researcher
→ reputation[G...researcher] += 10
→ discoveries[G...researcher] += 1
→ bounty.status = CLAIMED
→ Emite RewardClaimed, ReputationUpdated
```

**UI leaderboard:**
```
┌──────────────────────────────────────────────────┐
│  Leaderboard                                     │
│                                                  │
│  #1  G...researcher   Score: 10   Discoveries: 1 │
│      Last: AMM constant product break            │
│                                                  │
│  Bounties Closed                                 │
│  abc123 — AMM invariant — CLAIMED by G...        │
└──────────────────────────────────────────────────┘
```

---

### Caso Negativo: Prova Inválida

```bash
# Pesquisador altera public inputs tentando enganar o contrato
$ zkbounty submit --bounty-id abc123 --tampered

Submitting to Soroban...
✗ Transaction failed: Verifier rejected proof
  Status: OPEN (unchanged)
  Reputation: unchanged
  Cost: ~0.01 XLM (gas)
```

```bash
# Pesquisador tenta submeter mesma prova duas vezes
$ zkbounty submit --bounty-id abc123  # segunda vez

✗ Transaction failed: Nullifier already used
  This discovery was already claimed.
```

---

## Máquina de Estados Completa

```
OPEN
  │
  ├─[submit_proof inválida]──► OPEN (sem mudança)
  │
  ├─[submit_proof válida]────► PROOF_VERIFIED
  │                              │
  │                              ├─[submit_report_hash]──► REPORT_SUBMITTED
  │                              │                           │
  │                              │                           ├─[owner aceita]──► CLAIMED
  │                              │                           ├─[timeout]────────► CLAIMED (automático)
  │                              │                           └─[owner disputa]──► DISPUTED (futuro)
  │                              │
  │                              └─[timeout sem report]──► EXPIRED (USDC volta ao owner)
  │
  ├─[deadline passado sem proof]── EXPIRED
  └─[owner cancela antes de proof]─ CANCELLED
```

---

## On-chain vs Off-chain

| O que | Onde | Por quê |
|---|---|---|
| Encontrar o bug (witness) | Off-chain | Privado por definição |
| Geração da prova ZK | Off-chain | CPU-intensivo, privado, não precisa de blockchain |
| Conteúdo do exploit | Off-chain | Privado até disclosure completo |
| Escrow de recompensa | On-chain | Trustless, sem intermediário |
| Verificação da prova Groth16 | On-chain | Verificável por qualquer um; usa BN254 Protocol 26 |
| Reputação / score | On-chain | Auditável, não-falsificável, permanente |
| Nullifier registry | On-chain | Previne duplicatas e replay |
| Hash do report | On-chain | Comprometimento criptográfico sem revelar conteúdo |
| UI / leaderboard | Off-chain | Leitura de ledger, sem escrita |

```
Computador do pesquisador (privado)    Blockchain Stellar (público)
───────────────────────────────────    ──────────────────────────────
Encontrar dx=500              ──►      NÃO vai pra blockchain
Gerar proof.bin               ──►      NÃO vai pra blockchain
Conteúdo do report            ──►      NÃO vai pra blockchain

Enviar proof.bin (200 bytes)  ──►      Soroban VERIFICA via Groth16
report_hash (32 bytes)        ──►      Soroban REGISTRA comprometimento
claim_reward()                ──►      Soroban TRANSFERE USDC + reputação
```

---

## Arquitetura de Contratos

```
┌─────────────────────────────────────────────────────────────────┐
│  OFF-CHAIN                                                       │
│                                                                   │
│  Researcher Machine                 Protocol Owner               │
│  ┌─────────────────────┐           ┌─────────────────────────┐  │
│  │  find-witness CLI    │           │  Deploy scripts (Rust)  │  │
│  │  prove CLI (Circom)  │           │  Circuit compilation    │  │
│  │  proof.bin           │           │  VK generation          │  │
│  │  public_inputs.json  │           │  Report (privado)       │  │
│  └────────┬────────────┘           └───────────┬─────────────┘  │
└───────────┼─────────────────────────────────────┼────────────────┘
            │ submit_proof / claim                │ create_bounty
            ▼                                     ▼
┌─────────────────────────────────────────────────────────────────┐
│  ON-CHAIN (Stellar / Soroban)                                    │
│                                                                   │
│  ┌──────────────────────┐    ┌─────────────────────────────┐   │
│  │   BountyEscrow       │───►│   Verifier (Groth16)        │   │
│  │   - bounties[]       │    │   - verify(proof, inputs)   │   │
│  │   - nullifiers{}     │    │   - BN254 host functions    │   │
│  │   - reputation{}     │    │     (Protocol 26)           │   │
│  │   - locked_rewards{} │    └─────────────────────────────┘   │
│  │   - report_hashes{}  │                                       │
│  └──────────────────────┘    ┌─────────────────────────────┐   │
│                               │   Target Contract (demo)    │   │
│  ┌──────────────────────┐    │   - AMM com bug proposital  │   │
│  │   USDC SAC            │    │   - invariante pública      │   │
│  │   (Stellar Asset     │    └─────────────────────────────┘   │
│  │    Contract)         │                                       │
│  └──────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Por Que ZK É Obrigatório

| Alternativa | Por que não funciona |
|---|---|
| Só enviar hash do witness | Contrato não consegue verificar que o hash é de input que realmente quebra a invariante. Qualquer hash passaria. |
| Revelar o bug direto | Pesquisador perde toda alavancagem. Protocolo pode não pagar depois de receber o exploit. |
| Commit-reveal simples | Pesquisador faz commit do hash, depois revela. Mas o contrato não sabe se o revealed input realmente quebra a invariante sem executar o contrato (cara, perigoso on-chain). |
| **ZK proof** | Contrato verifica matematicamente que o witness satisfaz os constraints do circuito — inclui verificar que a invariante quebra — sem ver o witness. ✅ |

---

## Modelo de Reputação

Reputação é não-transferível. Gerada apenas por eventos objetivos on-chain.

```
reputation[researcher] += 10    quando proof válida + report_hash submetidos
reputation[researcher] += 5     quando patch verificado (futuro)
reputation[researcher] -= 5     quando nullifier detectado como tentativa de fraude (futuro)

discoveries[researcher] += 1    por bounty claimed
```

**Impacto (MVP):**
- Score visível no leaderboard público
- Histórico auditável por qualquer protocolo

**Impacto (futuro):**
- Score < 20: só bounties públicos abertos
- Score >= 50: acesso a bounties premium (maiores recompensas, contratos mais críticos)
- Multiplicador: `reward_final = base × (1 + score / 1000)`

---

## Fluxo MVP Hackathon — Checklist de Demo

```
☐ 1. AMM vulnerável deployado na Stellar testnet
☐ 2. Bounty criado on-chain (invariante + 100 USDC em escrow)
☐ 3. CLI: witness encontrado (dx privado, não aparece na tela)
☐ 4. CLI: prova gerada off-chain (~15s de output visível)
☐ 5. CLI: prova submetida → Soroban verifica via Groth16
☐ 6. Evento ProofVerified visível no ledger
☐ 7. Reward locked on-chain para o pesquisador
☐ 8. Report hash submetido on-chain
☐ 9. Claim → USDC transferido, score += 10
☐ 10. Leaderboard mostra score atualizado
☐ 11. Caso negativo: prova inválida → rejeição limpa sem mudança de estado
☐ 12. Caso negativo: nullifier duplicado → rejeição com mensagem clara
```

---

## Visão Produto Futuro (Pós-Hackathon)

| Feature | Descrição |
|---|---|
| **Multi-bounty por protocolo** | DEX publica 3 bounties simultâneos com recompensas diferentes por severidade |
| **Acesso por reputação** | Bounties premium exigem `reputation >= 50` |
| **Disclosure com timeline** | Protocol tem N dias para responder; timeout = claim automático |
| **Re-prova de patch** | Pesquisador prova que invariante agora PASSA com mesmo input → +5 rep |
| **Severity staking** | Pesquisadores com rep >= 30 votam severidade apostando reputação |
| **SBT Reputation Token** | Soulbound token não-transferível com histórico completo on-chain |
| **Invariant Template Marketplace** | Templates auditados de circuitos para facilitar criação de bounties sem expertise ZK |
| **Cross-protocol bounties** | Protocolo A contrata pesquisador high-score que encontrou bugs em B |
