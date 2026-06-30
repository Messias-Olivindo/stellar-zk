# Confidential Debt Raise (On-Chain): Captação de Dívida Privada para Protocolos na Stellar

> Projeto para o hackathon **Stellar Hacks: Real-World ZK**. Contexto do hackathon em `../../development/AGENT_CONTEXT.md`.
> Variante com **ICP = protocolos/startups já on-chain na Stellar**. Ideia irmã (ICP off-chain) em `../offchain-confidential-debt-raise/`.

## 1. Visão Geral

O **Confidential Debt Raise (On-Chain)** é uma plataforma onde um protocolo, DAO ou startup que já opera na Stellar capta dívida sem revelar ao mercado o **valor que precisa**, provando por conhecimento zero que tem capacidade de pagar.

A diferença em relação à variante off-chain está na **fonte de verdade**: como o tomador já tem tesouraria on-chain, o dado financeiro é **nativamente verificável**, dissolvendo o problema do oráculo. A prova de saúde se apoia em estado on-chain real, e a captação esconde apenas o **evento e o tamanho** do levantamento.

## 2. Dor Que O MVP Resolve

Um protocolo on-chain tem uma **valuation ao vivo** — o preço do seu token, negociado 24/7. Revelar que está captando dívida de emergência é um sinal imediato de distress que **derruba o token na hora**.

- Captar abertamente expõe o tamanho do levantamento, os termos e a intenção estratégica.
- Token holders, concorrentes e o mercado leem "precisa de runway" como fraqueza.
- A própria transparência nativa do on-chain torna esse vazamento mais agudo.

O Confidential Debt Raise resolve isso permitindo que o protocolo **prove saúde e capte com valor privado**, protegendo o token enquanto move capital real on-chain.

## 3. Como Funciona

1. O protocolo define a necessidade `X` e a **compromete on-chain** de forma oculta.
2. Publica **termos públicos**: juros e prazo.
3. Gera uma **prova de saúde** ancorada em estado on-chain (tesouraria, TVL, receita verificável) — sem oráculo externo.
4. Investidores aportam USDC na pool custodiada pelo contrato.
5. O contrato fecha quando prova `Σ depósitos ≥ X` sem revelar `X`.
6. Repagamento em USDC ao longo do prazo; investidores resgatam pro-rata.

## 4. Onde Entra ZK

O ZK é **load-bearing** em dois pontos:

- **Fechamento por threshold:** prova `Σ ≥ X` com `X` oculto.
- **Prova de saúde:** prova que a obrigação cabe em um múltiplo seguro da capacidade verificável on-chain, exibida como **banda/rating**, sem revelar os números crus.

Como o dado é on-chain, a prova de saúde pode referenciar diretamente estado verificável, reduzindo a superfície de confiança.

## 5. Modelo De Privacidade

Privacidade **seletiva**: o público vê identidade (endereço/protocolo), rating, juros e prazo; o **valor `X` e os números crus ficam ocultos**. Como parte da tesouraria já é pública on-chain, o sigilo ancora no **evento e tamanho da captação e nos termos**, não no saldo. Mostrar banda/rating, nunca ratio exato.

## 6. O Problema Dos Limões E O Anti-Limão

Esconder informação atrai os piores tomadores. A prova de saúde, sendo um **sinal custoso de imitar**, funciona como porteiro: protocolo insolvente não consegue gerá-la. Isso inverte o mercado de limões em **mercado de qualidade** — o investidor confia, os juros caem, e o bom tomador entra. Vale a ressalva de que parte da comunidade crypto **espera transparência**, então o pitch precisa enquadrar a privacidade como proteção contra distress signaling, não como ocultação.

## 7. Fonte De Dados E Confiança

A vantagem-chave deste ICP: **o dado é nativamente on-chain**, então o problema do *garbage-in* praticamente desaparece — não há balanço autodeclarado para forjar. A capacidade de pagamento se mede sobre tesouraria, TVL, fluxo de receita e histórico, tudo verificável. A **reputação também é nativamente verificável** (histórico de repagamento on-chain), o que habilita empréstimo por reputação como evolução natural.

## 8. Identidade

A identidade é o **próprio endereço/contrato do protocolo na Stellar**, já público e com histórico verificável. Não há necessidade de KYC externo para o vínculo básico, embora um provedor de atestação possa ser somado para dados off-chain complementares. Resistência a Sybil se apoia em reputação e histórico on-chain.

## 9. Fluxo De Dinheiro E Custódia

**Modelo trustless puro:** o contrato custodia a pool em USDC, libera ao endereço do protocolo no fechamento e governa o repagamento. Como tomador e investidor já são on-chain, **não há leg fiat** — o fluxo é inteiramente on-chain, o que simplifica frente à variante off-chain.

