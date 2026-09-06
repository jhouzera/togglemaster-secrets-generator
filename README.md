# togglemaster-secrets-generator

Ferramenta via automação de GitHub Actions para abstrair a criação e atualização de *Secrets* da AWS.

## 🎯 Propósito
Evitar acesso humano ao console produtivo da AWS ou ao CLI.
Facilitar o gerenciamento de credenciais e API Keys da plataforma utilizando fluxos auditáveis (IssueOps).

## 🚀 Como Utilizar

Toda a interação é feita pela própria interface do GitHub via **Issues**.

1. Vá na aba `Issues` > `New issue`.
2. Selecione o template **🔑 Secrets Generator**.
3. Preencha o ambiente (`dev`, `prd`), a chave (Key) e o valor criptografado ou texto puro (Value).
4. Submeta a Issue.

Assim que a Issue for aberta, um GitHub Action disparará silenciosamente se autenticando na nuvem, injetando o novo Secret no **AWS Secrets Manager** e fechando a Issue com um comentário de Sucesso (suprimindo a saída visual para não expor a senha nos logs da Action).

## 🔐 Segurança e Boas Práticas
- **Sem logs:** Os valores (payloads) do *Issue Form* são ocultos no processo de execução e não transbordam para os logs do terminal da pipeline.
- Permissões estritas vinculadas via GitHub OIDC garantem que a máquina possua somente permissões pontuais de escrita em serviços de cofre (KMS/SecretsManager).
