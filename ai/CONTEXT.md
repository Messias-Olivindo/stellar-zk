# Contexto Geral — Workspace do Hackathon Stellar ZK

> **Leia este arquivo primeiro.** Ele orienta qualquer agente de IA sobre o que é este repositório, como está organizado e por onde começar. Não entra no detalhe das ideias — os documentos de ideia fazem isso.

## 1. O Que É

Workspace de participação no hackathon **Stellar Hacks: Real-World ZK** (Stellar Development Foundation, via DoraHacks). Objetivo: construir algo que use **zero-knowledge** de forma essencial e rode na **Stellar** (verificar proofs em contrato Soroban).

## 2. Hackathon Em Resumo

| Campo | Valor |
| --- | --- |
| Prêmio | $10.000 XLM (1º $5.000 · 2º $2.000 · 3º $1.250 · 4º $1.000 · 5º $750) |
| Track | Único, aberto |
| Deadline | **~3 jul 2026** (estendido; original 29 jun) |
| Entrega | Repo público (README claro) + vídeo demo 2–3 min |
| Regra core | ZK **load-bearing** (essencial, não citado) + tocar Stellar |
| Base técnica | Protocolos 25 (X-Ray) e 26 (Yardstick): host functions BN254, Poseidon, verificação Groth16 barata |
| Paths ZK | Circom+Groth16 · Noir (UltraHonk) · RISC Zero (zkVM) |

Detalhes completos, skills de IA, docs oficiais, verifiers de referência e ferramental: **`development/AGENT_CONTEXT.md`**.

## 3. Organização Das Pastas

```
stellar_zk/
└── ai/
    ├── CONTEXT.md              ← este arquivo (orientação geral)
    ├── development/
    │   └── AGENT_CONTEXT.md    ← contexto técnico do hackathon: skills, docs, tooling, paths ZK
    └── projects/
        ├── compliance-zk/IDEA.md                    ← ideia: Agent Payment Vault (compliance p/ agentes)
        ├── confidential-debt-raise/IDEA.md          ← ideia: captação de dívida privada (ICP on-chain)
        └── offchain-confidential-debt-raise/IDEA.md ← ideia: captação de dívida privada (ICP off-chain) [em desenvolvimento ativo]
```

- **`development/`** = contexto do hackathon e da rede (versionável, serve a qualquer ideia).
- **`projects/*/IDEA.md`** = uma pasta por ideia; cada `IDEA.md` é autocontido no formato: Visão Geral → Dor → Como Funciona → Onde Entra ZK → ... → Pitch → Estado e Decisões em Aberto → Próximos Passos.

## 4. As Ideias (uma linha cada)

- **offchain-confidential-debt-raise** *(foco atual)* — empresa off-chain capta dívida com **identidade e saúde provadas por ZK**, mas **valor e balanço privados**.
- **confidential-debt-raise** — mesma tese para **protocolos já on-chain** na Stellar (oráculo dissolvido, tese de token).
- **compliance-zk** — **Agent Payment Vault**: cofre Soroban que só libera pagamento de agente de IA se a política for verificável, com ZK opcional.

Cada ideia guarda seu estado e decisões em aberto na seção **"Estado e Decisões em Aberto"** do próprio `IDEA.md`.

## 5. Por Onde Um Agente Começa

1. Ler este `CONTEXT.md` (orientação).
2. Ler `development/AGENT_CONTEXT.md` (técnica do hackathon + skills a instalar).
3. Ler o `IDEA.md` da ideia em que vai trabalhar.
4. Antes de codar na Stellar: instalar a **Stellar Dev Skill** e a **ZK Proofs skill**, e ler `skills.stellar.org` + `developers.stellar.org/llms.txt` (instruções em `AGENT_CONTEXT.md`).

## 6. Convenções

- **Idioma:** documentos e discussão em **português (pt-BR)**; código, commits e nomes técnicos em inglês.
- **Formato de ideia:** seguir o padrão de seções numeradas dos `IDEA.md` existentes (referência de redação: `compliance-zk/IDEA.md`).
- **Rastro de decisões:** toda decisão de produto/design fica registrada na seção "Estado e Decisões em Aberto" do `IDEA.md` da ideia, não solta.
- **ZK primeiro:** qualquer ideia precisa que o ZK seja load-bearing — se sair, o projeto quebra.
