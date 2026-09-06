# togglemaster-secrets-generator

Workflow do GitHub Actions para criar/atualizar secrets no AWS Secrets Manager do projeto ToggleMaster,
disparado através da abertura de uma issue.

### Como usar

Vá em `Issues` > `New issue` > escolha o template `🔑 Secrets Generator` e preencha os 3 campos:

`Environment` - O environment onde o secret será criado: `dev`

`Secret Name` - Deve seguir o padrão `<prefixo>/<componente>/<secret-name>`, por exemplo: `togglemaster-dev/evaluation/service-api-key`

`Secret Value` - O valor do secret, ou seja, a senha, API key, etc.

Clique em `Submit new issue`. O workflow é disparado pelo prefixo `[secrets-generator]` no título
(preenchido automaticamente pelo template) e sempre encerra a issue ao final, com um comentário de
sucesso ou de falha, e ofusca o valor do secret no corpo da issue.

### Contrato atual do projeto

Os secrets de runtime do ToggleMaster nao sao provisionados diretamente pelo Terraform. O contrato atual e:

- `DB_PASSWORD`: criado/consumido no bootstrap do Terraform como `TF_VAR_db_password`.
- `togglemaster-dev/auth/db-url`: URL do banco auth.
- `togglemaster-dev/auth/master-key`: chave mestre do auth-service.
- `togglemaster-dev/flag/db-url`: URL do banco flag.
- `togglemaster-dev/targeting/db-url`: URL do banco targeting.
- `togglemaster-dev/evaluation/redis-url`: URL do Redis.
- `togglemaster-dev/evaluation/service-api-key`: chave de servico do evaluation-service.
- `togglemaster-dev/evaluation/sqs-url`: URL da fila SQS, usada por evaluation e analytics.
- `togglemaster-dev/evaluation/sqs-arn`: ARN da fila SQS, reservado para consumidores que o exigirem.
- `togglemaster-dev/analytics/dynamodb-table-name`: nome da tabela DynamoDB, usado pelo analytics-service.

Depois do `terraform apply`, obtenha os endpoints e identificadores nos outputs do IaC e crie ou
atualize cada secret pela issue form. Esse padrao garante que todos os valores de runtime
permaneçam no Secrets Manager e sejam consumidos pelos workloads sem depender de inputs do IaC.

### Segurança

- O valor do secret fica visível no corpo da issue até o workflow concluir a execução (poucos segundos),
  quando então é substituído por `***REDACTED***`. Restrinja quem pode abrir issues neste repositório
  (repositório privado/interno, `Settings > General > Features > Issues`) para evitar exposição do valor.
- O log do workflow mascara o valor do secret (`::add-mask::`), mas evite reabrir a issue ou reutilizá-la
  para novos valores.

### Pré-requisitos

O environment `dev` do repositório precisa ter configurado:

- `vars.AWS_ROLE_TO_ASSUME` - ARN da role IAM assumida via OIDC (precisa de permissão `secretsmanager:CreateSecret`, `secretsmanager:UpdateSecret`, `secretsmanager:DescribeSecret`).
- `vars.AWS_REGION` - região da conta AWS do environment.

A role precisa confiar no provider OIDC do GitHub Actions (`token.actions.githubusercontent.com`) para o repositório `togglemaster-secrets-generator`.

A trust policy da role em `togglemaster-bootstrap-ci-iam` usa a condição
`token.actions.githubusercontent.com:sub = repo:<owner>/<repo>:environment:<env>`. Por isso o workflow
tem um job `create_secret_dev` com `environment: dev` — o claim `:environment:dev` do token OIDC só é incluído
quando o nome do environment é estático no job.

### Troubleshooting: jobs aparecem como `Skipped`

O gatilho do workflow depende do prefixo `[secrets-generator]` no título da issue (`if: startsWith(...)`),
não da label. Isso porque o campo `labels` do issue form só é aplicado se a label `secrets-generator`
**já existir** no repositório; caso contrário, o GitHub cria a issue sem a label e qualquer `if` baseado
nela nunca é satisfeito, fazendo todos os jobs aparecerem como `Skipped`.

Se os jobs continuarem pulados, confirme:

- O título da issue começa com `[secrets-generator]` (valor padrão do template, não deve ser removido).
- O workflow está na branch padrão do repositório (`main`) — GitHub só considera o `secrets-generator.yml`
  da branch default para eventos `issues`.
- Em `Settings > Actions > General`, a opção `Allow all actions and reusable workflows` (ou equivalente)
  está habilitada, permitindo o uso de actions de terceiros como `stefanbuck/github-issue-parser`.
- Opcionalmente, crie a label `secrets-generator` em `Issues > Labels` para fins de organização/filtro
  (não é mais exigida para o disparo do workflow).
