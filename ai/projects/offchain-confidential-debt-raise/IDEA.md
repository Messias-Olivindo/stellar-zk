# Confidential Debt Raise: Captação de Dívida com Saúde Provada e Valor Privado na Stellar

> Projeto para o hackathon **Stellar Hacks: Real-World ZK**. Contexto do hackathon em `../../development/AGENT_CONTEXT.md`.
> Variante com **ICP = empresas off-chain (tradicionais)**. Ideia irmã (ICP on-chain) em `../confidential-debt-raise/`.

## 1. Visão Geral

O **Confidential Debt Raise** é uma plataforma onde uma empresa capta dívida sem revelar ao mercado o **valor que precisa** nem os **números crus do seu balanço**, mas provando por conhecimento zero que é **financeiramente saudável**.

A ideia central é simples: a empresa publica uma captação com juros e prazo públicos, compromete on-chain o valor-alvo de forma oculta, e anexa uma prova ZK de que sua dívida cabe dentro de um múltiplo seguro da sua capacidade de pagamento. Investidores aportam stablecoin em uma pool no contrato; quando a soma atinge o alvo oculto, a pool fecha sozinha sem nunca expor esse alvo.

O resultado é uma captação onde a **identidade e a saúde da empresa são provadas e públicas**, mas o **valor e os números financeiros permanecem privados**.

## 2. Dor Que O MVP Resolve

Empresas saudáveis que precisam de capital enfrentam um dilema: **captar obriga a vazar informação.**

- No **banco**, abrem todo o balanço para a contraparte, esperam semanas e oferecem colateral. O dado pode vazar e o banco mete spread.
- Em **captação aberta**, mercado, concorrentes, fornecedores e clientes veem quanto a empresa precisa e por quê.

Esse vazamento tem custo real mesmo para empresa boa: concorrente descobre a estratégia e o war chest, fornecedor sobe preço ao ver caixa, investidor de equity desconfia e derruba a valuation, cliente teme que a empresa vá quebrar. **Captar passa a ser lido como fraqueza, mesmo quando não é.**

O Confidential Debt Raise resolve essa dor separando o que precisa ser provado do que precisa ser escondido:

- A empresa **prova saúde** sem abrir os números.
- O **valor da captação fica oculto** do mercado.
- O investidor ainda **precifica o risco** a partir de uma prova matemática, não de fé.
- O capital chega **global e em stablecoin**, sem o gargalo bancário.

## 3. Como Funciona

```
1. Investidor ──USDC──► [POOL / escrow no contrato]      (on-chain)
2.                       pool enche, custódia no contrato
3. fecha (prova Σ ≥ X) ──USDC──► carteira da empresa ──off-ramp──► fiat
4. Empresa ──on-ramp──► USDC ──► contrato                (repagamento)
5. Investidor ◄──USDC── claim pro-rata + juros (nullifier)
```

1. A empresa define a necessidade de capital `X` e a **compromete on-chain** (commitment Poseidon/Pedersen) — `X` fica oculto.
2. A empresa publica **termos públicos**: juros e prazo.
3. A empresa gera uma **prova ZK de cobertura** (saúde financeira), que funciona como gate de entrada.
4. Investidores aportam USDC na pool custodiada pelo contrato.
5. O contrato fecha a pool quando uma prova demonstra `Σ depósitos ≥ X` **sem revelar `X`**.
6. A empresa saca o capital e, ao longo do prazo, repaga principal e juros; investidores resgatam pro-rata.

## 4. Onde Entra ZK

O ZK aqui não é palavra de efeito — ele é **load-bearing** em dois pontos centrais:

- **Fechamento por threshold:** prova que `Σ depósitos ≥ X` com `X` oculto, fazendo a pool travar sozinha sem vazar o alvo.
- **Prova de cobertura (saúde):** prova que `dívida ≤ k · capacidade de pagamento` sem revelar os números crus. É a peça que sustenta toda a tese — sem ela, a privacidade seria apenas opacidade.

A prova de cobertura é exibida ao mercado como **banda ou rating** (por exemplo, "rating B = cobertura entre 2x e 3x"), nunca como ratio exato — senão seria possível fazer engenharia reversa do valor oculto.

