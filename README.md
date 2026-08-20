# togglemaster-secrets-generator

Workflow do GitHub Actions para criar/atualizar secrets no AWS Secrets Manager do projeto ToggleMaster,
disparado através da abertura de uma issue.

### Como usar

Vá em `Issues` > `New issue` > escolha o template `🔑 Secrets Generator` e preencha os 3 campos:

`Environment` - O environment onde o secret será criado: `dev`, `qa` ou `prod`

`Secret Name` - Deve seguir o padrão `<prefixo>/<componente>/<secret-name>`, por exemplo: `togglemaster-dev/app/service-api-key`

`Secret Value` - O valor do secret, ou seja, a senha, API key, etc.

Clique em `Submit new issue`. O workflow é disparado pelo prefixo `[secrets-generator]` no título
(preenchido automaticamente pelo template) e sempre encerra a issue ao final, com um comentário de
sucesso ou de falha, e ofusca o valor do secret no corpo da issue.

### Segurança

- O valor do secret fica visível no corpo da issue até o workflow concluir a execução (poucos segundos),
  quando então é substituído por `***REDACTED***`. Restrinja quem pode abrir issues neste repositório
  (repositório privado/interno, `Settings > General > Features > Issues`) para evitar exposição do valor.
- O log do workflow mascara o valor do secret (`::add-mask::`), mas evite reabrir a issue ou reutilizá-la
  para novos valores.

### Pré-requisitos

Cada environment (`dev`, `qa`, `prod`) do repositório precisa ter configurado:

- `vars.AWS_ROLE_TO_ASSUME` - ARN da role IAM assumida via OIDC (precisa de permissão `secretsmanager:CreateSecret`, `secretsmanager:UpdateSecret`, `secretsmanager:DescribeSecret`).
- `vars.AWS_REGION` - região da conta AWS do environment.

A role precisa confiar no provider OIDC do GitHub Actions (`token.actions.githubusercontent.com`) para o repositório `togglemaster-secrets-generator`.

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
