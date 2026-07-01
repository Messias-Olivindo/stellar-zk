# ZK-Bounty - Pesquisa Externa e Implicacoes

## 1. Resumo executivo

Para a ZK-Bounty superar o Agent Payment Vault, a ideia precisa deixar de ser "uma plataforma de bug bounty com ZK" e virar algo mais preciso:

**um mercado de invariant bounties para contratos Soroban, onde o pesquisador prova em ZK que conhece um input privado que quebra uma invariante publica, o contrato Soroban verifica a prova, trava a recompensa em USDC e registra reputacao on-chain.**

Esse recorte cobre as ausencias principais:

- A demo fica concreta: encontrar um bug financeiro em um contrato Soroban vulneravel.
- O ZK fica indispensavel: sem ZK, o pesquisador teria que revelar o exploit antes de garantir pagamento.
- A Stellar fica central: o alvo e Soroban, a verificacao acontece em Soroban, a recompensa usa Stellar Asset Contract e a reputacao vive no ledger.
- A diferenciacao melhora: o projeto nao tenta substituir HackerOne ou Immunefi em todos os casos; ele automatiza a parte objetiva e verificavel de um bounty.

## 2. Fontes pesquisadas

- Stellar ZK Proofs docs: https://developers.stellar.org/docs/build/apps/zk
- Stellar Privacy docs: https://developers.stellar.org/docs/build/apps/privacy
- Stellar X-Ray Protocol 25: https://stellar.org/blog/developers/announcing-stellar-x-ray-protocol-25
- Stellar Yardstick Protocol 26: https://stellar.org/blog/foundation-news/stellar-yardstick-protocol-26-upgrade-guide
- Soroban Groth16 verifier example: https://github.com/stellar/soroban-examples/tree/main/groth16_verifier
- Stellar Private Payments by Nethermind: https://github.com/NethermindEth/stellar-private-payments
- Cheesecloth, ZK proofs of real-world vulnerabilities: https://arxiv.org/abs/2301.01321
- LLM-era invalid report research: https://arxiv.org/abs/2511.18608
- OSS maintainer bug bounty review research: https://arxiv.org/abs/2409.07670

## 3. O que a pesquisa muda na tese

### 3.1 Stellar tem base tecnica real para ZK, mas nao entrega o produto pronto

Os docs oficiais de ZK em Stellar dizem que X-Ray, Protocol 25, introduziu primitivas ZK-friendly como BN254 e Poseidon/Poseidon2. Eles tambem deixam claro que as primitivas sao building blocks: o desenvolvedor ainda precisa gerar provas off-chain e publicar contratos verificadores.

Yardstick, Protocol 26, adiciona novas host functions BN254 e move mais matematica pesada para a host layer. Isso fortalece o pitch de que a Stellar esta investindo em verificacao ZK dentro de Soroban.

Implicacao para a ZK-Bounty:

- E defensavel dizer que Stellar tem primitives adequadas para verificar provas em Soroban.
- E arriscado prometer custo fixo tipo "1 XLM" sem benchmark proprio.
- O README final deve mostrar custo real da demo, medido no ambiente usado.

### 3.2 Ja existe referencia tecnica de proof verification em Soroban

O repo `stellar/soroban-examples` tem um exemplo de contrato verificador Groth16. O projeto `stellar-private-payments` da Nethermind tambem usa Circom, Groth16 e Soroban, com provas geradas client-side.

Implicacao para a ZK-Bounty:

- O path mais pragmatico de hackathon e Circom + Groth16 + verifier Soroban.
- Noir pode ser mais expressivo, mas deve ser escolhido somente se o verifier UltraHonk em Soroban estiver funcionando rapido no ambiente do time.
- A demo deve reaproveitar o padrao: proof off-chain, public inputs on-chain, verifier contract, escrow contract.

### 3.3 Provar vulnerabilidades reais em ZK e uma area valida, mas dificil

O paper Cheesecloth mostra que faz sentido provar conhecimento de vulnerabilidades em ZK sem revelar o exploit. Ele tambem mostra a dificuldade: modelar software real em ZK escala mal e exige compiladores, IRs e otimizacoes.

Implicacao para a ZK-Bounty:

- A tese e intelectualmente forte e tem precedente academico.
- O MVP nao deve prometer "provar qualquer bug de qualquer contrato Soroban".
- O MVP deve provar uma classe especifica: quebra de invariante em um harness publico e pequeno.

O recorte correto e:

```text
Arbitrary bug bounty: fora do escopo do MVP.
Provable invariant bounty: dentro do escopo do MVP.
```

### 3.4 A dor de bug bounty e real, mas nao e so pagamento

Bug bounties sofrem com reports invalidos, duplicados, out-of-scope e subjetividade de triagem. Pesquisas recentes tambem apontam pressao crescente de reports gerados por IA e dificuldade de classificacao.

Implicacao para a ZK-Bounty:

- O projeto deve vender reducao de falsos positivos para propriedades objetivas, nao eliminacao total de triagem humana.
- A prova ZK deve ser um filtro criptografico: "existe um contraexemplo valido para esta invariante".
- Severidade, impacto de negocio e qualidade do patch ainda podem exigir julgamento humano.

### 3.5 Reputacao on-chain e boa, mas precisa ser conservadora

Reputacao em bug bounty ja influencia credibilidade. A diferenca da ZK-Bounty e tornar parte dessa reputacao objetiva: provas validas, bounties pagos, patch verificado, duplicatas bloqueadas por nullifier.

Implicacao para a ZK-Bounty:

- Nao vender reputacao como "token negociavel" no MVP.
- Vender como score nao-transferivel, derivado de descobertas verificadas.
- O score deve contar apenas eventos objetivos ou aceitos pelo dono do bounty.

## 4. Posicionamento recomendado

Frase curta:

**Provable bug bounties for Soroban invariants.**

Frase de pitch:

**ZK-Bounty lets a researcher prove they found a real invariant break in a Soroban contract without revealing the exploit first. Soroban verifies the proof, locks the USDC bounty, and records the discovery as on-chain security reputation.**

## 5. Claims que devem ser evitados

- "Prova qualquer vulnerabilidade": falso para o MVP.
- "Elimina toda mediacao": exagerado; elimina mediacao apenas para checks objetivos.
- "Pagamento automatico para qualquer bug": perigoso; melhor dizer que a prova trava ou reserva a recompensa.
- "Custo 1-2 XLM": so usar depois de benchmark proprio.
- "Substitui Immunefi/HackerOne": errado; complementa com uma categoria verificavel on-chain.
- "Reputacao negociavel": cria incentivos ruins; MVP deve usar reputacao nao-transferivel.

## 6. Insight criativo para a demo

Criar uma "Invariant Arena" para Soroban:

1. O projeto publica um contrato Soroban vulneravel e uma invariante objetiva.
2. O pesquisador roda um buscador local que encontra um input secreto que quebra a invariante.
3. O pesquisador gera uma prova ZK de que conhece esse input.
4. O contrato ZK-Bounty verifica a prova e trava a recompensa.
5. A UI mostra o status: `OPEN -> PROOF_VERIFIED -> REWARD_LOCKED -> PATCH_VERIFIED`.
6. A reputacao do pesquisador sobe somente depois da prova e do disclosure.

Essa demo e mais forte que uma transferencia bloqueada por policy, porque o juiz ve o problema nascer: um bug real, um exploit privado, uma prova verificavel e um pagamento em Soroban.