## 5. Modelo De Privacidade

A privacidade é **seletiva, não opaca**. O que cada parte vê:

| Público / mercado / concorrente | Investidor |
| --- | --- |
| identidade da empresa, rating/banda, juros, prazo | tudo isso + a prova ZK de cobertura |
| **não vê:** valor exato `X`, números crus do balanço | **não vê:** valor cru `X` (recebe apenas a banda) |

Dois cuidados de design:

- Mostrar **banda/rating**, nunca o ratio exato, para impedir engenharia reversa do valor.
- Entregar ao investidor um **bound**, nunca o `X` cru — senão N investidores em colusão reconstruiriam o segredo.

## 6. O Problema Dos Limões E O Anti-Limão

Esconder informação, por si só, **atrai as piores empresas**. Empresa saudável capta barato e transparente no banco e tem pouco motivo para usar uma plataforma opaca; quem mais quer esconder é quem tem o pior número para esconder. Isso é o clássico **mercado de limões**: os bons saem, sobram os podres, o investidor desconfia de todos, exige juros altíssimo, e o mercado colapsa.

O ZK quebra essa espiral porque a **prova de cobertura é um sinal custoso de imitar**: uma empresa sobre-endividada simplesmente **não consegue gerar** uma prova de cobertura saudável, pois os números não fecham. A prova vira um **porteiro automático** — só passa quem é saudável.

Com isso, a lógica se inverte em **ciclo virtuoso**: o gate barra os podres, o investidor confia, os juros caem, e a empresa boa passa a achar vantajoso entrar. A plataforma deixa de ser feira de limões e vira **mercado de qualidade**.

## 7. Fonte De Dados E Confiança

ZK prova a **conta**, não a **verdade dos inputs**. Se a empresa mente os números, a prova sai correta e falsa (*garbage-in*). Por isso o dado financeiro precisa vir **assinado por uma fonte que a empresa não controla**, e o circuito prova duas coisas juntas: a assinatura válida dessa fonte e a conta de cobertura sobre o dado assinado.

- **No MVP:** um **oráculo mock assinado** (serviço com par de chaves de desenvolvimento) assina os financeiros; o circuito verifica a assinatura de verdade. Usa a mesma interface dos provedores reais, então não gera retrabalho.
- **Em produção:** uma **camada de atestação agnóstica**, não uma fonte única. A abstração é o **provedor de atestação** — qualquer entidade que assina uma declaração sobre um **schema canônico** (`{passivos, fluxo_de_caixa, período, moeda, ...}`) com chave pública registrada. Cada fonte (Open Finance, PSD2, zkTLS de um banco, adquirente, auditor, mock) é apenas um **adaptador** que mapeia o formato específico para o schema canônico. O circuito e o contrato ficam estáveis; a fragilidade de cada fonte vive isolada no adaptador; o investidor escolhe quais provedores aceita confiar.

Isso atende três requisitos do projeto: não ficar preso à tecnologia de um país, ser fácil de acoplar (basta um novo adaptador) e não depender de um formato específico não testado.

**Completude:** provar que um número é real não prova que é o quadro completo (a empresa pode esconder dívida em outro lugar). Em produção, mitiga-se com fonte agregada (bureau de crédito, fisco) ou atestação de completude por auditor. No MVP, reconhece-se a limitação no README.

## 8. Identidade

A cadeia de confiança amarra três coisas: que **quem lista é a empresa real**, que **a atestação é sobre a mesma entidade**, e que **a entidade controla a carteira que recebe**.

A solução é elegante: o **provedor de atestação já é a raiz de KYC/identidade**. A mesma assinatura atesta "a entidade `H` (commitment do CNPJ/LEI) tem financeiros `F` **e** controla a chave pública `P`". A empresa usa `P` para assinar a listagem e receber o USDC — amarração criptográfica, uma raiz de confiança só, sem KYC próprio da plataforma.

