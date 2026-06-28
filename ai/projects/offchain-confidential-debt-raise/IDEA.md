# Off-chain Confidential Debt Raise — ideia

> Projeto p/ hackathon **Stellar Hacks: Real-World ZK**. Contexto do hackathon: ver `../../development/AGENT_CONTEXT.md`.
> Variante com **ICP = empresas off-chain (tradicionais)**. Estado: **em design**, sem código.
> Ideia irmã (ICP em aberto / inclinado on-chain): `../confidential-debt-raise/`.

---

## Pitch em uma linha

Empresa **off-chain tradicional** capta dívida **sem revelar ao mercado o valor que precisa**, provando por ZK que é financeiramente saudável. Captura capital sem sinalizar fraqueza.

Headline: **"capital sem vazar — prove sua saúde, esconda seus números."**

---

## Como funciona (fluxo)

1. Empresa define necessidade `X` e a **compromete on-chain** (commitment Poseidon/Pedersen) — `X` oculto.
2. Termos **públicos**: juros e prazo.
3. Empresa gera **prova ZK de cobertura** (saúde financeira) → gate de entrada.
4. Investidores aportam na pool.
5. Contrato fecha quando prova `Σ depósitos ≥ X` sem revelar X.
6. Repagamento (principal + juros) no prazo.

---

## Onde o ZK é load-bearing

1. **Fechamento por threshold:** prova `Σ ≥ X` com `X` oculto.
2. **Prova de cobertura / saúde (anti-limão):** prova `dívida ≤ k · capacidade` sem revelar números crus — porteiro que mantém empresa podre fora.

---

## Modelo de privacidade (seletivo)

| Quem | Vê |
|---|---|
| Mercado/concorrente/público | cego ao valor cru → sem sinal de distress |
| Investidor | juros + prazo + **prova ZK de cobertura em BANDA/rating** |

- Banda/rating, NUNCA ratio exato (evita engenharia reversa do valor oculto).
- Investidor recebe bound, nunca `X` cru (evita reconstrução por colusão).

---

## Anti-limão + por que empresa boa usa

- Só esconder → atrai empresa podre → espiral de desconfiança → mercado morre.
- Esconder **e provar saúde** → sinal custoso de imitar → filtra limão → ciclo virtuoso.
- Empresa boa: dor não é falta de dinheiro, é "captar me obriga a vazar info". Aqui prova saúde, esconde números, capital global em stablecoin, sem expor balanço a banco/concorrente.

---

## Fonte de dados (o ponto crítico do ICP off-chain)

ZK prova a conta, não a verdade dos inputs (garbage-in). Dado tem que vir **assinado por fonte que a empresa não controla**.

### Decisão MVP — mock assinado
Serviço com keypair de dev assina os financeiros; circuito verifica a assinatura de verdade. Mesma interface dos provedores reais → **sem retrabalho** ao trocar depois. README aponta produção.

### Decisão produção — camada de atestação AGNÓSTICA (não fonte única)
Requisito do dono do projeto: **não pode ficar preso a tecnologia de um país (ex: Open Finance Brasil), tem que ser fácil de acoplar e não depender de formato muito específico/não testado por banco.**

Solução: abstração **"provedor de atestação"** = entidade que assina uma declaração sobre um **schema canônico** (`{passivos, fluxo_caixa, período, moeda, ...}`), com **chave pública registrada**.

- **Circuito** verifica só: (a) assinatura de provedor registrado, (b) cobertura sobre os campos canônicos.
- **Adaptador** por fonte mapeia formato específico → schema canônico. Formato do banco fica **isolado no adaptador**; circuito/contrato **estáveis**.
- **Investidor/mercado escolhe** quais provedores aceita confiar.
- Resolve as 3 exigências: não preso a 1 país · fácil acoplar (só novo adaptador) · fragilidade isolada no adaptador.

Provedores possíveis (complementares, não ou-um-ou-outro):
- **zkTLS** (Root14): qualquer HTTPS, sem cooperação do banco nem regulador → mais alcance global; custo = template de parsing por portal, frágil a mudança de UI.
- **Open Finance / Open Banking**: robusto e estruturado, mas **por jurisdição** (Brasil, UK, PSD2…), exige banco participante + regulação.
- adquirente (recebíveis) · SPED/fisco · auditor · etc.

