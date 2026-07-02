# Development Plan: ZK-Bounty Hackathon MVP

## 1. Escopo fechado

Construir uma demo funcional da ZK-Bounty com a seguinte tese:

**Um pesquisador encontra um input privado que quebra uma invariante publica de um contrato Soroban. Ele gera uma prova ZK off-chain sem revelar o input. Um contrato Soroban verifica a prova, reserva a recompensa e atualiza a reputacao on-chain.**

O MVP e deliberadamente estreito:

- alvo: um contrato Soroban demonstrativo de AMM vulneravel;
- bug: quebra de invariante financeira de constant product;
- prova: conhecimento de um `dx` privado que torna a invariante falsa;
- chain: Soroban verifica a prova, guarda recompensa e registra reputacao;
- disclosure: proof valida reserva recompensa; claim exige `report_hash`.

O MVP **nao** tenta provar execucao arbitraria da Soroban VM, qualquer bug de qualquer contrato, RCE, API/web app real, severidade subjetiva, patch proof, reputacao tokenizada ou marketplace de bounties.

## 2. Decisoes tecnicas

| Area | Decisao para MVP |
|---|---|
| Prova ZK | **Circom + Groth16** |
| Verificacao on-chain | Contrato Soroban Groth16 verifier, reaproveitando exemplo existente sempre que possivel |
| Alvo da demo | AMM Soroban vulneravel |
| Invariante | `new_x * new_y >= old_x * old_y` |
| Witness privado | `dx` e `salt` |
| Public inputs | `statement_hash`, `reserve_x`, `reserve_y`, `witness_commitment`, `nullifier` |
| Anti-duplicata | `nullifier` armazenado no escrow |
| Pagamento | Token mock no localnet; SAC/USDC testnet se sobrar tempo |
| Reputacao | Storage simples no `BountyEscrow`, nao token |
| UI | Opcional; CLI e logs on-chain bastam para demo tecnica |

Noir/UltraHonk e RISC Zero ficam como caminhos futuros ou fallback apenas se Circom/Groth16 travar de forma irrecuperavel. Nao vamos manter varios paths em paralelo.

## 3. Arquitetura

```text
Protocol owner
  -> deploy vulnerable Soroban AMM
  -> create_bounty(target, invariant, verifier, reward)
      -> BountyEscrow
          -> stores bounty metadata
          -> escrows reward token

Researcher
  -> finds private dx
  -> generates statement hash, witness commitment and nullifier
  -> generates Groth16 proof locally
  -> submit_proof(proof, public_inputs)
      -> BountyEscrow
          -> checks bounty metadata
          -> checks nullifier unused
          -> calls Groth16 verifier
          -> reserves reward for researcher
          -> emits ProofVerified / RewardLocked

Researcher
  -> submit_report_hash(report_hash)
  -> claim_reward()
      -> BountyEscrow
          -> transfers reward
          -> increments reputation
```

Componentes do repo a criar:

```text
contracts/
  vulnerable-amm/       # contrato alvo didatico
  bounty-escrow/        # escrow + reputacao minima
  groth16-verifier/     # verifier integrado ou wrapper do exemplo Stellar

circuits/
  amm_invariant.circom  # prova do invariant break
  input.json            # exemplo local, sem segredo real
  scripts/              # compile/prove/verify/export calldata

cli/
  zk-bounty-demo        # comandos de demo: setup, find-witness, prove, submit, claim

docs/
  demo-script.md        # roteiro de video e comandos
```

## 4. Demo escolhida: AMM invariant break

### Contrato alvo

Criar um AMM didatico em Soroban com reservas `reserve_x` e `reserve_y`.

Formula correta esperada:

```text
dy = reserve_y * dx / (reserve_x + dx)
```

Formula vulneravel proposital:

```text
dy_buggy = reserve_y * dx / reserve_x
```

Essa formula paga mais `y` do que deveria e pode reduzir o produto das reservas.

### Invariante publica

```text
(reserve_x + dx) * (reserve_y - dy_buggy) >= reserve_x * reserve_y
```

O pesquisador prova que conhece um `dx` privado que torna a invariante falsa:

