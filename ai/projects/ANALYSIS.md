# Análise Comparativa das Ideias — Concorrentes, Viabilidade e Forças/Fraquezas

> Documento transversal do workspace. Compara as quatro ideias em `projects/*` sob três lentes: **concorrentes**, **viabilidade de negócio** e **pontos fortes/fracos**. Serve para decidir onde investir o tempo restante do hackathon (**deadline ~3 jul 2026**). Cada `IDEA.md` continua sendo a fonte autocontida da respectiva ideia; aqui é a visão de portfólio.

## 0. Resumo Executivo

| Ideia | Tese em uma linha | ZK load-bearing? | Escopo p/ MVP | Viabilidade negócio | Risco hackathon |
| --- | --- | --- | --- | --- | --- |
| **compliance-zk** (Agent Payment Vault) | Cofre Soroban que só paga se a política for verificável; ZK opcional para whitelist/credencial privada. | **Parcial** — ZK é *opcional*, regras públicas já resolvem o core. | **Baixo** (fácil de demonstrar). | **Alta** — narrativa de agentic payments quente. | ZK pode parecer decorativo → fere regra core. |
| **offchain-confidential-debt-raise** | Empresa off-chain capta dívida com saúde provada por ZK, valor e balanço privados. | **Sim** — anti-limão + threshold oculto quebram sem ZK. | **Alto** (oráculo assinado + 2 circuitos + escrow). | **Média/Alta** (mercado grande, regulatório pesado). | Escopo grande + dependência de oráculo. |
| **confidential-debt-raise** (on-chain) | Protocolo já on-chain capta dívida com valor oculto, saúde provada sobre estado on-chain. | **Sim** — igual à irmã, sem oráculo. | **Médio** (oráculo dissolvido corta 1 subsistema). | **Média** (mercado menor e jovem). | Métrica de capacidade on-chain indefinida. |
| **zk-bounty** | Escrow trustless de bug bounty: hacker prova ZK que achou a falha; exploit só revelado após recompensa travada. | **Sim** — provar vuln sem revelar witness é o núcleo. | **Baixo/Médio** (escrow + circuito toy). | **Média** — dor real, mas produto real é tecnicamente duro. | Baixo p/ demo; **gap enorme entre MVP toy e produto real**. |

**Recomendação p/ hackathon:** ver §6.

---

## 1. compliance-zk — Agent Payment Vault

### 1.1 Concorrentes

| Concorrente | Categoria | O que cobre | Gap que a ideia explora |
| --- | --- | --- | --- |
| **Circle Agent Stack** | Custodial / stack fechada | Agent wallets, spending controls, USDC. | Fechado; enforcement não vive on-chain. |
| **Turnkey** | Custody / key mgmt | Wallets programáveis, policy engine, limites. | Enforcement depende da infra Turnkey. |
| **Fireblocks Agentic Payments** | Enterprise | Policy engine, KYT, Travel Rule, audit trail. | Maduro mas fechado/enterprise. |
| **x402 / Stellar x402** | Protocolo de pagamento | "Como" o agente paga API/serviço. | Não resolve "quem pode, quanto, p/ quem". |
| **zkMe / Privado ID** | ZK identity | KYC/KYB e credenciais privadas. | Provam identidade; não são vault Stellar-native. |
| **Safe (ex-Gnosis) modules** | Account abstraction | Módulos de policy/limite em smart accounts (EVM). | Equivalente EVM — valida a dor; gap = Stellar + ZK. |

**Leitura:** espaço **lotado e bem-capitalizado**. Dor validada (bom p/ mercado, mau p/ diferenciação). Diferencial = *enforcement on-chain Soroban + privacy-preserving compliance*, recorte estreito.

### 1.2 Viabilidade de negócio

- **Timing:** excelente. Agentic payments é narrativa 2025–2026.
- **Monetização:** fee por volume/vault; precisa de **volume real de agentes** — ainda incipiente.
- **GTM:** primitive aberta compete com plataformas turnkey de UX pronta. **Fricção alta** de adoção.
- **Fosso:** raso. Vault com policy é copiável; valor está em rede/integrações que a ideia não tem.
- **Regulatório:** custódia programática de fundos de terceiros → possível money transmitter dependendo do desenho.

**Veredito:** boa dor, mercado quente, **fosso fraco e ZK acessório**. Vira produto se for a camada open-source de referência que grandes integram, não app final.

### 1.3 Forças e fraquezas

