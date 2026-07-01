# Agent Payment Vault: Compliance ZK para Agentes Autonomos na Stellar

## 1. Visao Geral

O **Agent Payment Vault** e uma infraestrutura de seguranca para agentes de IA que precisam mover stablecoins, RWAs ou outros ativos na rede Stellar sem receber controle irrestrito sobre uma carteira.

A ideia central e simples: o agente pode propor pagamentos, mas o dinheiro fica em um cofre Soroban que so executa a transacao se ela respeitar politicas definidas pelo gestor. Essas politicas podem incluir limite por transacao, limite diario, ativo permitido, destinatarios aprovados e aprovacao humana para valores acima de certo patamar.

Quando a politica precisa permanecer privada, uma prova de conhecimento zero pode ser usada para demonstrar que a transacao e valida sem revelar todos os detalhes da regra, da whitelist ou da operacao.

## 2. Dor Que O MVP Resolve

Empresas, DAOs, fintechs e times de produto estao comecando a usar agentes autonomos para executar tarefas financeiras: pagar APIs, liquidar servicos, mover USDC, fazer repasses, operar fluxos cross-border e interagir com contratos.

O problema e que uma wallet tradicional da poder demais ao agente. Se o agente alucinar, for manipulado por prompt injection, tiver um bug ou for comprometido, ele pode enviar fundos para o destino errado, gastar alem do combinado ou drenar a tesouraria.

Hoje, a alternativa comum e manter validacoes em backends centralizados, aprovacoes manuais ou scripts internos. Isso reduz automacao, cria dependencia de logs privados e nao oferece uma garantia verificavel na propria rede.

O Agent Payment Vault resolve essa dor colocando a autoridade final no contrato:

- O agente nao controla diretamente os fundos.
- O contrato so libera pagamentos que passam pelas regras.
- Cada execucao gera um rastro auditavel on-chain.
- Politicas sensiveis podem ser comprovadas sem serem expostas integralmente.
- Pagamentos pequenos e recorrentes podem ser automatizados sem abrir mao de governanca.

## 3. MVP Para Hackathon

O MVP deve demonstrar um fluxo completo e facil de entender:

1. Um gestor deposita USDC em um vault Soroban.
2. O gestor configura uma politica de uso:
   - ativo permitido: USDC;
   - limite por transacao;
   - limite diario;
   - lista de destinatarios aprovados;
   - aprovacao humana opcional para valores altos.
3. Um agente de IA tenta executar um pagamento.
4. Um policy prover valida a intencao do agente e gera uma prova ou comprovacao verificavel.
5. O contrato Soroban verifica a autorizacao.
6. Se a politica for respeitada, o vault libera o pagamento.
7. Se a politica for violada, a transacao e bloqueada.

Esse fluxo permite apresentar uma demo clara: um agente tenta pagar um fornecedor aprovado dentro do limite e consegue; depois tenta mandar um valor acima do limite ou para um endereco nao autorizado e o contrato bloqueia.

## 4. Onde Entra ZK

O MVP nao deve usar ZK apenas como palavra de efeito. O uso mais forte e provar conformidade sem revelar informacoes sensiveis.

Casos praticos:

- **Whitelist privada:** provar que o destinatario pertence a uma lista aprovada sem publicar todos os enderecos da lista.
- **Limite privado:** provar que o valor esta dentro de uma politica configurada sem revelar a politica completa.
- **Credencial de fornecedor:** provar que o recebedor possui uma credencial valida de fornecedor, parceiro ou entidade KYC/KYB sem expor dados desnecessarios.
- **Politica proprietaria:** permitir que uma empresa automatize regras internas sem transformar toda a governanca em informacao publica.

Para uma primeira demo, o caminho mais viavel e combinar regras publicas simples no contrato com uma prova ZK focada em um ponto especifico, como membership em whitelist privada via commitment/Merkle root.

## 5. Por Que Stellar

A proposta conversa bem com Stellar porque a rede e naturalmente forte em pagamentos, stablecoins, ativos tokenizados e fluxos cross-border.

