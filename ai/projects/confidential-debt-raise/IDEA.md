# Confidential Debt Raise — ideia

> Projeto p/ hackathon **Stellar Hacks: Real-World ZK**. Contexto do hackathon: ver `../../development/AGENT_CONTEXT.md`.
> Estado: **em design** (decisões de produto em aberto no fim). Sem código ainda.

---

## Pitch em uma linha

Plataforma onde uma empresa/protocolo capta dívida **sem revelar ao mercado o valor que precisa**, mas **provando por ZK que é financeiramente saudável** — captura capital sem sinalizar fraqueza.

Headline: **"capital sem vazar — prove sua saúde, esconda seus números."**

---

## Como funciona (fluxo)

1. Tomador define necessidade de capital `X` e o **compromete on-chain** (commitment Poseidon/Pedersen) — `X` fica **oculto**.
2. Tomador define termos **públicos**: juros e prazo.
3. Tomador gera **prova ZK de cobertura** (saúde financeira) — vira gate de entrada.
4. Investidores aportam na pool.
5. Contrato fecha a pool quando prova `Σ depósitos ≥ X` **sem revelar X nem (opcional) os depósitos**.
6. Repagamento (principal + juros) no prazo definido.

---

## Onde o ZK é load-bearing (não decorativo)

1. **Fechamento por threshold:** prova `Σ ≥ X` com `X` oculto → pool trava sozinha sem vazar o alvo.
2. **Prova de cobertura / saúde (anti-limão):** prova `dívida ≤ k · capacidade` sem revelar números crus. É o **porteiro** que mantém empresa podre fora.

---

## Modelo de privacidade (seletivo, não opaco)

| Quem | Vê o quê |
|---|---|
| Mercado / concorrente / público | **cego** ao valor cru → sem sinal de distress → sem desvalorização |
| Investidor | juros + prazo + **prova ZK de cobertura em BANDA/rating** (ex: rating B = cobertura 2x–3x) |

- Mostrar **banda/rating**, NUNCA o ratio exato → senão dá pra engenharia-reversar o valor oculto.
- Investidor recebe **bound**, nunca o `X` cru → senão N investidores em colusão reconstroem o segredo.

---

## Por que resolve o "mercado de limões"

- Só esconder o número → atrai empresa **podre** (quem mais quer esconder) → espiral de desconfiança → mercado morre.
- Esconder **e provar saúde** → prova é sinal **custoso de imitar** (podre não gera prova de cobertura saudável) → **filtra limão** → investidor confia → juros baixo → empresa boa vem. **Ciclo virtuoso.**
- O ZK aqui = anti-limão, não só privacidade.

## Por que empresa BOA usaria

Dor da empresa boa não é "falta dinheiro" — é "captar me obriga a vazar info". Hoje: banco expõe todo o balanço (lento, colateral, spread); captação pública expõe valor a todos (concorrente vê estratégia, fornecedor sobe preço, valuation cai). Aqui: prova saúde, esconde números, capital global em stablecoin.

---

## O problema do oráculo (garbage-in) e a saída

ZK prova **a conta, não a verdade dos inputs**. Se a empresa mente os números, prova sai linda e falsa. Solução: dado tem que vir **assinado por fonte que a empresa não controla**; circuito prova (a) assinatura válida + (b) a conta sobre o dado assinado.

Menu de fontes (forte → fácil): on-chain nativo · adquirente (Stone/Cielo) · Open Finance Brasil · SPED/Receita · zkTLS (Root14) · auditor.

**Armadilha de completude:** provar 1 número real ≠ provar quadro completo (pode esconder dívida em outro lugar). Mitigar com fonte agregada (Serasa/SPED) ou atestação de completude (auditor). Reconhecer no README do MVP.

---

## Decisões já tomadas

- **Enforcement de repagamento:** reputação + staking = **visão de produção**; **MVP pula** enforcement (documentar no README).
- **ZK é anti-limão + privacidade**, via prova de cobertura como gate.
- **Privacidade seletiva**: público cego ao valor; investidor vê rating em banda.
- **Mostrar banda/rating, não ratio exato.**

## Decisões em aberto (a fechar)

1. **ICP final:** protocolos/startups **já on-chain na Stellar** (resolve oráculo, tese de token forte, judge-crível, TAM menor) **vs** empresa tradicional (visão maior, precisa mock/zkTLS). → *atualmente inclinado p/ on-chain no MVP.*
2. **Fonte do dado financeiro no MVP:** on-chain nativo · oráculo mock assinado · zkTLS (Root14). → zkTLS interessa mas é arriscado no prazo; on-chain nativo é o mais limpo se ICP for on-chain.
3. **Depósitos públicos vs confidenciais** (muda complexidade do circuito de fechamento).
4. **Path ZK:** Circom+Groth16 (reusa PoC Nethermind stellar-private-payments) vs Noir.
5. **Repagamento confidencial?** (provável future work).
6. **Corte de escopo do MVP.**

---

## Tensões/riscos conhecidos (do brainstorm adversarial)

- Enforcement on-chain não força repagamento real (MVP assume; produção = colateral/reputação/legal).
- Seleção adversa — mitigada pelo gate de cobertura.
- Legal: esconder dívida pode ser ilegal p/ empresa **listada** → ICP = privada/PME ou entidade on-chain.
- "Por que on-chain?" → prova verificável **sem operador confiável** (investidor confia na matemática).
- Se ICP on-chain c/ tesouraria pública: privacidade ancora no **evento/termos da captação**, não no saldo; e parte da comunidade crypto espera transparência (trade-off).
- Paradoxo do fechamento se depósitos forem confidenciais (somar valores ocultos = mais complexo).
- Over/undershoot do último depósito (capacidade restante oculta) → refund de excesso.
- Nullifiers no resgate (anti duplo-saque).

---

## Próximos passos

- Fechar decisões em aberto (esp. ICP + fonte de dados).
- Desenhar MVP técnico: o que é privado/público/provado + circuitos + contrato Soroban + plano por dias.
- Sugestão de MVP mínimo defensável: captação única + valor-alvo oculto + prova de fechamento por threshold + prova de cobertura (rating banda) no onboarding. Repagamento e anonimato de investidor = future work no README.
