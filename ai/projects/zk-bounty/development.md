# Development Plan: ZK-Bounty Hackathon MVP

## 1. Objetivo

Construir uma demo funcional da ZK-Bounty com a seguinte tese:

**Um pesquisador encontra um input privado que quebra uma invariante de um contrato Soroban. Ele gera uma prova ZK sem revelar o input. Um contrato Soroban verifica a prova, reserva a recompensa e atualiza a reputacao on-chain.**

O MVP nao tenta provar execucao arbitraria da Soroban VM. Ele prova uma transicao pequena, publica e modelada em circuito, suficiente para demonstrar o mecanismo de bounty verificavel.

## 2. Arquitetura

```text
Protocol owner
  -> create_bounty(invariant, target, reward)
      -> BountyEscrow Soroban contract
          -> stores reward and verifier reference

Researcher
  -> finds private witness
  -> generates ZK proof locally
  -> submit_proof(proof, public_inputs)
      -> BountyEscrow
          -> calls Verifier
          -> locks reward
          -> updates reputation
```

Componentes:

1. **Target Soroban Contract**
   - Contrato demonstrativo com bug financeiro.
   - Exemplo recomendado: AMM com formula de swap vulneravel.

2. **Invariant Circuit**
   - Circuito Circom ou Noir que replica a transicao vulneravel.
   - Prova que existe um input privado que viola a invariante.

3. **Verifier Contract**
   - Contrato Soroban que verifica Groth16 ou UltraHonk.
   - Para menor risco, priorizar Groth16 se o starter estiver funcionando.

4. **Bounty Escrow Contract**
   - Guarda recompensa em token Soroban.
   - Guarda metadata do bounty.
   - Recebe e verifica proof.
   - Reserva ou libera recompensa.
   - Atualiza reputacao.

5. **Researcher CLI**
   - Encontra ou recebe o witness secreto.
   - Gera proof localmente.
   - Submete proof.
   - Imprime status claro para video.

6. **Demo UI opcional**
   - Lista bounty aberto.
   - Mostra status `OPEN`, `PROOF_VERIFIED`, `REWARD_LOCKED`, `CLAIMED`.
   - Mostra leaderboard simples.

## 3. Demo escolhida: AMM invariant bounty

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

O pesquisador deve provar que conhece um `dx` privado que torna a invariante falsa:

```text
(reserve_x + dx) * (reserve_y - dy_buggy) < reserve_x * reserve_y
```

### Por que essa demo e boa

- E financeira e facil de visualizar.
- O bug e realista o suficiente para DeFi.
- A invariante e objetiva.
- O input do exploit fica privado.
- A prova e pequena o bastante para MVP.
- O contrato de bounty e claramente Stellar-native.

## 4. Circuito ZK

### Public inputs

```text
bounty_id
target_wasm_hash
invariant_id
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
4. witness_commitment == Poseidon(dx, salt, bounty_id)
5. nullifier == Poseidon(witness_commitment, bounty_id)
6. dy_buggy == floor(reserve_y * dx / reserve_x)
7. dy_buggy < reserve_y
8. new_x == reserve_x + dx
9. new_y == reserve_y - dy_buggy
10. new_x * new_y < reserve_x * reserve_y
```

Para evitar complexidade de overflow, usar bounds pequenos no MVP:

```text
reserve_x, reserve_y, dx < 2^32
```

Para division em circuito, representar:

```text
reserve_y * dx == dy_buggy * reserve_x + remainder
0 <= remainder < reserve_x
```

## 5. Contrato BountyEscrow

Interface sugerida:

```rust
initialize(admin: Address, reputation: Address)
create_bounty(
    owner: Address,
    target_contract: Address,
    target_wasm_hash: BytesN<32>,
    invariant_id: BytesN<32>,
    verifier: Address,
    reward_token: Address,
    reward_amount: i128,
    deadline: u64
)
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
claim_reward(researcher: Address, bounty_id: BytesN<32>)
get_bounty(bounty_id: BytesN<32>)
```

Estados:

```text
OPEN
PROOF_VERIFIED
REWARD_LOCKED
REPORT_SUBMITTED
CLAIMED
CANCELLED
EXPIRED
```

Eventos:

```text
BountyCreated
ProofVerified
RewardLocked
ReportSubmitted
RewardClaimed
ReputationUpdated
```

## 6. Reputacao

Para MVP, reputacao deve ser simples e nao-transferivel.

Contrato ou storage no escrow:

```text
score[address] += 10 quando proof valida e report_hash existe
discoveries[address] += 1
last_discovery[address] = ledger_timestamp
```

Nao incluir:

- token transferivel;
- marketplace de reputacao;
- staking de reputacao;
- multiplicador automatico de recompensa.

Esses itens sao visao futura.

## 7. Fluxos de demo

### Fluxo 1: bounty criado

1. Owner cria bounty para AMM vulneravel.
2. Deposita recompensa em token Soroban.
3. UI mostra invariante e recompensa.

### Fluxo 2: proof valida

1. Researcher usa um `dx` secreto.
2. CLI gera proof.
3. Researcher submete proof.
4. Verifier passa.
5. Escrow muda para `PROOF_VERIFIED`.
6. Recompensa fica reservada para o researcher.

### Fluxo 3: proof invalida

1. Researcher tenta enviar proof errada ou public inputs alterados.
2. Verifier falha.
3. Escrow permanece `OPEN`.
4. Reputacao nao muda.

### Fluxo 4: duplicate bloqueado

1. Mesmo nullifier tenta ser submetido de novo.
2. Contrato rejeita.
3. UI mostra `duplicate discovery`.

### Fluxo 5: claim e reputacao

1. Researcher submete hash do report.
2. Researcher chama `claim_reward`.
3. Token e transferido.
4. Score sobe de `0` para `10`.

## 8. Ordem de implementacao

1. Scaffold Soroban workspace.
2. Implementar token mock ou usar SAC testnet.
3. Implementar `BountyEscrow` sem verifier, usando flag fake apenas em teste local.
4. Implementar `Reputation` simples.
5. Criar AMM vulneravel e teste que encontra um `dx` que quebra invariante.
6. Implementar circuito local.
7. Gerar proof e verification key.
8. Integrar verifier Soroban.
9. Conectar `submit_proof` ao verifier.
10. Criar CLI de demo.
11. Criar README e roteiro de video.

## 9. Path ZK recomendado

### Path A: Circom + Groth16

Recomendado para hackathon se o foco for reduzir risco.

Motivos:

- Existe exemplo de Groth16 verifier em Soroban.
- Stellar Private Payments usa Circom + Groth16 em Soroban.
- Groth16 tem proof pequena e verificacao eficiente.

Trade-off:

- Circom e mais verboso.
- Division e comparacoes precisam ser implementadas com cuidado.

### Path B: Noir + UltraHonk

Bom se o time ja tiver verifier UltraHonk rodando.

Motivos:

- Linguagem mais ergonomica.
- Melhor para iterar no circuito.
- Protocol 26 melhora a viabilidade de verificacao.

Trade-off:

- Integracao on-chain pode consumir mais tempo.
- Nao escolher sem testar verifier primeiro.

## 10. Criterios de sucesso

O MVP esta pronto se:

1. Bounty e criado on-chain.
2. Recompensa fica em escrow.
3. Proof valida e verificada pelo contrato.
4. Proof invalida falha.
5. Nullifier impede duplicata.
6. Recompensa e reservada/liberada.
7. Reputacao muda on-chain.
8. README explica claramente que o MVP prova uma invariante, nao qualquer bug arbitrario.

## 11. Riscos tecnicos

### Verifier on-chain atrasar

Mitigacao:

- Comecar pelo exemplo Groth16.
- Manter circuito pequeno.
- Medir custo real cedo.

### Circuito aceitar witness invalido

Mitigacao:

- Testar casos positivos e negativos.
- Fixar bounds.
- Evitar arithmetic complexa demais.
- Publicar circuit hash no bounty.

### Demo parecer artificial

Mitigacao:

- Usar bug financeiro intuitivo.
- Mostrar invariant break numericamente apos claim.
- Explicar que o produto generaliza para templates de invariantes.

### Critica de "nao prova Soroban VM"

Resposta:

**O MVP prova o modelo publico da funcao alvo. Provar a VM inteira seria pesquisa pesada. O produto comeca por invariant templates, como bug bounties objetivos para propriedades financeiras.**

## 12. Roteiro de video tecnico

1. Mostrar bounty aberto: "AMM invariant: constant product must not decrease".
2. Mostrar AMM vulneravel com formula errada.
3. Rodar CLI: `find-witness`, sem revelar `dx`.
4. Rodar CLI: `prove`, gerando proof.
5. Submeter proof para Soroban.
6. Mostrar evento `ProofVerified`.
7. Mostrar reward locked.
8. Submeter `report_hash`.
9. Fazer claim.
10. Mostrar leaderboard com score atualizado.