O projeto usa Stellar nao apenas como infraestrutura generica de smart contracts, mas como trilho financeiro:

- **Soroban** implementa o vault e a verificacao das politicas.
- **USDC/stablecoins** tornam o caso de uso facil de demonstrar.
- **Baixo custo e liquidacao rapida** permitem pagamentos frequentes por agentes.
- **RWA e anchors** abrem caminho para casos institucionais e cross-border.
- **Agentic payments** ficam mais seguros quando o agente opera sob politicas verificaveis.

## 6. Panorama Competitivo E Posicionamento

Essa ideia nao deve ser apresentada como algo que nao existe em nenhuma forma. O mercado ja esta atacando a dor de wallets seguras para agentes.

Exemplos relevantes:

- **Circle Agent Stack:** oferece agent wallets, spending controls, compliance guardrails e limites de uso para agentes que movimentam stablecoins.
- **Turnkey:** oferece agentic wallets com politicas por endereco, contrato, escopo de permissao e limites de gasto.
- **Fireblocks Agentic Payments:** oferece uma solucao enterprise para pagamentos iniciados por agentes, com policy engine, KYT, Travel Rule e audit trail.
- **x402 / Stellar x402:** permite que agentes paguem APIs e servicos usando stablecoins, incluindo fluxos sobre Stellar.
- **zkMe, Privado ID e projetos de ZK identity:** resolvem partes de identidade, KYC/KYB e credenciais privadas, mas nao sao um vault de pagamento Stellar-native.

Portanto, o pitch correto nao e "ninguem faz wallets seguras para agentes". Esse espaco ja existe.

O posicionamento mais defensavel e:

**"Ja existem agent wallets com policy controls, mas ainda falta uma camada aberta, Stellar-native e privacy-preserving para enforcement de politicas de pagamento em Soroban."**

O diferencial do Agent Payment Vault esta no recorte:

- enforcement de politicas diretamente em um contrato Soroban;
- foco em pagamentos, USDC, stablecoins e RWAs na Stellar;
- transparencia e auditabilidade on-chain;
- privacidade opcional com ZK para whitelists, credenciais e politicas sensiveis;
- composabilidade com agentes, x402 e apps que precisam mover dinheiro de forma programatica.

Em outras palavras: Turnkey, Circle e Fireblocks validam a dor. O projeto se diferencia por ser uma implementacao aberta, demonstravel em Stellar e com privacy-preserving compliance como parte nativa do design.

A diferenca principal esta em **onde a confianca fica**. Nas plataformas atuais, a politica geralmente e aplicada pela infraestrutura da propria plataforma: uma wallet gerenciada, um policy engine custodial, um enclave, um backend ou uma suite enterprise. No Agent Payment Vault, a regra critica vive no contrato Soroban que segura os fundos. O contrato e ao mesmo tempo o cofre e o juiz.

Comparacao direta:

| Solucao | O que faz | Diferenca do Agent Payment Vault |
| --- | --- | --- |
| Circle Agent Stack | Agent wallets, spending controls, compliance guardrails e pagamentos com stablecoins. | E uma stack de wallets e infraestrutura da Circle. O Agent Payment Vault seria um vault aberto em Soroban, Stellar-native, com enforcement on-chain e ZK opcional. |
| Turnkey | Wallets programaveis com policy engine, limites, permissoes e key management. | Forte em custody/key management, mas o enforcement depende da infraestrutura da Turnkey. No Agent Payment Vault, o contrato segura o dinheiro e aplica a regra na Stellar. |
| Fireblocks Agentic Payments | Solucao enterprise para pagamentos iniciados por agentes, com KYT, Travel Rule, audit trail e policy engine. | Mais maduro para instituicoes, mas fechado e enterprise. O Agent Payment Vault e uma primitive aberta e composavel on-chain. |
| x402 / Stellar x402 | Protocolo para agentes pagarem APIs e servicos usando stablecoins. | x402 resolve o "como pagar". O Agent Payment Vault resolve "quem pode pagar, quanto, para quem e sob qual politica". |
| zkMe / Privado ID | ZK identity, KYC/KYB e credenciais privadas. | Eles provam identidade ou credencial. O Agent Payment Vault usa esse tipo de prova para liberar ou bloquear fundos em um vault. |