**Forças**
- Demo objetiva: aprovado vs. bloqueado em minutos → ótimo p/ juízes.
- Menor escopo técnico → termina antes do deadline.
- Narrativa AI agents atrai atenção imediata.

**Fraquezas**
- **ZK é opcional** — regras públicas já resolvem o core. Fere "ZK load-bearing".
- Diferenciação frágil contra incumbentes fortes.
- Sem privacidade obrigatória, é "só mais um vault com policy".

**Blindar:** tornar ZK *não removível* — whitelist/credencial de fornecedor **tem** de ser privada (política = segredo de negócio). Reposicionar de "vault com ZK opcional" para "vault cuja política **é** privada por construção".

---

## 2. offchain-confidential-debt-raise

### 2.1 Concorrentes

| Concorrente | Categoria | O que cobre | Gap que a ideia explora |
| --- | --- | --- | --- |
| **Maple Finance** | Crédito institucional on-chain | Empréstimo sub-colateralizado. | Transparente; aqui valor + números privados. |
| **Goldfinch** | RWA credit | Underwriting por pools delegadas. | Sem privacidade de valor nem prova ZK de cobertura. |
| **Centrifuge** | RWA / recebíveis | Securitização tokenizada aberta. | Diferencial = confidencialidade + gate de saúde. |
| **Credix, Clearpool, TrueFi** | Private credit on-chain | Pools de crédito privado, KYC de tomador. | Termos/valores visíveis; sem anti-limão por ZK. |
| **Bancos / factoring** | Off-chain | Crédito com colateral e disclosure total. | Status quo que a ideia ataca. |
| **Aztec / DeFi privado** | Privacidade genérica | Privacidade de transação. | Não é captação com prova de solvência como filtro. |

**Leitura:** private credit on-chain é categoria **real e crescente**. Ninguém combina **valor oculto + prova de saúde como gate anti-limão**. Diferencial conceitual **forte**.

### 2.2 Viabilidade de negócio

- **TAM:** grande — crédito PME é trilhões; RWA on-chain tese quente.
- **Monetização:** fee por captação/volume; escalável se houver fluxo.
- **Dor real e cara:** vazamento de info ao captar tem custo mensurável. Anti-limão por ZK é economicamente sólido.
- **Barreiras:**
  - **Oráculo/fonte de dados** é o calcanhar. ZK prova a conta, não a verdade dos inputs → atestação assinada por fonte não controlada. Em produção = Open Finance/auditor/bureau, integração pesada por jurisdição.
  - **Completude:** provar um número real não prova ausência de dívida escondida.
  - **Enforcement/default:** empresa off-chain que não paga → cobrança fora da cadeia. Contrato não resolve inadimplência real. **Furo comercial mais grave.**
  - **Regulatório:** oferta de dívida = securities law. Sério.
- **Fosso:** conceito + rede de provedores de atestação + reputação on-chain. **Bom se atingir escala.**

**Veredito:** a **mais ambiciosa e maior teto**, mas depende de resolver oráculo, enforcement e regulatório — nenhum trivial. Ótima tese, produto difícil.

### 2.3 Forças e fraquezas

**Forças**
- ZK genuinamente **load-bearing** em dois pontos. Não removível.
- Narrativa anti-limão forte p/ pitch.
- Mercado grande, dor cara, privacidade seletiva bem pensada.

**Fraquezas**
- **Escopo MVP grande:** escrow + 2 circuitos + oráculo mock + frontend. Risco de não terminar até 3 jul.
- Depende de oráculo — parte mais frágil, mais "explicada no README" que demonstrada.
- Enforcement de default fora da cadeia limita o "trustless".
- Regulatório securities pesado.

**Blindar:** cortar escopo agressivo — oráculo mock minimalista, focar demo nos **dois circuitos ZK**, narrar enforcement/regulatório como roadmap.

---

## 3. confidential-debt-raise (on-chain)

### 3.1 Concorrentes

| Concorrente | Categoria | O que cobre | Gap que a ideia explora |
| --- | --- | --- | --- |
| **Maple Finance** | Crédito institucional | Empréstimo on-chain. | Transparente; aqui valor privado + saúde por ZK. |
| **Blend (Stellar)** | Lending pool nativo Stellar | Empréstimo colateralizado aberto. | Sem captação de valor oculto nem prova de saúde. Vizinho direto no ecossistema. |
| **DAO treasury tooling (Aragon)** | Governança de tesouraria | Gestão de fundos on-chain. | Não faz captação de dívida privada com prova. |
| **Aztec / DeFi privado** | Privacidade genérica | Privacidade de transação. | Não é captação com prova de solvência. |

