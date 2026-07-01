# ZK-Bounty: Descoberta Verificável de Vulnerabilidades em Soroban com Reputação On-Chain

## 1. Visão Geral

O **ZK-Bounty** é um protocolo de segurança on-chain especificamente para **Soroban smart contracts**, onde pesquisadores de segurança descobrem vulnerabilidades provando criptograficamente que conhecem um input que viola uma **invariante do contrato** — sem revelar o exploit até que a recompensa seja garantida e a reputação on-chain seja creditada.

A proposta transforma duas dinâmicas do mercado de segurança:

1. **De desconfiança para confiança matemática:** O pesquisador prova via ZK que conhece uma entrada que quebra uma invariante pública do contrato. A prova não expõe a entrada, mas é verificável on-chain e impossível forjar.
2. **De one-shot para reputação duradoura:** Cada descoberta bem-sucedida incrementa um score on-chain do pesquisador (`reputation_score = vulnerabilidades_encontradas + taxa_de_acurácia + severidade_média`). O score não é vanity — impacta futuras recompensas e acesso a bugbounties premium.

Todo o fluxo acontece na Stellar: o contrato escrow em Soroban custodia USDC, verifica a prova ZK, libera pagamento e registra reputação no próprio ledger de Soroban.

## 2. Dor Que O MVP Resolve

Projetos Soroban enfrentam um dilema crítico:

- **Descobrir antes de deploy:** Auditorias caras, tempo limitado, cobertura incompleta.
- **Descobrir após deploy:** Vulnerabilidades vivas no mainnet afetam usuários; descobridor terá incentivo para explorar, não reportar (moral hazard).
- **Mediação centralizada:** Immunefi e HackerOne ficam no meio, cobram 10%+, avaliam relatórios manualmente, podem recusar pagamento sem justificativa matemática.

O **ZK-Bounty** resolve três problemas simultâneos:

1. **Segurança permanente:** Qualquer pessoa pode descobrir vulnerabilidades pós-deploy, com garantia criptográfica de recompensa (não depende de empresa aceitar).
2. **Incentivo correto:** Pesquisador prova que conhece o bug (e o know-how vale dinheiro). Reputação on-chain funciona como título — pesquisador com score alto ganha acesso a auditorias premium e recompensas maiores.
3. **Trustless e barato:** Soroban custodia o dinheiro, ZK verifica a prova, contrato libera USDC automaticamente. Sem mediador.

O diferencial é que **ZK prova o conhecimento, não o relatório**. Isso inverte a curva de incentivos: descobrir passa a compensar mais que explorar.

## 3. Como Funciona

**Fluxo com reputação on-chain:**

1. **Criação do bounty:** Protocolo/desenvolvedor Soroban publica uma **invariante pública** (p.ex., `total_supply() >= sum_of_balances()` ou `nonce > 0 para transações válidas`). Deposita USDC como recompensa no contrato escrow Soroban. A invariante é **testável e verificável** — não é uma description subjetiva.

2. **Descoberta:** Pesquisador encontra um input/trace que viola a invariante. Gera uma **prova ZK off-chain** mostrando que conhece esse input sem revelar qual é:
   - Prova: `∃ input : invariante(contrato_soroban, input) = false`
   - Isso é concreto: o pesquisador está provando que a invariante específica daquele contrato Soroban é violável.

3. **Submissão e Verificação:** Pesquisador envia a prova para o contrato escrow Soroban. O contrato verifica a prova usando as host functions nativas de BN254 + Poseidon (Protocol 26 / Yardstick). Se válida:
   - Contrato trava o pagamento USDC para o endereço do pesquisador.
   - Incrementa `reputation_score` on-chain do pesquisador.
   - Registra descoberta no histórico público.

4. **Divulgação Responsável:** Pesquisador revela o exploit de forma privada ao desenvolvedor. Desenvolvedor tem N dias para fazer patch. Após patch + verificação:
   - Pesquisador pode gerar uma **"re-prova"** mostrando que a invariante agora passa.
   - Contrato marca descoberta como `PATCHED_VERIFIED` — novo milestone de reputação.