Resumo do posicionamento:

**As plataformas atuais dao wallets com controles. O Agent Payment Vault propoe uma camada de execucao: o dinheiro fica preso em Soroban e so sai se a politica for verificavel.**

Isso nao significa que o projeto seja mais completo que Circle, Turnkey ou Fireblocks. Essas plataformas sao mais maduras em custody, UX enterprise, integracoes e compliance regulatorio. Para o hackathon, o objetivo e mostrar uma primitive aberta em Soroban: **policy enforcement verificavel e privado para pagamentos autonomos na Stellar**.

## 7. Clientes E Usuarios Alvo

O primeiro ICP para hackathon nao precisa ser um hedge fund. O caso inicial deve ser mais direto:

- **Fintechs e apps de pagamento:** agentes que automatizam repasses, refunds, cobrancas ou pagamentos cross-border.
- **DAOs e tesourarias on-chain:** automacao de pagamentos recorrentes com limites e destinatarios aprovados.
- **Marketplaces B2B:** agentes pagando fornecedores, APIs ou prestadores de servico.
- **Empresas com stablecoin treasury:** times que querem automatizar operacoes sem entregar uma chave livre ao agente.
- **Projetos de agentic commerce:** agentes que compram servicos digitais, dados, compute ou chamadas de API.

## 8. Diferenciais

- **Seguranca por design:** o agente nunca tem permissao irrestrita para mover fundos.
- **Governanca verificavel:** regras criticas sao aplicadas pelo contrato, nao apenas por um backend.
- **Privacidade opcional com ZK:** compliance pode ser provado sem revelar toda a politica.
- **Demo objetiva:** sucesso e falha podem ser mostrados em poucos minutos.
- **Aderencia ao ecossistema Stellar:** foco em pagamentos, stablecoins e ativos reais.

## 9. Pontos Negativos E Riscos

A ideia tem um recorte defensavel para um hackathon da Stellar, mas tambem tem fragilidades importantes que precisam ser reconhecidas no pitch e no escopo do MVP.

1. **Concorrencia forte e bem financiada**

   Circle, Turnkey e Fireblocks ja estao atacando agent wallets, policy controls e pagamentos por agentes. Mesmo que o recorte Stellar/Soroban/ZK seja diferente, o problema geral ja tem players muito mais maduros.

2. **Diferencial pode parecer pequeno se o ZK nao for convincente**

   Se o MVP tiver apenas limite por transacao, limite diario e whitelist publica, ele pode parecer um smart contract comum de permissoes. Para justificar "ZK compliance", a demo precisa mostrar uma prova realmente util, como whitelist privada, credencial privada ou politica sensivel verificada sem exposicao total.

3. **Soroban ainda tem ecossistema menor que EVM**

   Para um hackathon da Stellar isso e aceitavel e ate desejavel, mas como produto real pode ser uma desvantagem: menos tooling, menos bibliotecas prontas, menos integracoes com wallets, agentes e infraestrutura institucional.

4. **Compliance real nao e so regra on-chain**

   Um contrato consegue bloquear valores, ativos e destinatarios, mas nao resolve sozinho AML, KYC, KYB, sancoes, Travel Rule, fraude, chargeback, auditoria regulatoria ou responsabilidade legal. O pitch precisa evitar a mensagem de que "ZK resolve compliance" de forma ampla.

5. **O agente ainda depende de infraestrutura off-chain**

   O agente, o prover, o dashboard e a configuracao das politicas continuam fora da blockchain. Se o prover for centralizado ou mal implementado, parte da confianca continua fora do contrato. O MVP precisa deixar claro quais garantias estao on-chain e quais continuam off-chain.

6. **Politicas privadas podem conflitar com auditoria**

   Empresas querem privacidade, mas reguladores, auditores e times internos podem exigir explicabilidade. Em alguns contextos, "a prova passou" nao basta; alguem precisa conseguir revisar a regra, os dados e o motivo da aprovacao.