**Leitura:** mercado **menor e mais jovem** que o off-chain, mas **muito mais crível tecnicamente** — dado já on-chain, oráculo dissolve.

### 3.2 Viabilidade de negócio

- **TAM:** menor — protocolos/DAOs na Stellar que precisam de runway. Nicho hoje.
- **Vantagem-chave:** **oráculo dissolvido** — dado nativamente verificável mata *garbage-in* e a integração mais frágil da irmã.
- **Reputação nativa:** histórico de repagamento on-chain habilita crédito por reputação sem KYC. Fosso composável.
- **Fluxo 100% on-chain:** sem leg fiat → sem money transmitter, sem anchor, fácil de demonstrar.
- **Tese de token forte:** proteger valuation ao vivo contra distress signaling é dor genuína e específica.
- **Contra:** parte da comunidade crypto **espera transparência** → pitch precisa enquadrar como anti-distress. Mercado endereçável raso hoje.

**Veredito:** **melhor relação viabilidade/esforço**. Menor teto que a off-chain, muito mais executável e defensável.

### 3.3 Forças e fraquezas

**Forças**
- **Oráculo dissolvido** → corta subsistema mais arriscado; MVP menor.
- ZK load-bearing ancorado em estado real → crível p/ juízes.
- Fluxo 100% on-chain → demo limpa.
- **Dogfooding Stellar** (Blend, DEXs) → agrada juízes.
- Reputação/enforcement mais naturais.

**Fraquezas**
- **Métrica de capacidade on-chain indefinida** (tesouraria? TVL? receita?) — coração do circuito, em aberto. Fechar já.
- Mercado pequeno hoje → tese de longo prazo mais fraca.
- Enforcement de default ainda aberto no contrato.
- Atrito cultural "privacidade em crypto".

**Blindar:** fechar métrica rápido (ex.: obrigação ≤ k·tesouraria líquida verificável), reusar máximo do Private Payments PoC.

---

## 4. zk-bounty

### 4.1 Concorrentes

| Concorrente | Categoria | O que cobre | Gap que a ideia explora |
| --- | --- | --- | --- |
| **HackerOne / Bugcrowd** | Bug bounty centralizado | Triagem, mediação humana, taxas altas. | Centralizado, sem confiança matemática nem escrow trustless. |
| **Immunefi** | Bug bounty Web3 | Líder DeFi; triagem intermediada, PoC manual. | Ainda depende de avaliação humana; sem prova ZK de disclosure. |
| **Hats Finance** | Bounty on-chain | Escrow/vaults on-chain, pagamento programático. | **Concorrente mais direto**: já faz escrow on-chain, mas sem prova ZK do exploit. |
| **Code4rena / Sherlock / Cantina** | Contest de auditoria | Competição paga de achados, julgada por humanos. | Modelo de contest, não disclosure privado provado por ZK. |
| **"Proof of exploit" (RISC Zero / zkVM PoCs)** | Research | Provar execução de exploit via zkVM. | Base técnica emergente; ninguém entregou produto de bounty sobre isso. |

**Leitura:** bounty on-chain **já existe** (Hats, Immunefi vaults). Diferencial = **prova ZK de que o exploit é válido sem revelá-lo** antes do pagamento travado. Recorte novo, mas tecnicamente o mais audacioso.

### 4.2 Viabilidade de negócio

- **Dor real:** desconfiança bidirecional (hacker teme não receber; empresa teme relatório falso). Legítima e conhecida.
- **Monetização:** fee por bounty intermediado; two-sided marketplace com **cold start** clássico (precisa de empresas E hackers ao mesmo tempo).
- **O gap central:** o MVP prova um **predicado fixo toy** ("conheço x que leva ao estado de erro Y"). Provar exploit de **software arbitrário real** exige expressar o alvo como circuito/zkVM — problema **essencialmente não resolvido em escala prática**. A distância entre a demo e o produto real é o maior risco de tese.
- **Problema do ovo-e-galinha:** para definir a "condição de falha" como circuito, o autor do desafio precisa **formalizar o que é a vulnerabilidade** — trivial no toy, quase impossível para bugs desconhecidos em código real. Se soubesse formalizar a falha, já conheceria o bug.
- **Nicho viável de verdade:** **smart contracts** (Soroban/EVM) — invariantes formalizáveis ("prove que consegue quebrar o invariante de saldo") via zkVM/execução verificável. É aí que o produto real pode existir; software genérico não.
- **Regulatório:** leve comparado às ideias de dívida.

