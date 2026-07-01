# Development Plan: Hackathon MVP

## 1. Objetivo

Construir uma demo pequena, funcional e convincente do **Agent Payment Vault** para o hackathon da Stellar.

O MVP deve provar uma tese simples:

**Um agente de IA pode propor pagamentos em USDC, mas o dinheiro so sai do vault Soroban se a transacao respeitar limites publicos e provar, via ZK, que o destinatario pertence a uma whitelist privada.**

O foco nao e construir uma plataforma completa de compliance. O foco e demonstrar uma primitive aberta em Soroban para pagamentos autonomos com policy enforcement verificavel e privacidade de whitelist.

## 2. Arquitetura Escolhida Para O Hackathon

A arquitetura recomendada para o hackathon e uma versao **MVP-plus**: pequena o bastante para entregar, mas estruturada de forma que pareca extensivel.

Fluxo principal:

```text
AI Agent
  -> Payment Intent
      -> Local Prover / Policy Service
          -> ZK proof: recipient is in private whitelist
              -> Soroban Vault
                  -> checks public limits
                  -> verifies ZK proof
                  -> transfers USDC if valid
```

Componentes:

1. **AI Agent Demo**
   - Simula um agente recebendo uma tarefa de pagamento.
   - Exemplo: "pague 50 USDC ao fornecedor A".
   - O agente nao assina nem movimenta fundos diretamente.
   - Ele cria uma intent de pagamento.

2. **Policy Service / Prover Local**
   - Recebe a intent do agente.
   - Busca os dados privados necessarios para provar membership na whitelist.
   - Gera uma prova ZK.
   - Envia `to`, `amount` e `proof` para o vault.

3. **Soroban Vault Contract**
   - Guarda fundos.
   - Guarda parametros publicos da policy.
   - Guarda o commitment da whitelist privada, como `whitelist_root`.
   - Verifica limites publicos.
   - Verifica a prova ZK.
   - Executa ou bloqueia o pagamento.

4. **ZK Circuit**
   - Prova que o destinatario pertence a uma whitelist privada.
   - Usa como entrada privada o Merkle path do destinatario.
   - Usa como entradas publicas o `whitelist_root` e o destinatario comprometido.

5. **Demo CLI**
   - Executa os cenarios aprovados e bloqueados.
   - Mostra logs claros para o video.
   - E prioridade maior que dashboard.

## 3. O Que Fica On-chain E Off-chain

On-chain:

- Vault Soroban.
- Saldo do vault.
- Admin do vault.
- Token permitido.
- Limite maximo por pagamento.
- Limite diario.
- Total gasto no periodo atual.
- `whitelist_root`.
- Verificacao da prova.
- Eventos de pagamento aprovado ou bloqueado.

Off-chain:

- Agente de IA.
- Lista completa de destinatarios autorizados.
- Merkle tree.
- Merkle path.
- Geracao da prova ZK.
- Scripts de demo.
- Possivel dashboard futuro.

Essa separacao e importante para o pitch: o projeto nao afirma que tudo e descentralizado. Ele afirma que a decisao final de liberar fundos acontece no contrato.

## 4. Politicas Do MVP

O MVP deve implementar poucas regras, mas elas precisam ser claras.

### Regras publicas no contrato

1. **Ativo permitido**
   - O vault e configurado para um token especifico.
   - Para demo, usar USDC ou token Soroban mock equivalente.

2. **Limite por transacao**
   - Exemplo: nenhum pagamento acima de 100 USDC.
   - Valor acima do limite deve falhar mesmo que o destinatario esteja na whitelist.

3. **Limite diario**
   - Exemplo: maximo de 300 USDC por periodo.
   - Para simplificar, o "dia" pode ser uma janela baseada em ledger timestamp ou um periodo fixo configurado.

### Regra privada via ZK

1. **Whitelist privada**
   - O contrato guarda apenas o root da whitelist.
   - A lista completa nao fica on-chain.
   - A prova demonstra que o destinatario esta na lista sem revelar todos os membros.

## 5. Interface Do Vault

Interface sugerida do contrato:

```rust
initialize(admin, token, max_per_payment, daily_limit, whitelist_root)
deposit(from, amount)
withdraw(to, amount)
update_policy(max_per_payment, daily_limit, whitelist_root)
execute_payment(to, amount, proof)
get_policy()
get_spent_today()
```

Comportamento esperado de `execute_payment`:

1. Verificar se o vault foi inicializado.
2. Verificar se `amount <= max_per_payment`.
3. Verificar se `spent_today + amount <= daily_limit`.
4. Verificar se a prova ZK e valida contra `whitelist_root` e `to`.
5. Atualizar `spent_today`.
6. Transferir tokens para `to`.
7. Emitir evento de pagamento aprovado.

