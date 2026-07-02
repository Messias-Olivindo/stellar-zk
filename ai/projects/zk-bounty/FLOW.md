# ZK-Bounty: Fluxo de Plataforma

## O Que É

**Protocolo com UI.** Contratos Soroban são a base — ninguém controla. UI é camada de conveniência em cima dos contratos públicos.

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

Pesquisador encontra bug em contrato DeFi Soroban. Dilema:

- **Revelar o bug** → empresa pode ignorar, não pagar, ou alguém exploita antes
- **Não revelar** → empresa não sabe, não paga

ZK-Bounty resolve: pesquisador **prova matematicamente que conhece o bug sem mostrar o bug**. Contrato trava USDC automaticamente. Só depois o pesquisador revela.

```
Immunefi:
  Pesquisador → report → HUMANO AVALIA → empresa paga (ou não)

ZK-Bounty:
  Pesquisador → prova ZK → CONTRATO VERIFICA → USDC trava automático
```

---

## Personas

| Persona | Analogia | O que faz |
|---|---|---|
| **Protocol Owner** | "Contrata segurança" | Cria bounty, deposita USDC |
| **Security Researcher** | "Auditor pago por resultado" | Acha bug, gera prova, saca USDC |
| **Protocolo ZK-Bounty** | "Cartório automático" | Verifica prova, libera dinheiro, registra reputação |
| **Visitor/Auditor** | "Observador" | Lê leaderboard e bounties abertos |

---

## Fluxo Core (5 Passos)

```
PASSO 1 — Protocol Owner cria bounty
─────────────────────────────────────
Acessa UI, conecta wallet Stellar, preenche:
  - Contrato alvo (endereço do AMM vulnerável)
  - Invariante pública ("produto das reservas não diminui")
  - Recompensa (ex: 100 USDC)
  - Prazo (ex: 30 dias)
Assina transação → USDC vai para escrow → bounty aparece público.


PASSO 2 — Pesquisador encontra o bug (off-chain, privado)
──────────────────────────────────────────────────────────
Analisa contrato alvo localmente.
Descobre: "se eu fizer swap com dx=999, a invariante quebra."
Esse número fica APENAS no computador do pesquisador.


PASSO 3 — Pesquisador gera prova ZK (off-chain, ~15s)
──────────────────────────────────────────────────────
CLI computa prova matemática que diz:
  "Existe um número secreto que quebra a invariante."
A prova NÃO revela qual é o número.
A prova GARANTE que ele existe e o pesquisador sabe qual é.

$ zkbounty find-witness --bounty abc123
$ zkbounty prove --bounty abc123 --witness dx=999
$ zkbounty submit --bounty abc123


PASSO 4 — Contrato Soroban verifica a prova (on-chain)
───────────────────────────────────────────────────────
Pesquisador envia prova (200 bytes) para Soroban.
BountyEscrow chama Verifier → Groth16 check via BN254 (Protocol 26).
Prova válida:
  - Status: PROOF_VERIFIED
  - 100 USDC travados para o pesquisador
  - Nullifier salvo (previne duplicata)
  - Evento ProofVerified emitido
Prova inválida: revert, status permanece OPEN, sem custo.


PASSO 5 — Pesquisador revela, saca USDC, ganha reputação
──────────────────────────────────────────────────────────
Pesquisador envia exploit completo (privado) para o dono do protocolo.
Registra hash do report on-chain.
Chama claim_reward() → USDC transferido.
reputation[researcher] += 10 on-chain.
```

---

## Máquina de Estados (Bounty)

```
OPEN
  └─[submit_proof válida]──► PROOF_VERIFIED
                               └─[submit_report_hash]──► REPORT_SUBMITTED
                                                           ├─[claim_reward]──► CLAIMED
                                                           └─[timeout]────────► EXPIRED (USDC retorna ao owner)
  └─[deadline passado]────► EXPIRED
  └─[owner cancela]───────► CANCELLED
```

---

## On-chain vs Off-chain

