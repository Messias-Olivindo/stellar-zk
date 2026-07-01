# ZK-Bounty - Documento Critico de Posicionamento

## 1. Ideia refinada

ZK-Bounty nao deve ser apresentada como uma plataforma generica de bug bounty. O recorte mais defensavel e:

**um protocolo de bounties verificaveis para invariantes de contratos Soroban.**

Um protocolo publica uma invariante objetiva, um alvo Soroban, uma verification key e uma recompensa em USDC. Um pesquisador prova em ZK que conhece um input privado que quebra essa invariante, sem revelar o exploit antes de garantir a recompensa. O contrato Soroban verifica a prova, reserva o pagamento e registra a descoberta no historico on-chain do pesquisador.

O objeto central nao e o "relatorio de bug". O objeto central e o **contraexemplo criptografico**:

```text
Existe um input privado w tal que Transition(state, w) viola Invariant.
```

## 2. Mudanca essencial em relacao ao documento atual

O `IDEA.md` novo ja melhora muito a proposta, mas ainda fala alto demais em alguns pontos:

- Parece prometer prova de vulnerabilidades arbitrarias de Soroban.
- Parece prometer pagamento automatico sem nenhum processo de disclosure.
- Mistura reputacao, patch proof e bounty marketplace em um escopo grande.
- Usa claims de custo que precisam de benchmark.

O ajuste recomendado e separar o produto em duas camadas:

1. **Invariant Bounties:** verificaveis por ZK, objetivos, escopo do hackathon.
2. **General Bug Reports:** subjetivos, com triagem humana, visao futura.

O hackathon deve entregar apenas a primeira camada.

## 3. O que torna a ideia forte

### ZK e load-bearing

Sem ZK, o pesquisador precisa revelar o exploit para convencer o protocolo. Ao revelar, perde poder de negociacao e aumenta risco para usuarios. Com ZK, ele prova que conhece o contraexemplo sem mostrar o input.

A prova nao e decorativa. Ela e a ponte entre:

- privacidade do pesquisador;
- garantia de pagamento;
- verificabilidade para o protocolo;
- seguranca temporaria dos usuarios ate o patch.

### Stellar e central

A Stellar nao entra apenas como escrow. A tese depende de quatro pecas Stellar-native:

- O alvo e um contrato Soroban.
- O verifier roda em Soroban.
- A recompensa usa Stellar Asset Contract, idealmente USDC ou token mock equivalente.
- A reputacao e registrada como estado on-chain em Soroban.

Protocolos 25 e 26 reforcam a narrativa porque adicionam e expandem primitivas ZK-friendly como BN254 e Poseidon/Poseidon2.

### A demo pode ser inevitavel

Uma boa demo nao deve mostrar "um usuario subiu um arquivo de proof". Ela deve mostrar:

1. Um contrato Soroban com uma invariante financeira publica.
2. Um pesquisador encontrando um input secreto que quebra a invariante.
3. Uma prova ZK verificando isso on-chain.
4. A recompensa ficando reservada antes do exploit ser revelado.
5. A reputacao do pesquisador subindo.

Esse fluxo e mais memoravel do que um policy vault porque o conflito e obvio: "eu descobri um bug real, mas nao quero entregar de graca".

## 4. Produto minimo defensavel

O MVP deve ter quatro componentes:

1. **Bounty Escrow Contract**
   - Cria bounty.
   - Guarda recompensa.
   - Registra invariant id, target contract, wasm hash, verifier e deadline.
   - Recebe proof e public inputs.
   - Reserva recompensa para o pesquisador se a proof passar.

2. **Verifier Contract**
   - Verifica Groth16 ou UltraHonk.
   - Recebe public inputs vinculados ao bounty.
   - Retorna valido/invalido.

3. **Invariant Circuit**
   - Modela uma transicao pequena do contrato alvo.
   - Usa witness privado como input de exploit.
   - Prova que a transicao viola uma invariante publica.
   - Gera commitment e nullifier para evitar replay e duplicatas.

4. **Researcher CLI/UI**
   - Mostra o bounty aberto.
   - Gera prova local.
   - Submete prova.
   - Mostra recompensa reservada e reputacao.

## 5. Demo recomendada: AMM invariant break

Criar um contrato Soroban demonstrativo de AMM com uma formula vulneravel:

```text
dy_buggy = reserve_y * dx / reserve_x
new_x = reserve_x + dx
new_y = reserve_y - dy_buggy
```

Invariante publica:

```text
new_x * new_y >= reserve_x * reserve_y
```

O pesquisador conhece um `dx` privado que faz a formula vulneravel quebrar a invariante. O circuito prova:

```text
poseidon(dx, salt, bounty_id) == witness_commitment
dy_buggy == floor(reserve_y * dx / reserve_x)
dy_buggy < reserve_y
new_x == reserve_x + dx
new_y == reserve_y - dy_buggy
new_x * new_y < reserve_x * reserve_y
```