7. **Pode ser resolvido de forma mais simples em muitos casos**

   Para varias empresas, uma wallet multisig, uma policy engine centralizada, uma conta programavel ou um backend com aprovacao humana ja resolvem suficientemente bem. O projeto precisa escolher casos onde on-chain + ZK tragam uma vantagem clara, como whitelist privada, compliance auditavel ou automacao de pagamentos por agentes sem chave irrestrita.

8. **Responsabilidade de custody e bugs e alta**

   Se o vault segura dinheiro, qualquer bug no contrato, erro de configuracao ou falha de atualizacao vira risco financeiro direto. Isso aumenta a exigencia de auditoria, testes e controles de emergencia, mesmo em um MVP.

9. **Mercado de agentic payments ainda e inicial**

   A tese e promissora, mas a demanda real ainda esta se formando. Pode ser cedo demais para um produto B2B amplo, especialmente fora de nichos como pagamentos de APIs, compute, cross-border, tesouraria com stablecoins e automacao de repasses.

Os tres riscos mais importantes para o hackathon sao: ZK parecer desnecessario, a narrativa ser engolida por players maiores de agent wallets e o MVP tentar cobrir compliance demais. A melhor mitigacao e focar em uma demo especifica: **whitelist privada para pagamentos de agentes em USDC na Stellar**.

## 10. Modelo De Produto

O produto final pode evoluir para uma camada B2B de governanca para agentic payments:

- **Vaults por agente:** cada agente recebe um escopo de permissao.
- **Dashboard de politicas:** gestores definem limites, whitelists, ativos e niveis de aprovacao.
- **API para agentes:** agentes enviam intents de pagamento em vez de transacoes diretas.
- **Relatorios de compliance:** historico auditavel de execucoes aprovadas e bloqueadas.
- **Fee por volume ou por vault:** monetizacao alinhada ao valor protegido.

## 11. Escopo Tecnico Do MVP

Componentes recomendados:

1. **Soroban Vault Contract**
   - deposito e saque por administrador;
   - configuracao de policy;
   - execucao condicional de pagamento;
   - eventos de pagamento aprovado/bloqueado.

2. **Policy Layer**
   - validacao de limite por transacao;
   - controle de limite diario;
   - validacao de destinatario autorizado;
   - geracao de proof/comprovacao.

3. **ZK Proof Minimal**
   - prova de que um destinatario pertence a uma whitelist privada;
   - root/commitment registrado no contrato;
   - proof enviada junto com a intencao de pagamento.

4. **Agent Demo**
   - agente recebe uma tarefa como "pague 50 USDC ao fornecedor X";
   - agente monta a intencao;
   - prover gera a autorizacao;
   - vault executa ou bloqueia.

5. **Dashboard Simples**
   - saldo do vault;
   - politicas configuradas;
   - pagamentos aprovados;
   - tentativas bloqueadas.

## 12. Pitch

**Agentes de IA vao mover dinheiro. O problema e que uma wallet normal da poder demais ao agente. O Agent Payment Vault transforma uma carteira Stellar em um cofre programavel: o agente pode propor pagamentos, mas so o contrato decide se eles respeitam as politicas de compliance. Com ZK, a empresa pode provar que a regra foi cumprida sem revelar toda a sua politica interna.**

**O projeto nao tenta competir como mais uma wallet custodial para agentes. Ele propoe uma camada aberta de policy enforcement em Soroban, usando Stellar como trilho de pagamento e ZK para privacidade de compliance.**

Uma frase curta para apresentacao:

**"Safe policy vaults for autonomous agents on Stellar."**

## 13. Proximos Passos

1. Definir a policy inicial: limite por transacao, limite diario e whitelist.
2. Implementar o vault Soroban com regras publicas basicas.
3. Adicionar uma prova ZK simples para whitelist privada.
4. Criar uma demo com dois fluxos: pagamento aprovado e pagamento bloqueado.
5. Preparar pitch focado em agentic payments, USDC e compliance verificavel.