```text
(reserve_x + dx) * (reserve_y - dy_buggy) < reserve_x * reserve_y
```

### Parametros de demo

Usar numeros pequenos para evitar complexidade de overflow no circuito:

```text
reserve_x, reserve_y, dx < 2^32
reward_amount = 100 token units
reputation_delta = 10
```

## 5. Circuito ZK

### Public inputs

```text
statement_hash
reserve_x
reserve_y
witness_commitment
nullifier
```

### Private inputs

```text
dx
salt
```

### Constraints

```text
1. dx > 0
2. reserve_x > 0
3. reserve_y > 0
4. reserve_x, reserve_y, dx < 2^32
5. witness_commitment == Poseidon(statement_hash, dx, salt)
6. nullifier == Poseidon(statement_hash, witness_commitment)
7. reserve_y * dx == dy_buggy * reserve_x + remainder
8. 0 <= remainder < reserve_x
9. dy_buggy < reserve_y
10. new_x == reserve_x + dx
11. new_y == reserve_y - dy_buggy
12. new_x * new_y < reserve_x * reserve_y
```

Division deve ser modelada por quotient/remainder, nao por divisao direta:

```text
dy_buggy = floor(reserve_y * dx / reserve_x)
reserve_y * dx == dy_buggy * reserve_x + remainder
0 <= remainder < reserve_x
```

O circuito deve ter testes positivos e negativos antes da integracao Soroban.

`statement_hash` e calculado pelo contrato e pela CLI a partir da metadata publica do bounty:

```text
statement_hash = Poseidon(
  bounty_id,
  target_wasm_hash,
  invariant_id,
  reserve_x,
  reserve_y
)
```

O circuito prova o invariant break para esse `statement_hash` e para essas reservas. Ele nao prova que executou a Soroban VM nem que analisou o WASM completo; essa e uma limitacao intencional do MVP.

## 6. Contrato BountyEscrow

### Interface

```rust
initialize(admin: Address)

create_bounty(
    owner: Address,
    target_contract: Address,
    target_wasm_hash: BytesN<32>,
    invariant_id: BytesN<32>,
    verifier: Address,
    reward_token: Address,
    reward_amount: i128,
    reserve_x: u64,
    reserve_y: u64,
    deadline: u64
) -> BytesN<32>

submit_proof(
    researcher: Address,
    bounty_id: BytesN<32>,
    proof: Bytes,
    public_inputs: Vec<BytesN<32>>,
    witness_commitment: BytesN<32>,
    nullifier: BytesN<32>
)

submit_report_hash(
    researcher: Address,
    bounty_id: BytesN<32>,
    report_hash: BytesN<32>
)

claim_reward(
    researcher: Address,
    bounty_id: BytesN<32>
)

get_bounty(bounty_id: BytesN<32>)
get_reputation(researcher: Address)
```

### Estados

```text
OPEN
PROOF_VERIFIED
REPORT_SUBMITTED
CLAIMED
CANCELLED
EXPIRED
```

`REWARD_LOCKED` nao precisa ser estado separado; e consequencia de `PROOF_VERIFIED` com `reserved_researcher` preenchido.

### Regras

```text
create_bounty:
  - owner autoriza transferencia da recompensa
  - contrato recebe/guarda reward_token
  - grava target_wasm_hash, invariant_id, verifier, reserves e deadline

submit_proof:
  - bounty precisa estar OPEN
  - deadline nao pode ter passado
  - nullifier precisa estar unused
  - contrato recalcula `statement_hash` a partir da metadata do bounty
  - public inputs precisam conter o `statement_hash`, `reserve_x` e `reserve_y` esperados
  - verifier(proof, public_inputs) precisa retornar true
  - grava nullifier como used
  - status = PROOF_VERIFIED
  - reserved_researcher = researcher

submit_report_hash:
  - somente reserved_researcher
  - status precisa ser PROOF_VERIFIED
  - grava report_hash
  - status = REPORT_SUBMITTED

claim_reward:
  - somente reserved_researcher
  - status precisa ser REPORT_SUBMITTED
  - transfere reward_token para researcher
  - reputation[researcher].score += 10
  - reputation[researcher].discoveries += 1
  - status = CLAIMED
```