## 10. Por Que Stellar

- **Soroban** implementa pool, escrow e verificação de provas (BN254, Poseidon, Groth16).
- **USDC/stablecoins** como unidade de captação e repagamento.
- **Baixo custo e liquidação rápida** para o ciclo on-chain completo.
- Primitivas ZK dos **Protocolos 25 e 26** barateiam a verificação.
- Dogfooding do próprio ecossistema DeFi da Stellar (Blend, DEXs, anchors).

## 11. Panorama Competitivo E Posicionamento

| Solução | O que faz | Diferença |
| --- | --- | --- |
| Maple Finance | Crédito institucional on-chain. | Transparente; aqui o valor é privado e a saúde provada por ZK. |
| Blend (Stellar) | Lending pool primitivo na Stellar. | Empréstimo colateralizado aberto; não há captação de valor oculto nem prova de saúde. |
| Aztec / DeFi privado | Privacidade genérica. | Não é captação de dívida com prova de solvência como filtro. |
| Stellar Private Payments (PoC) | Privacy pools em Soroban. | Base técnica reaproveitável, não produto de crédito. |

Posicionamento:

**"Captação de dívida privada para protocolos Stellar — prove saúde on-chain, esconda o tamanho do levantamento, proteja o token."**

## 12. Clientes E Usuários Alvo

- **Protocolos DeFi** (lending, DEX) com tesouraria que precisam de runway/liquidez.
- **DAOs** com treasury que automatizam captação.
- **Emissores de stablecoin / anchors** e **startups do ecossistema** (SCF).

Mercado menor e mais jovem que o off-chain, porém mais consciente de privacidade, mais disposto a testar ZK e mais crível para os juízes do hackathon.

## 13. Diferenciais

- **Oráculo dissolvido:** dado nativamente verificável on-chain.
- **Reputação nativa:** histórico de repagamento on-chain habilita crédito por reputação.
- **Tese de token forte:** protege a valuation ao vivo contra distress signaling.
- **Fluxo 100% on-chain:** sem leg fiat, mais simples de demonstrar.
- **Dogfooding Stellar:** ZK real sobre o próprio ecossistema.

## 14. Modelo De Produto

- **Crédito por reputação on-chain + staking.**
- **Re-prova de saúde** ligada a estado de tesouraria em tempo real.
- **Mercado secundário** de posições tokenizadas.
- **View key** para disclosure seletivo.
- **Fee por captação ou volume.**

## 15. Escopo Técnico Do MVP

1. **Soroban Pool Contract:** escrow, commitment de `X` oculto, fechamento por prova `Σ ≥ X`, liberação e repagamento, resgate com nullifier.
2. **Circuito de Saúde:** prova sobre estado on-chain de que a obrigação cabe em `k · capacidade`, emitindo rating.
3. **Circuito de Fechamento:** prova `Σ ≥ X` sem revelar `X`.
4. **Frontend / Demo:** protocolo cria captação, investidor aporta, visualização de fechamento e resgate.

## 16. Pitch

**Protocolos na Stellar têm valuation ao vivo: revelar uma captação de dívida derruba o token na hora. O Confidential Debt Raise deixa o protocolo captar com valor privado, provando saúde diretamente sobre estado on-chain verificável. O contrato custodia o capital, fecha a pool no alvo oculto e governa o repagamento — tudo on-chain, sem oráculo externo.**

Frase curta:

**"Capte sem mover seu token — prove saúde on-chain, esconda o tamanho."**

## 17. Estado E Decisões Em Aberto

**Decidido (herdado da ideia irmã):** ZK como anti-limão + privacidade · privacidade seletiva, banda não ratio · enforcement (reputação+staking) é produção.

**Específico desta variante (a confirmar):** ICP on-chain (protocolos/startups) · fonte = estado on-chain nativo · fluxo 100% on-chain sem leg fiat · identidade = endereço do protocolo.

**Em aberto:** depósitos públicos vs confidenciais · path ZK (Circom+Groth16 vs Noir) · definição exata da métrica de capacidade on-chain · mecânica de anti-overshoot · precificação dos termos · parcial vs all-or-nothing + deadline · default handling · corte de escopo do MVP.

## 18. Próximos Passos

1. Definir a **métrica de capacidade verificável on-chain** (tesouraria, TVL, receita).
2. Decidir **depósitos públicos vs confidenciais** e o **path ZK**.
3. Reaproveitar o **Stellar Private Payments PoC** como esqueleto.
4. Implementar circuito de saúde e de fechamento.
5. Montar demo e pitch focados na tese de token.
