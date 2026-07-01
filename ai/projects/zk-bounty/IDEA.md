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

- Empresas de tecnologia que precisam de segurança contínua e querem reduzir fraude em disclosures.
- Protocolos DeFi e DAOs que lidam com bugs críticos e precisam de fluxo confiável de recompensa.
- Pesquisadores de segurança, auditors e hackers éticos que valorizam privacidade e garantia de pagamento.

## 8. Diferenciais

- Confiança matemática, não confiança institucional.
- Privacidade do exploit até o pagamento estar garantido.
- Pagamento mais rápido e com menos fricção do que modelos tradicionais.
- Incentivo a bug reports honestos e à cultura de hacking ético.
- Arquitetura aberta e composável para futuras integrações com auditorias e protocolos de segurança.

## 9. Modelo de Produto

O produto opera como um protocolo de escrow com revelação gradual:

1. a empresa publica o desafio e deposita a recompensa;
2. o hacker gera e envia a prova ZK;
3. o contrato verifica a prova e trava o valor;
4. o pesquisador revela o exploit de forma privada ou criptografada;
5. a empresa confirma e libera o saque ou o contrato faz a liberação por timeout.

Esse modelo cria uma ponte entre segurança, privacidade e pagamentos on-chain.

## 10. Escopo Técnico do MVP

Para o hackathon, o escopo deve ser simples e bem demonstrável.

### Componente 1 — Contrato Soroban

Implementar um contrato com funções como:

- criar_bounty()
- submeter_prova()
- liberar_pagamento()
- cancelar_bounty()

### Componente 2 — Circuito ZK

Usar um circuito minimalista, por exemplo:

- prova de conhecimento de um witness secreto;
- prova de que esse witness satisfaz uma condição fixa de falha;
- publicação de um commitment público para o witness ou para o estado de erro.

Para o MVP, a implementação pode usar Noir, Circom ou um fluxo de prova simples com foco em demonstrar a lógica de verificação.

### Componente 3 — Frontend

Uma interface simples com:

- painel da empresa para criar bounty e depositar fundos;
- painel do hacker para subir a prova gerada localmente;
- status do bounty: aberto, verificado, liberado ou vencido.

## 11. Pitch (Roteiro de 3 Minutos)

- Gancho: “Hoje, descobrir uma falha crítica pode ser arriscado: o hacker pode não ser pago e a empresa pode ignorar o relatório.”
- Problema: desconfiança mútua, burocracia, taxas e atrasos em plataformas centralizadas.
- Solução: um protocolo que usa ZK para provar a existência da vulnerabilidade sem expor o exploit, e Soroban para garantir a recompensa.
- Demonstração: contrato verificando a prova, fundo travado e pagamento liberado.
- Visão: transformar bug bounties em um fluxo trustless, privado e escalável.

## 12. Por Que Esta Ideia É Boa Para o Hackathon

A proposta tem boa aderência ao desafio porque:

- o ZK é essencial e não decorativo;
- o fluxo toca Stellar de forma nativa via Soroban;
- o MVP é viável em poucas semanas;
- a demo é clara e fácil de explicar.

A aposta certa é mostrar um fluxo simples, robusto e convincente, em vez de tentar resolver o problema completo de auditoria de software arbitrário na primeira versão.

## 13. Próximos Passos

1. Definir o circuito ZK mínimo e a função de verificação.
2. Implementar o contrato Soroban com o fluxo de escrow e liberação.
3. Criar uma demo frontend com duas telas: empresa e hacker.
4. Preparar o pitch focalizado em confiança matemática e pagamentos instantâneos.
5. Evoluir o projeto para suporte a desafios mais reais e maior complexidade de provas.
