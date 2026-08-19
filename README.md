# togglemaster-secrets-generator

Workflow do GitHub Actions para criar/atualizar secrets no AWS Secrets Manager do projeto ToggleMaster,
disparado através da abertura de uma issue.

### Como usar

Vá em `Issues` > `New issue` > escolha o template `🔑 Secrets Generator` e preencha os 3 campos:

`Environment` - O environment onde o secret será criado: `dev`, `qa` ou `prod`

`Secret Name` - Deve seguir o padrão `<prefixo>/<componente>/<secret-name>`, por exemplo: `togglemaster-dev/app/service-api-key`

`Secret Value` - O valor do secret, ou seja, a senha, API key, etc.

Clique em `Submit new issue`. A label `secrets-generator` é adicionada automaticamente pelo template e
dispara o workflow, que valida os campos, cria/atualiza o secret, ofusca o valor no corpo da issue e a
encerra com um comentário de confirmação.

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