### Eventos

```text
BountyCreated(bounty_id, target_contract, invariant_id, reward_amount)
ProofVerified(bounty_id, researcher, witness_commitment, nullifier)
RewardLocked(bounty_id, researcher, reward_amount)
ReportSubmitted(bounty_id, researcher, report_hash)
RewardClaimed(bounty_id, researcher, reward_amount)
ReputationUpdated(researcher, score, discoveries)
```

## 7. Verifier

O verifier deve ser tratado como componente de maior risco. Plano:

1. Primeiro, rodar Groth16 localmente fora da chain e provar que o circuito aceita/rejeita corretamente.
2. Depois, integrar o contrato Groth16 Soroban com uma proof fixa.
3. Por ultimo, conectar `BountyEscrow.submit_proof` ao verifier.

Durante o desenvolvimento inicial do escrow, e aceitavel usar um `MockVerifier` em testes unitarios. A demo final precisa ter pelo menos um caminho com proof real verificada em Soroban. Se isso falhar, o README/video deve declarar explicitamente o que ficou mockado.

## 8. Fluxos obrigatorios de demo

### Fluxo 1: bounty criado

1. Owner deploya AMM vulneravel.
2. Owner cria bounty para `constant_product_v1`.
3. Owner deposita recompensa.
4. CLI mostra status `OPEN`.

### Fluxo 2: proof valida

1. Researcher usa um `dx` secreto.
2. CLI gera proof sem imprimir `dx`.
3. Researcher submete proof.
4. Verifier passa.
5. Escrow muda para `PROOF_VERIFIED`.
6. Recompensa fica reservada para o researcher.

### Fluxo 3: proof invalida

1. CLI altera public input ou usa proof errada.
2. Verifier falha.
3. Escrow permanece `OPEN`.
4. Reputacao nao muda.

### Fluxo 4: duplicate bloqueado

1. Mesmo nullifier e submetido de novo.
2. Contrato rejeita.
3. CLI mostra `duplicate discovery`.

No MVP, cada bounty aceita apenas o primeiro proof valido porque o status sai de `OPEN`. O nullifier bloqueia replay/duplicata tecnica; deteccao de bugs semanticamente equivalentes fica fora do escopo.

### Fluxo 5: claim e reputacao

1. Researcher submete `report_hash`.
2. Researcher chama `claim_reward`.
3. Token e transferido.
4. Score sobe de `0` para `10`.

## 9. Ordem de implementacao

### Gate 0: validar tooling ZK/Soroban

Objetivo: reduzir risco antes de escrever muito codigo de produto.

1. Confirmar toolchain Stellar CLI/Rust/Soroban local.
2. Confirmar toolchain Circom/snarkjs ou alternativa Groth16 disponivel no ambiente.
3. Rodar exemplo minimo de Groth16 verifier Soroban, mesmo que com proof dummy do exemplo.
4. Decidir se a demo final roda em localnet ou testnet.

Saida esperada: comando documentado em `docs/demo-script.md` que verifica uma proof em Soroban.

### Gate 1: contratos sem ZK real

1. Scaffold Soroban workspace.
2. Implementar token mock ou integrar SAC local.
3. Implementar `BountyEscrow` com `MockVerifier`.
4. Testar create, submit valido, submit invalido, duplicate, report hash, claim e reputacao.

Saida esperada: testes unitarios do fluxo de bounty completos.

### Gate 2: AMM alvo

1. Implementar `vulnerable-amm`.
2. Adicionar teste que demonstra numericamente que existe `dx` quebrando a invariante.
3. Fixar parametros de demo (`reserve_x`, `reserve_y`, `dx`) para o circuito.

Saida esperada: teste Soroban do AMM vulneravel + valores de demo.

### Gate 3: circuito e proof

1. Implementar `amm_invariant.circom`.
2. Criar teste positivo com o `dx` secreto.
3. Criar testes negativos:
   - `dx = 0`;
   - public reserve alterado;
   - witness que nao quebra invariante;
   - nullifier/commitment inconsistente.