A **identidade da empresa é pública**, junto com o **rating público**. O rating público **neutraliza o sinal de distress**: uma empresa verificadamente saudável captando capital parece crescimento, não socorro — estar na plataforma vira selo, não estigma. Para resistência a Sybil, usa-se `nullifier = hash(CNPJ)`, garantindo uma listagem ativa por empresa.

A tese se recalibra: não é "captação invisível", e sim **"capto com identidade e saúde provadas, mas com valor e números crus privados."**

## 9. Fluxo De Dinheiro E Custódia

O modelo escolhido é o **Modelo 1: o contrato é um escrow trustless e a empresa faz o próprio off-ramp.**

- A custódia da pool fica **no contrato**, não na plataforma — é o que torna o sistema confiável sem operador e responde ao "por que on-chain".
- No fechamento, o contrato libera USDC para a carteira Stellar da empresa, que saca via um **anchor existente**. A plataforma **nunca toca fiat**, evitando virar money transmitter.
- Tudo é **denominado em USDC** de ponta a ponta; a empresa absorve o câmbio nas bordas.
- No MVP, o on/off-ramp fiat fica **fora do escopo e narrado no README** — não se constrói anchor. A demo mostra o USDC indo do contrato para a empresa.

## 10. Por Que Stellar

A proposta conversa diretamente com a força da Stellar em **pagamentos, stablecoins, ativos tokenizados e fluxos cross-border**:

- **Soroban** implementa a pool, o escrow e a verificação das provas (BN254, Poseidon, Groth16).
- **USDC/stablecoins** tornam a custódia e o repagamento fáceis de demonstrar.
- **Baixo custo e liquidação rápida** viabilizam captação e repagamento on-chain.
- **Anchors (SEP-24/SEP-6/SEP-31)** abrem o caminho fiat para empresas off-chain.
- As primitivas ZK dos **Protocolos 25 (X-Ray) e 26 (Yardstick)** tornam a verificação de proof barata o suficiente para o caso de uso.

## 11. Panorama Competitivo E Posicionamento

O mercado de crédito on-chain já existe, mas nenhum player combina **valor de captação oculto** com **prova ZK de saúde como gate anti-limão** em um trilho de pagamento real.

| Solução | O que faz | Diferença do Confidential Debt Raise |
| --- | --- | --- |
| Maple Finance | Crédito institucional sub-colateralizado on-chain. | Transparente em valores e termos. Aqui o valor e os números ficam privados, com saúde provada por ZK. |
| Goldfinch | Crédito do mundo real para tomadores off-chain. | Foco em underwriting por pools delegadas; sem privacidade do valor nem prova ZK de cobertura. |
| Centrifuge | Financiamento de RWA e recebíveis tokenizados. | Estrutura de securitização aberta; o diferencial aqui é a confidencialidade do valor + gate de saúde. |
| Aztec / DeFi privado | Privacidade genérica de transações. | Resolve privacidade de pagamento, não captação de dívida com prova de solvência como filtro. |
| Stellar Private Payments (PoC) | Privacy pools com Circom/Groth16 em Soroban. | É a base técnica reaproveitável; não é um produto de crédito. |

Posicionamento defensável:

**"Já existe crédito on-chain, mas falta uma camada onde a empresa capta dívida com valor privado e saúde provada por ZK — transformando privacidade em filtro de qualidade, não em opacidade."**

## 12. Clientes E Usuários Alvo

O ICP são **empresas off-chain tradicionais**, com ênfase em quem mais sofre com o vazamento de informação ao captar:

- **PMEs privadas (não listadas)** que precisam de capital de giro — caso mais limpo legalmente.
- **Originadores de RWA / recebíveis** que querem antecipar fluxo sem expor números.
- **Empresas em negociação sensível** (M&A, rodada de equity) que não podem sinalizar dívida ao mercado.

Do lado do capital: investidores que buscam retorno em dívida com risco verificável, locais ou globais via stablecoin.

## 13. Diferenciais