Public inputs:

- `bounty_id`
- `target_wasm_hash`
- `invariant_id`
- `reserve_x`
- `reserve_y`
- `witness_commitment`
- `nullifier`

Private witness:

- `dx`
- `salt`

Por que essa demo funciona:

- E financeira, portanto combina com Stellar.
- E simples de explicar.
- A invariante e objetiva.
- O exploit fica privado ate a recompensa ser garantida.
- A prova e pequena o bastante para hackathon.

## 6. Fluxo de disclosure responsavel

O contrato nao precisa pagar imediatamente no primeiro bloco. O fluxo mais defensavel:

1. `submit_proof`: proof valida, recompensa fica reservada ao pesquisador.
2. `submit_report_hash`: pesquisador registra hash do report privado ou link criptografado.
3. `accept_report` ou `timeout`: protocolo aceita e libera, ou timeout libera se o protocolo nao responder.
4. `mark_patched`: opcional para demo plus, registra patch verificado.

Para o hackathon, a demo pode simplificar:

```text
proof valida -> reward locked -> researcher claims after report hash
```

Essa diferenca evita a critica de que o sistema incentiva pesquisadores a travarem recompensa sem cooperar com o patch.

## 7. Como vence o Payment Vault

ZK-Bounty so vence o Payment Vault se for apresentada neste recorte:

| Criterio | Payment Vault | ZK-Bounty refinada |
|---|---|---|
| Papel do ZK | Provar whitelist privada ou policy privada | Provar existencia de bug sem revelar exploit |
| Demo | Pagamento aprovado/bloqueado | Bug encontrado, proof verificada, reward travado |
| Stellar | Trilha de pagamentos e vault Soroban | Alvo Soroban, verifier Soroban, escrow SAC, reputacao Soroban |
| Diferenciacao | Policy vault aberto | Mercado de invariant proofs para contratos Soroban |
| Network effect | Baixo, vaults isolados | Alto, reputacao e historico de descobertas |

O argumento central:

**Payment Vault protege pagamentos de agentes. ZK-Bounty protege o proprio ecossistema Soroban, transformando bugs formalizaveis em provas, recompensas e reputacao.**

## 8. Fragilidades e respostas honestas

### 8.1 "Isso prova qualquer bug?"

Nao. O MVP prova apenas bugs que podem ser expressos como quebra de invariante em um harness publico e pequeno.

Resposta de pitch:

**Comecamos com invariant bounties porque sao objetivos e verificaveis. Reports subjetivos continuam existindo, mas nao sao o que estamos automatizando no hackathon.**

### 8.2 "Por que nao usar Immunefi?"

Immunefi e HackerOne sao bons para triagem e coordenacao humana. ZK-Bounty e diferente: quando a propriedade e formalizavel, a primeira etapa pode ser verificada por contrato.

Resposta:

**Nao substituimos toda triagem. Transformamos a parte objetiva do bug bounty em uma prova on-chain.**

### 8.3 "E se o circuito estiver errado?"

Esse e o maior risco tecnico. O circuito vira parte do bounty. Uma verification key ruim pode aceitar provas erradas.

Mitigacao:

- Templates auditados de invariantes.
- Circuit hash e verification key hash publicados no bounty.
- Recompensas limitadas por bounty.
- Pause/cancel antes de proof se houver bug no template.

### 8.4 "E se dois pesquisadores acharem o mesmo bug?"

Usar `nullifier = poseidon(witness_commitment, bounty_id)` e status por bounty. O primeiro proof valida reserva a recompensa. Futuramente, duplicatas podem receber reputacao menor ou nada.

### 8.5 "Como lidar com severidade?"

No MVP, nao calcular severidade. A recompensa e fixa por invariante. Severidade subjetiva fica para versao futura.

### 8.6 "Automatic payout pode ser perigoso?"

Sim. O pitch deve falar em "lock" ou "reserve" antes do disclosure, nao necessariamente pagamento final instantaneo. Para uma demo objetiva e controlada, pode haver claim automatico apos report hash.

## 9. Frase final recomendada

**ZK-Bounty is a provable bug bounty layer for Soroban: researchers prove they know a private input that breaks a public invariant, Soroban verifies the proof, locks the USDC reward, and records on-chain security reputation.**

## 10. Criterio de sucesso no hackathon

O projeto passa no crivo se mostrar:

1. Um bounty criado para uma invariante Soroban.
2. Uma prova ZK real gerada a partir de um input privado.
3. O contrato Soroban verificando a prova.
4. A recompensa ficando reservada ou liberada.
5. A reputacao do pesquisador mudando on-chain.
6. Um caso invalido falhando.

Se qualquer um desses pontos virar mock sem explicacao, o Payment Vault volta a ser mais seguro como escolha.