Em caso de falha, a transacao deve reverter ou emitir erro claro.

## 6. ZK Circuit Minimal

O circuito deve ser pequeno e load-bearing.

Prova desejada:

**"Eu conheco um caminho Merkle valido que prova que este destinatario pertence a arvore comprometida por este root."**

Entradas publicas:

- `whitelist_root`
- `recipient_commitment` ou representacao publica do destinatario

Entradas privadas:

- `recipient_secret` ou hash do endereco
- Merkle path
- Indices do path

Saida:

- `true` se o root reconstruido for igual ao `whitelist_root`.

Para hackathon, o circuito nao precisa provar limite de valor. O limite de valor pode ser regra publica no contrato. Isso reduz complexidade e mantem o ZK essencial.

## 7. Fluxos De Demo

A demo deve ter tres cenarios curtos.

### Cenario 1: Pagamento aprovado

Input:

- Agente recebe: "pague 50 USDC ao fornecedor A".
- Fornecedor A esta na whitelist privada.
- Valor esta abaixo do limite.

Resultado:

- Prova e gerada.
- Vault verifica prova e limites.
- Pagamento e executado.
- Evento mostra pagamento aprovado.

### Cenario 2: Destinatario nao autorizado

Input:

- Agente recebe: "pague 50 USDC ao endereco B".
- Endereco B nao esta na whitelist.
- Valor esta abaixo do limite.

Resultado:

- Prova valida nao pode ser gerada ou a prova falha.
- Vault bloqueia pagamento.
- Saldo permanece no vault.

### Cenario 3: Valor acima do limite

Input:

- Agente recebe: "pague 500 USDC ao fornecedor A".
- Fornecedor A esta na whitelist.
- Valor ultrapassa `max_per_payment`.

Resultado:

- Mesmo com destinatario autorizado, o contrato bloqueia o pagamento.
- Isso mostra que ZK nao substitui as regras publicas de policy.

## 8. O Que Nao Construir No MVP

Para manter o escopo viavel, nao construir no primeiro corte:

- Vault factory.
- Dashboard completo.
- Compliance AML/KYC/KYB completo.
- Travel Rule.
- KYT/sanctions screening.
- Multiplos tipos de prova.
- Politicas privadas complexas.
- Gestao multi-tenant.
- Integracao real com custodians.
- App mobile.

Esses itens podem entrar na visao de produto, mas nao devem bloquear a entrega do hackathon.

## 9. Criterios De Sucesso

O MVP esta bom o suficiente se:

1. Um vault Soroban segura fundos.
2. Um pagamento autorizado e executado.
3. Um pagamento para destinatario fora da whitelist e bloqueado.
4. Um pagamento acima do limite e bloqueado.
5. A prova ZK e necessaria para liberar o pagamento autorizado.
6. O README explica claramente o papel do ZK.
7. O video mostra o fluxo em 2 a 3 minutos.

## 10. Ordem De Implementacao

1. Scaffold do projeto Soroban.
2. Implementar token mock ou integrar token existente de testnet.
3. Implementar vault sem ZK, apenas com admin, deposito, saque e limite por pagamento.
4. Adicionar limite diario.
5. Criar script de demo para pagamento aprovado/bloqueado por limite.
6. Implementar Merkle tree off-chain.
7. Implementar circuito de membership.
8. Integrar verifier no contrato.
9. Atualizar `execute_payment` para exigir proof valida.
10. Criar demo final com tres cenarios.
11. Escrever README e roteiro do video.

## 11. Narrativa Para O Video

Roteiro sugerido:

1. "Agentes de IA podem mover dinheiro, mas uma wallet normal da poder demais ao agente."
2. "Nosso vault em Soroban segura os fundos e aplica politicas antes de liberar qualquer pagamento."
3. "A whitelist de fornecedores e privada. O contrato conhece apenas o root."
4. "Quando o agente tenta pagar um fornecedor aprovado, geramos uma prova ZK e o contrato libera o pagamento."
5. "Quando o agente tenta pagar um endereco fora da whitelist, o pagamento falha."
6. "Quando o agente tenta pagar acima do limite, o contrato tambem bloqueia."
7. "O resultado e uma primitive aberta para agentic payments seguros e privacy-preserving na Stellar."

## 12. Frase De Escopo

**Hackathon scope: private-whitelist Agent Payment Vault on Stellar.**

Descricao curta:

**A Soroban vault that lets AI agents propose USDC payments, but only executes when public spending limits pass and a ZK proof confirms the recipient belongs to a private whitelist.**