- **Privacidade que vira qualidade:** o ZK esconde o valor e, ao mesmo tempo, filtra os limões.
- **Confiança sem operador:** o contrato é o escrow e o juiz; a plataforma não custodia fiat.
- **Fonte de dados agnóstica:** não fica presa a um país ou formato bancário.
- **Saúde provada, não declarada:** o rating é matemático, não autodeclarado.
- **Aderência ao ecossistema Stellar:** pagamentos, stablecoins e anchors como trilho real.

## 14. Modelo De Produto

A versão de produção pode evoluir para uma camada de **crédito privado verificável**:

- **Reputação on-chain + staking:** histórico de repagamento vira score; tomador arrisca skin in the game (resolve enforcement, fora do MVP).
- **Registro de provedores de atestação:** governança de quais fontes são confiáveis.
- **View key para auditores/reguladores:** disclosure seletivo para compliance.
- **Re-prova de saúde periódica:** covenants ao longo do empréstimo.
- **Mercado secundário:** posições de dívida tokenizadas e transferíveis.
- **Fee por captação ou por volume:** monetização alinhada ao valor intermediado.

## 15. Escopo Técnico Do MVP

1. **Soroban Pool Contract**
   - depósito de USDC em escrow e custódia;
   - commitment do valor-alvo `X` oculto;
   - fechamento condicional ao verificar a prova `Σ ≥ X`;
   - liberação para a empresa e trilho de repagamento;
   - resgate pro-rata com nullifier; eventos de aberto/fechado/repago.
2. **Circuito de Cobertura**
   - inputs financeiros assinados (mock no MVP);
   - verificação da assinatura do provedor;
   - prova de `dívida ≤ k · capacidade`, emitindo rating/banda público.
3. **Circuito de Fechamento**
   - prova de `Σ depósitos ≥ X` sem revelar `X`.
4. **Oráculo Mock**
   - serviço com keypair que assina o schema canônico de financeiros.
5. **Frontend / Demo**
   - empresa cria captação, gera provas client-side;
   - investidor vê rating/termos e aporta;
   - visualização de pool enchendo, fechamento e resgate.

## 16. Pitch

**Empresas saudáveis precisam de capital, mas captar hoje obriga a vazar quanto precisam e como estão — e o mercado lê isso como fraqueza. O Confidential Debt Raise deixa a empresa captar dívida na Stellar com identidade e saúde provadas por ZK, enquanto o valor da captação e os números do balanço ficam privados. O contrato Soroban custodia o capital, fecha a pool quando o alvo oculto é atingido e governa o repagamento.**

**A privacidade aqui não é opacidade: a mesma prova que esconde o valor filtra as empresas podres, transformando o mercado de limões em mercado de qualidade.**

Frase curta para apresentação:

**"Capital sem vazar — prove sua saúde, esconda seus números."**

## 17. Estado E Decisões Em Aberto

**Decidido:** ICP off-chain · enforcement (reputação+staking) é produção, MVP pula · ZK como anti-limão + privacidade · privacidade seletiva, banda não ratio · fonte MVP = mock assinado · produção = camada de atestação agnóstica · identidade pública + rating público + provedor como raiz de KYC + nullifier por CNPJ · fluxo de dinheiro Modelo 1 (escrow no contrato, USDC ponta a ponta, off-ramp pela empresa).

**Em aberto:** depósitos públicos vs confidenciais · path ZK (Circom+Groth16 vs Noir) · repagamento confidencial · schema canônico exato + fórmula de cobertura/rating · mecânica de anti-overshoot/refund · mecânica do nullifier no resgate · Sybil (detalhe) · governança do registro de provedores · precificação dos termos · parcial vs all-or-nothing + deadline da pool · default handling no contrato · re-prova de saúde · view key para auditor · mercado secundário · geração de prova client-side/UX · privacidade de metadados · modelo de receita.

## 18. Próximos Passos

1. Fechar o **schema canônico + fórmula de cobertura** (coração do circuito).
2. Decidir **depósitos públicos vs confidenciais** e o **path ZK**.
3. Reaproveitar o **Stellar Private Payments PoC** como esqueleto do contrato + verifier.
4. Implementar o circuito de cobertura com oráculo mock e o circuito de fechamento.
5. Montar a demo (criar captação → aportar → fechar → resgatar) e o pitch.