5. **Liberação e Reputação:** Pagamento é liberado automaticamente (sem mediação). Reputação do pesquisador passa a influenciar:
   - Acesso a bounties premium (maiores recompensas, contratos mais críticos).
   - Multiplicador de recompensa: pesquisador com score 100 recebe +20% em recompensas futuras.
   - Visibilidade pública: leaderboard on-chain.

## 4. Onde Entra ZK (Load-Bearing)

O ZK é **obrigatório** em três pontos — não é decorativo:

### 4.1 Prova de Descoberta
**Problema:** Pesquisador diz "encontrei bug" — como provar sem revelar?
**Solução ZK:** Pesquisador prova `∃ input : invariante(input) = false` sem revelar qual é o input.

- Circuito ZK recebe como witness privado: `input_secreto, estado_inicial, estado_final`
- Circuito prova (publicamente):
  - Executa o contrato Soroban simulado com `input_secreto`
  - Verifica que `invariante(estado_final) = false`
  - Publica um **commitment Poseidon** do input (para impedir re-uso)
- Output: prova Groth16 verificável on-chain, sem expor `input_secreto`

**Por que ZK e não hash simples?** Porque o contrato precisa verificar que a execução ocorreu corretamente, não só que um hash bate. Com Soroban como sistema-alvo, a prova refere-se diretamente à máquina de estados do contrato.

### 4.2 Re-prova de Patch
**Problema:** Como verificar que o patch realmente consertou a falha?
**Solução ZK:** Pesquisador prova que o mesmo input agora passa na invariante:
- `∃ input : invariante_novo(input) = true AND input_commitment == commitment_antigo`
- A re-prova usa o mesmo commit do input original, vinculando a descoberta ao patch verificado.

### 4.3 Reputação Tokenizada (Opcional mas Potente)
**Evolução:** Score do pesquisador pode virar token (SBT / soulbound token) em Soroban:
- Mercado secundário de reputação: projetos competem por pesquisadores high-score
- Staking: pesquisador coloca reputação em risco para validar bounties de terceiros (oráculo descentralizado)

Sem ZK, seria impossível vincular reputação de forma criptográfica ao histórico de descobertas.

## 5. Por Que Stellar (Não Genérico)

ZK-Bounty não é um bounty genérico em Stellar — é um **bounty específico para Soroban**. Isso muda tudo:

- **Alvo concreto:** A invariante é um predicado Soroban. O contrato escrow é um contrato Soroban. A prova refere-se diretamente à máquina de estados de Soroban. Não é "bug em qualquer coisa", é "bug neste contrato nesta rede".

- **Host functions nativas:** Stellar Protocol 26 (Yardstick) adicionou 9 host functions BN254 especificamente para tornar verificação de proofs barata em Soroban. ZK-Bounty **dogfoods** essas primitivas — não é demo teórica, é demo que só é viável/barata em Stellar hoje.

- **Ecossistema Soroban:** Todo protocolo novo em Soroban herda risco de bug. ZK-Bounty oferece uma **infraestrutura pública de segurança permanente**. Não é terceirizado para Immunefi; fica na rede.

- **USDC e XLM:** Recompensas em stablecoin (USDC) tornam o incentivo real. Reputação é registrada em XLM/Soroban ledger.

- **Custo:** Verificar uma prova Groth16 em Soroban custa ~1-2 XLM. Em Ethereum custaría 100x mais. Isso torna o modelo economicamente viável.

**Portanto:** ZK-Bounty não usa Stellar como blockchain genérico; **Stellar é o elo crítico** — sem as host functions de Protocol 26 e Soroban, a verificação seria inviável ou muito cara.

## 6. Panorama Competitivo e Posicionamento