| O que | Onde | Por quê |
|---|---|---|
| Encontrar o bug (witness) | Off-chain | Privado por definição |
| Geração da prova ZK | Off-chain | CPU-intensivo, privado |
| Conteúdo do report | Off-chain | Privado até disclosure |
| Escrow de recompensa | On-chain | Trustless, sem intermediário |
| Verificação da prova | On-chain | Verificável por qualquer um |
| Reputação/score | On-chain | Auditável, não-falsificável |
| Nullifier registry | On-chain | Previne duplicata |
| UI/leaderboard | Off-chain | Leitura de ledger, sem escrita |

```
Seu computador (privado)         Blockchain Stellar (público)
─────────────────────────        ──────────────────────────────
Encontrar o bug          ──►     NÃO vai pra blockchain
Gerar a prova            ──►     NÃO vai pra blockchain

Enviar prova (200 bytes) ──►     Soroban VERIFICA a prova
                                 Soroban TRAVA o USDC
                                 Soroban REGISTRA reputação
```

---

## Arquitetura Completa

```
┌─────────────────────────────────────────────────────────────────┐
│  OFF-CHAIN                                                       │
│                                                                   │
│  Researcher Machine                 Protocol Owner               │
│  ┌─────────────────────┐           ┌─────────────────────────┐  │
│  │  find-witness CLI    │           │  Deploy scripts (Rust)  │  │
│  │  prove CLI (Circom)  │           │  Circuit compilation    │  │
│  │  proof.bin           │           │  VK generation          │  │
│  │  public_inputs.json  │           │  Report encryption key  │  │
│  └────────┬────────────┘           └───────────┬─────────────┘  │
│           │                                     │                │
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
│  │   - reputation{}     │    └─────────────────────────────┘   │
│  │   - locked_rewards{} │                                       │
│  └──────────────────────┘    ┌─────────────────────────────┐   │
│                               │   Target Contract (demo)    │   │
│  ┌──────────────────────┐    │   - AMM com bug proposital  │   │
│  │   USDC SAC            │    └─────────────────────────────┘   │
│  │   (Stellar Asset     │                                       │
│  │    Contract)         │                                       │
│  └──────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## Por Que ZK É Obrigatório

| Alternativa | Problema |
|---|---|
| Só enviar hash do bug | Contrato não sabe se o hash é de bug real ou qualquer coisa |
| Revelar o bug direto | Pesquisador perde alavancagem antes de garantir pagamento |
| Confiar na empresa | Empresa pode negar recebimento ou não pagar |
| **ZK proof** | Contrato verifica **que o bug existe** sem ver **qual é o bug** ✅ |

---

## Fluxo MVP Hackathon — Checklist de Demo

```
☐ 1. Bounty criado on-chain (AMM + invariante + 100 USDC)
☐ 2. CLI: witness encontrado (dx privado, não aparece na tela)
☐ 3. CLI: prova gerada off-chain (~15s)
☐ 4. CLI: prova submetida → Soroban verifica
☐ 5. Evento ProofVerified emitido
☐ 6. Reward locked on-chain
☐ 7. Claim → USDC transferido, score += 10
☐ 8. Leaderboard atualizado
☐ 9. Caso negativo: prova inválida → rejeição limpa
```

---

## Visão Produto Futuro (Pós-Hackathon)

| Feature | Descrição |
|---|---|
| **Multi-bounty por protocolo** | DEX publica 3 bounties simultâneos com recompensas diferentes |
| **Acesso por reputação** | Bounties premium exigem score >= 50 |
| **Disclosure com timeline** | Protocol tem N dias para responder; timeout = claim automático |
| **Re-prova de patch** | Pesquisador prova que invariante agora PASSA → +5 reputação |
| **Severity staking** | Comunidade vota severidade apostando reputação |
| **SBT Reputation Token** | Soulbound token não-transferível com histórico completo |
| **Invariant Template Marketplace** | Templates auditados de circuitos para facilitar criação de bounties |
