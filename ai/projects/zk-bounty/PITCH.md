# ZK-Bounty - Pitch e Respostas Para Juizes

## 1. One-liner

**Provable bug bounties for Soroban invariants.**

## 2. Pitch curto

Protocolos Soroban precisam de seguranca continua, mas bug bounty tradicional depende de uma troca ruim: o pesquisador precisa revelar o exploit para provar que merece pagamento, enquanto o protocolo precisa confiar que o report e real antes de travar dinheiro.

ZK-Bounty resolve a parte objetiva desse problema. O protocolo publica uma invariante verificavel e deposita uma recompensa. O pesquisador prova em zero-knowledge que conhece um input privado que quebra essa invariante. Soroban verifica a prova, trava a recompensa e registra a descoberta como reputacao on-chain.

Nao e uma plataforma generica de reports. E uma camada de bounties verificaveis para bugs formalizaveis em contratos Soroban.

## 3. Script de 3 minutos

### 0:00 - 0:25, problema

"Todo protocolo DeFi tem invariantes: reservas nao podem ser criadas do nada, supply precisa bater com balances, um AMM nao deve perder valor por causa de uma formula errada. Quando um pesquisador encontra um bug, ele precisa provar que o bug existe sem entregar o exploit de graca."

### 0:25 - 0:55, solucao

"ZK-Bounty transforma esse momento em uma prova. O protocolo cria um bounty para uma invariante Soroban e deposita USDC. O pesquisador gera uma prova ZK local: eu conheco um input secreto que quebra esta invariante. O contrato Soroban verifica a prova e reserva a recompensa antes do exploit ser revelado."

### 0:55 - 1:45, demo

"Aqui temos um AMM Soroban vulneravel. A invariante publica e que o produto das reservas nao deve diminuir. O pesquisador encontra um trade size secreto que quebra essa regra. Agora ele gera a proof. O input nao aparece. Enviamos a proof para Soroban. O verifier passa. O bounty muda para `PROOF_VERIFIED`, a recompensa fica locked para o pesquisador e a reputacao dele sobe."

### 1:45 - 2:20, caso negativo

"Se eu alterar os public inputs, usar outra invariante ou tentar submeter a mesma descoberta duas vezes, o contrato rejeita. Isso reduz report invalido e duplicata para a parte objetiva do bounty."

### 2:20 - 3:00, visao

"Bug bounties tradicionais continuam importantes para bugs subjetivos. ZK-Bounty foca no que pode ser formalizado: invariantes de contratos Soroban. A longo prazo, protocolos publicam invariant bounties permanentes, pesquisadores constroem reputacao verificavel e a seguranca do ecossistema vira uma camada publica, com pagamentos em Stellar."

## 4. Demo visual recomendada

Tela 1: `Bounty`

```text
Target: Vulnerable Soroban AMM
Invariant: new_x * new_y >= old_x * old_y
Reward: 100 USDC
Status: OPEN
```

Tela 2: `Researcher CLI`

```text
Finding private witness...
Witness found.
Generating ZK proof...
Proof generated.
Submitting proof to Soroban...
Proof verified.
Reward locked.
```

Tela 3: `Leaderboard`

```text
Researcher: G...
Valid discoveries: 1
Score: 10
Last bounty: AMM invariant break
```

## 5. Perguntas dificeis e respostas

### "Isso prova qualquer vulnerabilidade?"

Nao. O MVP prova uma classe especifica: quebra de invariante em um harness publico. Esse recorte e proposital porque torna o bounty objetivo e verificavel. Bugs subjetivos continuam precisando de triagem.

### "Entao nao e uma plataforma completa como Immunefi?"

Correto. E uma primitive complementar. Immunefi coordena reports humanos. ZK-Bounty automatiza o primeiro passo quando a propriedade e formalizavel: provar que existe um contraexemplo sem revelar o exploit.

### "Por que precisa de ZK?"

Porque o pesquisador precisa convencer o protocolo sem revelar o input. Hash simples so prova que ele se comprometeu com algo; ZK prova que esse algo realmente quebra a invariante.

### "Por que Stellar?"

Porque o alvo e Soroban, a prova e verificada em Soroban, a recompensa fica em Stellar Asset Contract e a reputacao vive no ledger. Alem disso, Protocol 25 e Protocol 26 adicionam primitivas ZK-friendly como BN254 e Poseidon/Poseidon2, melhorando a viabilidade de verifiers on-chain.

### "Como voces evitam report falso?"

O contrato aceita apenas proof valida contra a verification key do bounty, os public inputs do bounty e o invariant id. Se a prova nao satisfaz o circuito, nao ha reward lock nem reputacao.

### "Como evitam replay ou duplicata?"

Cada proof publica um `nullifier` vinculado ao `bounty_id` e ao `witness_commitment`. O contrato rejeita nullifiers ja usados.

### "O pesquisador recebe antes de revelar o exploit?"

No desenho responsavel, a proof valida reserva ou trava a recompensa. O claim final acontece apos hash de report, confirmacao do protocolo ou timeout. Para a demo, o report hash pode ser suficiente.

### "E se o circuito do bounty estiver errado?"

Esse e o principal risco. Por isso o bounty publica `circuit_hash`, `verification_key_hash` e usa templates pequenos de invariantes no MVP. Em producao, esses templates precisam ser auditados.

## 6. Frase para competir contra Payment Vault

**Payment Vault mostra policy enforcement para pagamentos. ZK-Bounty mostra descoberta verificavel de bugs em Soroban. Para um hackathon de Real-World ZK, a segunda narrativa faz o ZK carregar mais peso: sem a prova, o pesquisador precisa revelar o exploit; com a prova, a recompensa pode ser garantida sem expor o bug.**

## 7. Mensagem final

**ZK-Bounty turns vulnerability discovery into a verifiable on-chain object: a private exploit witness, a public invariant, a Soroban-verified proof, a locked USDC reward, and durable researcher reputation.**