| Competidor | Modelo | Diferença do ZK-Bounty |
|---|---|---|
| **HackerOne / Bugcrowd** | Plataforma centralizada, curadoria humana, 10-15% de taxa. | Centralizadas; dependem de mediador. ZK-Bounty é trustless, paga automaticamente, sem taxa. |
| **Immunefi** | Especializada em Web3, depende de triagem e avaliação. | Boa em Web3, mas avaliação é manual. ZK-Bounty: prova é matemática, sem avaliador. |
| **Cantina** | Auditorias compartilhadas, menos profundo que auditorias full. | Modelo de auditoria, não contínuo. ZK-Bounty: segurança permanente pós-deploy. |
| **OpenZeppelin Audit** | Auditorias profundas off-chain antes de deploy. | Pré-deploy, não contínuo. ZK-Bounty: pós-deploy, contínuo, incentiva descoberta. |

O **posicionamento defensável**:

**"Não é mais uma plataforma de bounty — é uma camada de segurança verificável para Soroban onde descobertas são comprovadas por ZK, recompensas são automáticas, e reputação de pesquisador é on-chain e duradoura."**

Diferenciais:
- **Trustless:** Sem mediador; contrato verifica prova e paga automaticamente.
- **Específico para Soroban:** Invariantes são predicados Soroban reais, não genéricas.
- **Reputação on-chain:** Score do pesquisador é ativo público com impacto econômico.
- **Segurança contínua:** Não é pré-deploy; é pós-deploy permanente.

## 7. Clientes e Usuários Alvo

- **Desenvolvedores de protocolos Soroban:** Lending pools, DEXs, stablecoin emissores que já estão na Stellar e querem segurança pós-deploy.
- **DAOs em Stellar:** Governança e tesouraria precisam de contratos seguros.
- **Pesquisadores de segurança especializados em ZK/Soroban:** Novo mercado de trabalho — descobrir bugs com provável (não explorar para si).
- **Auditores e security firms:** Podem usar ZK-Bounty como camada complementar após auditoria (sempre existem bugs não descobertos).

Mercado menor que Immunefi, porém **muito mais consciente de ZK** e **dispostos a testar primitivasnovas**.

## 8. Diferenciais (Concretos)

- **Confiança matemática, não curadoria:** Prova é verificável; mediador é máquina, não humano.
- **Reputação on-chain com impacto econômico:** Score do pesquisador influencia recompensas futuras, acesso a bounties premium, possibilidade de staking em validacredibilidade de terceiros.
- **Specificity a Soroban:** Não é bug genérico; é bug neste contrato nesta rede. Prova refere-se à máquina de estados Soroban.
- **Pagamento automático:** Sem esperar por avaliação humana. Prova válida = USDC liberado.
- **Custo baixo:** Host functions Protocol 26 tornam verificação barata (~1-2 XLM) vs Ethereum (100x mais caro).
- **Dogfooding Stellar:** Usa Host functions de Protocol 26 que Stellar adicionou para ZK.

## 9. Modelo de Reputacão On-Chain

A reputação não é cosmética; tem impacto real:

### Scoring
```
reputation_score = 
  + descobertas_válidas (cada uma +10)
  - falsos_positivos (cada um -5)
  + severidade_média (0-50 por descoberta)
  + bonus_patch_verificado (+5 por patch confirmado)
```

### Impacto Econômico
1. **Acesso:** Score < 20 = bounties públicos abertos. Score >= 50 = convites para bounties premium (maiores recompensas).
2. **Multiplicador de recompensa:** `recompensa_final = base_recompensa × (1 + score_percentual / 1000)` — score 100 = +10% em futuras recompensas.
3. **Staking (evolução):** Pesquisador pode colocar reputação em risco para validar bounties de terceiros, ganhando taxa (oráculo descentralizado de severidade).

### Token
Evolução: reputação pode ser tokenizada como **Soul Bound Token (SBT)** em Soroban — inséparável do pesquisador, com histórico completo on-chain.

## 10. Escopo Técnico Do MVP

O MVP demonstra um fluxo **completo e convincente** em 4 componentes atrelados:

### Componente 1 — Contrato Escrow Soroban

Implementar:
- `create_bounty(protocol_id, invariant_commitment, base_reward_usdc, deadline)`
- `submit_proof(bounty_id, proof_groth16, commitment_input)` — verifica prova e trava recompensa
- `claim_reward()` — pesquisador saca USDC
- `record_reputation(researcher, proof_valid, severity)` — incrementa score on-chain
- `get_researcher_score(address)` — consulta reputação pública

Evento: `ProofValid`, `ReputationUpdated`, `RewardClaimed`

### Componente 2 — Circuito ZK (Noir / Circom)

**Objetivo:** Prova que pesquisador conhece um input que viola invariante **sem revelar o input**.

**Inputs privados (witness):**
- `input_vec: [u32; 32]` — entrada que viola invariante
- `initial_state: [u32; 8]` — estado inicial do contrato
- `secret_nonce: u32` — salt para commitment

**Outputs públicos:**
- `commitment_input: Field` — `hash(input_vec + secret_nonce)` via Poseidon
- `proof_valid: bool` — sempre true se prova passa

**Lógica do circuito:**
```
1. Valida que commitment_input == Poseidon(input_vec, secret_nonce)
2. Executa simulado do contrato Soroban:
   - Estado s0 = initial_state
   - Estado s1 = ContractStep(s0, input_vec)  
   - Valida que Invariant(s1) == false
3. Publica commitment_input
```

Usrar **Noir** pelo tempo de dev. Path alternativo: **Circom** se tempo permitir (mais barato on-chain).

### Componente 3 — Contrato "Dummy" Soroban com Invariante

Para MVP, criar um contrato Soroban simples com invariante **desafiante mas provável**:

**Exemplo:** Contador com overflow
```rust
pub fn increment(self, amount: u32) {
    self.counter = self.counter + amount;  // Bug: sem verificação de overflow
}

// Invariante publicada: counter <= MAX_COUNTER
// Pesquisador prova: ∃ amount tal que increment(amount) viola invariante
```

**Outro exemplo:** Token com double-spend
```rust
pub fn transfer(from: Address, to: Address, amount: i128) {
    let balance_from = get_balance(from);
    require(balance_from >= amount);
    set_balance(from, balance_from - amount);  // Bug: sem atualizar total_supply
    set_balance(to, get_balance(to) + amount);
}

// Invariante: sum_of_balances == total_supply
```

### Componente 4 — Frontend + Demo

**Flows:**
1. **Criar Bounty:** Desenvolvedor conecta wallet, publica invariante (string legível), deposita 100 USDC.
2. **Pesquisador Gera Prova:** Bot/CLI que:
   - Executa contrato Soroban localmente
   - Encontra input que viola invariante
   - Gera prova Noir/Circom
   - Envia para contract
3. **Verificação:** Contrato recebe prova, verifica, registra `ProofValid`, trava 100 USDC.
4. **Reclamação:** Pesquisador clica "Claim" — USDC é transferido, score incrementa de 0 para 10.
5. **View Pública:** Leaderboard mostra "Researcher X discovered 5 bugs, reputation score: 45".

## 11. Pitch (Roteiro de 3 Minutos)

**Gancho:** "Protocolos Soroban têm bugs. Auditorias encontram alguns. Mas a maioria vive no mainnet. Pesquisadores evitam reportar porque não há garantia de pagamento. Empresas evitam investir em bounties porque têm fraude."

**Problema:** Desconfiança mútua em segurança de Soroban. Plataformas centralizadas cobram 10-15%, avaliam manualmente. 

**Solução:** ZK-Bounty: pesquisador prova via ZK que conhece um input que quebra uma invariante do contrato. Prova é verificável on-chain. Pagamento é automático. Reputação do pesquisador fica pública e duradoura.