4. Gerar proof, verification key e calldata/public inputs.

Saida esperada: proof real local com public inputs finais.

### Gate 4: integracao on-chain real

1. Integrar Groth16 verifier Soroban.
2. Converter proof/public inputs para tipos aceitos pelo contrato.
3. Trocar `MockVerifier` por verifier real em `submit_proof`.
4. Medir custo/fee real da verificacao no ambiente usado.

Saida esperada: proof real aceita on-chain e proof invalida rejeitada.

### Gate 5: CLI e roteiro

1. Criar CLI de demo com comandos:
   - `setup`
   - `find-witness`
   - `prove`
   - `submit-proof`
   - `submit-invalid`
   - `submit-duplicate`
   - `submit-report`
   - `claim`
   - `status`
2. Escrever `docs/demo-script.md`.
3. Atualizar README com escopo, arquitetura e limitacoes honestas.

Saida esperada: demo reproduzivel em 2-3 minutos.

## 10. Criterios de sucesso

O MVP esta pronto se:

1. Bounty e criado on-chain.
2. Recompensa fica em escrow.
3. Proof ZK real e gerada off-chain.
4. Proof valida e verificada pelo contrato Soroban.
5. Proof invalida falha.
6. Nullifier impede duplicata.
7. Recompensa e reservada/liberada apos `report_hash`.
8. Reputacao muda on-chain.
9. README explica claramente que o MVP prova uma invariante, nao qualquer bug arbitrario.
10. O video mostra o papel load-bearing do ZK e da Stellar.

## 11. Riscos tecnicos e mitigacoes

### Verifier on-chain atrasar

Mitigacao:

- validar o exemplo Groth16 antes de expandir UI;
- manter circuito pequeno;
- manter `MockVerifier` apenas para desenvolvimento paralelo;
- medir custo real cedo e evitar claims fixos tipo "1 XLM" sem benchmark.

### Circuito aceitar witness invalido

Mitigacao:

- bounds pequenos;
- division por quotient/remainder;
- testes negativos obrigatorios;
- publicar `circuit_hash` e `verification_key_hash` no bounty ou README.

### Conversao proof/public inputs travar

Mitigacao:

- criar script unico de exportacao para o formato Soroban;
- congelar ordem dos public inputs;
- adicionar teste que compara hash/ordem dos inputs no CLI e no contrato.

### Demo parecer artificial

Mitigacao:

- usar bug financeiro intuitivo;
- mostrar invariant break numericamente so depois do reward lock/report hash;
- explicar que o produto comeca por templates de invariantes, nao por reports subjetivos.

### Critica de "nao prova Soroban VM"

Resposta:

**O MVP prova o modelo publico da funcao alvo. Provar a VM inteira seria pesquisa pesada. O produto comeca por invariant templates, que sao a parte objetiva e verificavel de muitos bug bounties financeiros.**

## 12. Roteiro de video tecnico

1. Mostrar bounty aberto: "AMM invariant: constant product must not decrease".
2. Mostrar AMM vulneravel e explicar a formula errada sem revelar o `dx`.
3. Rodar CLI: `find-witness`, sem imprimir o witness.
4. Rodar CLI: `prove`, gerando proof.
5. Submeter proof para Soroban.
6. Mostrar evento `ProofVerified`.
7. Mostrar reward locked/reserved.
8. Rodar caso invalido falhando.
9. Rodar duplicate falhando.
10. Submeter `report_hash`.
11. Fazer claim.
12. Mostrar leaderboard/reputation atualizado.

## 13. O que ainda falta definir

Nao falta decisao de produto para comecar a desenvolver. O escopo tecnico esta fechado para o MVP.

Faltam apenas validacoes de ambiente/tooling, que devem ser feitas no Gate 0:

1. Qual exemplo Groth16 Soroban sera usado como base concreta.
2. Se a demo final roda em localnet ou testnet.
3. Se o token de recompensa sera mock local ou SAC/USDC testnet.

Se qualquer uma dessas validacoes falhar, a decisao de produto nao muda; ajusta-se apenas o nivel de integracao demonstravel e isso deve ser declarado no README/video.