**Veredito:** dor legítima, mas **viabilidade de produto real limitada ao subdomínio de invariantes formalizáveis** (contratos). Como produto genérico de bounty, o ZK esbarra em problema em aberto da área.

### 4.3 Forças e fraquezas

**Forças**
- ZK inequivocamente **load-bearing** e fácil de explicar (provar sem revelar).
- **Demo limpa e narrativa forte** — escrow + prova + pagamento em 3 min. Ótimo p/ juízes.
- Escopo MVP contido; toca Stellar nativamente (escrow + verifier).
- Tema segurança/hacking ético atrai atenção.

**Fraquezas**
- **Gap MVP↔produto enorme:** circuito toy demonstra a lógica, mas não o caso real (exploit de software arbitrário). Fácil de um juiz atento questionar.
- **Ovo-e-galinha** na definição do predicado de falha.
- Two-sided cold start; Hats/Immunefi já ocupam bounty on-chain.
- Revelação do exploit pós-pagamento ainda depende de etapa off-chain / confiança residual (§9 do IDEA).

**Blindar:** posicionar explicitamente no nicho de **invariantes de smart contract** (prova de quebra de invariante Soroban), não "bug de qualquer software". Assim MVP e produto real ficam na mesma trilha e o gap some.

---

## 5. Cruzamento Direto

| Critério | compliance-zk | offchain-debt | onchain-debt | zk-bounty |
| --- | --- | --- | --- | --- |
| ZK realmente essencial | ⚠️ opcional | ✅ sim | ✅ sim | ✅ sim |
| Facilidade de terminar até 3 jul | ✅ alta | ❌ baixa | 🟡 média | ✅ alta |
| Impressiona juízes de ZK | 🟡 médio | ✅ alto | ✅ alto | ✅ alto |
| Aderência ao pitch da Stellar | ✅ pagamentos | 🟡 cross-border | ✅ dogfooding DeFi | ✅ escrow/pagamento |
| Diferenciação vs. concorrentes | ❌ fraca | ✅ forte | ✅ forte | 🟡 média (Hats existe) |
| Teto de mercado (negócio) | 🟡 médio | ✅ alto | 🟡 nicho | 🟡 médio |
| Clareza da demo | ✅ alta | 🟡 média | ✅ alta | ✅ alta |
| Risco de execução | ✅ baixo | ❌ alto | 🟡 médio | ✅ baixo |
| Gap MVP↔produto real | 🟡 médio | 🟡 médio | ✅ pequeno | ❌ grande |

---

## 6. Recomendação p/ Hackathon

Critério do hackathon = **ZK load-bearing + toca Stellar + demo em 2–3 min + repo claro**, com ~3 dias.

1. **Aposta principal: `confidential-debt-raise` (on-chain).** Melhor equilíbrio: ZK essencial, oráculo dissolvido (menos risco), fluxo 100% on-chain (demo limpa), dogfooding Stellar, diferenciação forte, gap MVP↔produto pequeno. Ação imediata: **fechar a métrica de capacidade** e reusar o Private Payments PoC.

2. **Melhor pura jogada de hackathon: `zk-bounty`.** Se o objetivo é **maximizar chance de ganhar o hackathon** (não construir empresa), é a mais forte: ZK inequívoco, demo cristalina, escopo pequeno, narrativa que vende sozinha. Custo: viabilidade de produto real limitada — bom para prêmio, fraco para virar negócio. **Blindar posicionando no nicho de invariantes de smart contract.**

3. **Maior teto de negócio, maior risco: `offchain-debt`.** Tese comercial mais forte, mas escopo grande + oráculo frágil ameaçam o deadline. Só com corte de escopo agressivo.

4. **Plano B seguro: `compliance-zk`.** Menor risco de execução e narrativa quente, mas **precisa tornar o ZK não-removível** antes de ser competitivo num hackathon de ZK.

**Nota transversal:** as quatro compartilham o mesmo esqueleto técnico (contrato Soroban de escrow + verifier Groth16 + circuito de membership/threshold/witness, base = Stellar Private Payments PoC). Isso **reduz o custo de trocar de ideia** — infra reaproveitável.

**Decisão sugerida em uma linha:** se a meta é **ganhar o prêmio**, `zk-bounty` (nicho smart-contract) ou `onchain-debt`; se a meta é **germe de startup**, `onchain-debt` com `offchain-debt` como evolução narrada.