**Demo:** 
1. Mostrar contrato Soroban com invariante: `balance == sum_of_all_balances`.
2. Pesquisador encontra input que viola (double-spend).
3. Gera prova ZK off-chain (30s).
4. Envia para contrato Soroban.
5. Contrato verifica prova (~0.5s, custa 1 XLM).
6. USDC é travado e marcado para pesquisador.
7. Mostrar leaderboard: pesquisador agora tem score 10.

**Visão:** "Segurança de Soroban é uma camada pública, confiável e econômica. Pesquisadores ganham reputação duradoura. Protocolos dormem melhor."

**Frase curta:**
**"Segurança verificável para Soroban: prove que encontrou um bug sem revelar qual, receba USDC automaticamente, ganhe reputação on-chain."**

## 12. Por Que Esta Ideia É Boa Para o Hackathon (vs. Payment Vault)

**Vulnerabilidades do Payment Vault que ZK-Bounty resolve:**

| Aspecto | Payment Vault | ZK-Bounty (novo) |
|--------|---|---|
| **Conceitual vs. Real** | Genérico: "wallets com políticas". Competidores já fazem. | Real: "bugs em Soroban contracts". Alvo concreto e urgenté. |
| **ZK É Essential?** | Superficial. Whitelist Merkle é bonita, não obrigatória. | Load-bearing em 3 pontos: prova descoberta, re-prova patch, rep on-chain. |
| **Stellar É Central?** | Genérico: qualquer blockchain roda isso. | Essencial: Protocol 26 host functions barateiam verificação. Só viável em Stellar hoje. |
| **Demo Natural** | Entédio: "agente tenta transferir, falha, política valida". | Convincente: encontrar bug real, ZK prova, prova verifica no-chain, USDC libera. |
| **Escopo Fechado** | Confuso: ICP misto (DAO, agentes, fintechs). | Cristalino: pesquisadores, protocolos Soroban, segurança. |
| **Diferencial** | Moderado: "open-source policy vault". | Muito forte: "ZK prova bug real, reputação on-chain, segurança permanente". |
| **Oportunidade de Rede** | Nula: cada vault é isolado. | Forte: toda descoberta alimenta leaderboard; reputation vira ativo com impacto econômico. |

**Por que ZK-Bounty vence agora:**

1. ✅ **Real, não conceitual:** Pesquisador encontra bug em contrato Soroban — não é teoria.
2. ✅ **ZK indispensável:** Sem ZK, pesquisador teria que revelar o bug (perde direito de discovery). Com ZK, prova sem revelar.
3. ✅ **Stellar não é genérico:** Só funciona assim em Stellar porque Protocol 26 tem primitivas certas + custo baixo.
4. ✅ **Demo é convincente:** 1 minuto mostra tudo; ZK claramente não é cosmético.
5. ✅ **Impacto duradouro:** Não é um evento (comme bounty); é uma infraestrutura de segurança público (como qu Blend ou Soroban specs).

## 13. Próximos Passos

**MVP (Semanas 1-2):**
1. Definir invariante simples + contrato Soroban dummy com bug.
2. Implementar circuito Noir (prova conhecimento input que viola invariante).
3. Implementar contrato escrow Soroban + verifier.
4. CLI que gera prova localmente e submete.
5. Leaderboard web simples (HTML + JS lendo blockchain).

**Demo (Semana 2.5):
- Vídeo 3 min:
  1. Mostrar contrato com bug (invariante).
  2. Rodar prova localmente (~10s de output).
  3. Enviar para Soroban (~5s de confirmação).
  4. Mostrar reputação incrementada no-chain.
  5. Öptimo: 2 ciclos (descoberta + patch verificado).

**Pós-Hackathon (Evolução):**
1. Re-prova de patch: pesquisador prova que invariante agora passa.
2. Governança de severidade: comunidade valida se severidade está correta (staking de reputação).
3. Mercado de reputacão: tokens de reputação podem ser emprestados/stakeados.
4. Integração com auditorias: audit reports linkam a ZK-Bounty (incentivo complementar).
5. Cross-protocol bounties: protocolo A aluga pesquisador com reputacão alta de protocolo B.