### Completude (dívida escondida em outro lugar)
Provar 1 número real ≠ provar quadro completo. Mitigar com fonte agregada (Serasa/SPED) ou atestação de completude por auditor. Reconhecer no README do MVP.

---

## Identidade (decidido — pública + provedor como raiz de KYC)

Cadeia de confiança amarra 3 coisas: (1) quem lista é a empresa real, (2) a atestação é sobre a mesma entidade, (3) a entidade controla a carteira que recebe.

- **Provedor de atestação = raiz de identidade/KYC** (sem KYC próprio da plataforma). Mesma assinatura atesta: "entidade `H` (commitment CNPJ/LEI) tem financeiros `F` **e** controla a chave pública `P`". Empresa usa `P` p/ assinar a listagem e receber USDC → amarração criptográfica, uma raiz só.
- **Identidade da empresa = PÚBLICA**, junto com **rating/banda público**. O rating público **neutraliza o sinal de distress**: "empresa verificadamente saudável captando" parece crescimento, não socorro. Estar na plataforma + rating bom = selo, não estigma.
- **Sybil:** `nullifier = hash(CNPJ)` → uma empresa = uma listagem ativa (ou limite).
- **Recalibração da tese:** não é "captação invisível" — é **"capto com identidade e saúde provadas, mas com valor e números crus privados."**

| Público vê | Escondido |
|---|---|
| identidade, rating/banda, juros, prazo | **valor exato `X`** + **números crus do balanço** |

## Fluxo de dinheiro / custódia (decidido — Modelo 1)

Trajeto: `Investidor ─USDC→ [escrow no contrato] → (fecha, prova Σ≥X) ─USDC→ carteira Stellar da empresa ─off-ramp→ fiat`. Repagamento: `empresa ─on-ramp→ USDC → contrato → investidores claim pro-rata + juros (nullifier)`.

- **Modelo 1:** contrato = **escrow trustless**; empresa faz o **próprio off-ramp** via anchor existente; plataforma **nunca toca fiat** (não vira money transmitter; resolve o "por que on-chain").
- **Denominação = USDC ponta a ponta**; empresa absorve câmbio nas bordas (ramp). Juros em USDC.
- **Custódia no contrato**, não na plataforma.
- **MVP:** tudo on-chain em USDC; off-ramp/on-ramp fiat **fora do escopo / narrado no README** (não construir anchor). Demo mostra USDC contrato→empresa.
- Ferramental de produção: anchors Stellar (SEP-24/SEP-6/SEP-31), USDC (Circle).

## Decisões já tomadas

- **ICP = empresa off-chain tradicional.**
- **Enforcement:** reputação+staking = produção; **MVP pula**.
- **ZK = anti-limão + privacidade** via prova de cobertura (gate).
- **Privacidade seletiva**; mostrar banda/rating, não ratio exato.
- **Fonte MVP = mock assinado.**
- **Fonte produção = camada de atestação agnóstica** (zkTLS + Open Finance + outros como provedores plugáveis).

## Decisões em aberto

1. **Depósitos públicos vs confidenciais** (complexidade do circuito de fechamento).
2. **Path ZK:** Circom+Groth16 (reusa PoC Nethermind stellar-private-payments) vs Noir.
3. **Repagamento confidencial?** (provável future work).
4. **Schema canônico exato** + fórmula de cobertura/rating.
5. **Corte de escopo do MVP.**

---

## Riscos conhecidos

- Enforcement on-chain não força repagamento real (MVP assume; produção = reputação/colateral/legal).
- Legal: esconder dívida pode ser ilegal p/ empresa **listada** → ICP = privada/PME.
- "Por que on-chain?" → prova verificável **sem operador confiável**.
- Paradoxo do fechamento se depósitos confidenciais (somar valores ocultos = mais complexo).
- Over/undershoot do último depósito (capacidade oculta) → refund de excesso.
- Nullifiers no resgate (anti duplo-saque).
- Fragilidade de parsing por portal no zkTLS; cobertura de completude.

---

## Próximos passos

- Fechar decisões em aberto (depósitos públicos/confidenciais, path ZK).
- Desenhar MVP técnico: privado/público/provado + circuitos + contrato Soroban + plano por dias.
- MVP mínimo sugerido: captação única + valor-alvo oculto + prova de fechamento por threshold + prova de cobertura (rating banda) no onboarding, fonte = mock assinado. Repagamento e anonimato de investidor = future work.
