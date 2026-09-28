# 🛠️ Guia Prático — Criando e Gerenciando Stacks com AWS CLI

Este guia mostra os comandos práticos para o ciclo de vida completo de uma stack CloudFormation usando a **AWS CLI** (Command Line Interface).

---

## Pré-requisitos

1. **Conta AWS ativa** — o [Free Tier](https://aws.amazon.com/free/) é suficiente
2. **AWS CLI instalada** — [guia de instalação](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
3. **Credenciais configuradas:**

```bash
aws configure
# Será pedido:
#   AWS Access Key ID: sua chave de acesso
#   AWS Secret Access Key: sua chave secreta
#   Default region name: us-east-1 (ou sua região preferida)
#   Default output format: json
```

> 💡 **Dica:** Nunca compartilhe suas credenciais AWS. Não suba elas para o GitHub.

---

## 1. Validar o Template

Antes de criar a stack, verifique se o template está correto:

```bash
aws cloudformation validate-template \
  --template-body file://templates/EC2.yaml
```

Se o template for válido, a AWS retorna informações sobre ele (parâmetros, descrição). Se tiver erros de sintaxe, a mensagem de erro aponta o problema.

---

## 2. Criar uma Stack

```bash
aws cloudformation create-stack \
  --stack-name minha-ec2-simples \
  --template-body file://templates/EC2.yaml
```

**Para templates com recursos IAM** (como o EC2_S3_UserGroup.yaml), adicione `--capabilities`:

```bash
aws cloudformation create-stack \
  --stack-name minha-stack-completa \
  --template-body file://templates/EC2_S3_UserGroup.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

> ⚠️ A flag `--capabilities CAPABILITY_NAMED_IAM` é obrigatória quando o template cria recursos IAM (usuários, grupos, roles). É uma confirmação de segurança.

**Para templates com Parameters:**

```bash
aws cloudformation create-stack \
  --stack-name minha-stack \
  --template-body file://templates/EC2_S3_UserGroup.yaml \
  --parameters ParameterKey=InstanceType,ParameterValue=t2.micro \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## 3. Acompanhar o Status da Stack

```bash
aws cloudformation describe-stacks \
  --stack-name minha-ec2-simples
```

**Status possíveis:**
| Status | Significado |
|---|---|
| `CREATE_IN_PROGRESS` | Criação em andamento |
| `CREATE_COMPLETE` | ✅ Stack criada com sucesso |
| `CREATE_FAILED` | ❌ Falha na criação |
| `ROLLBACK_IN_PROGRESS` | Desfazendo após uma falha |
| `ROLLBACK_COMPLETE` | Rollback concluído |
| `DELETE_IN_PROGRESS` | Exclusão em andamento |
| `DELETE_COMPLETE` | Stack deletada |

---

## 4. Listar Eventos da Stack

Para ver o histórico detalhado de tudo que aconteceu:

```bash
aws cloudformation describe-stack-events \
  --stack-name minha-ec2-simples
```

Isso é muito útil para **debugar problemas** — cada recurso mostra quando começou, se teve sucesso ou falhou, e qual foi o erro.

---

## 5. Listar Recursos Criados

```bash
aws cloudformation list-stack-resources \
  --stack-name minha-ec2-simples
```

Mostra todos os recursos que a stack criou, com seus IDs e status.

---

## 6. Atualizar uma Stack

Se você alterou o template e quer aplicar as mudanças:

```bash
aws cloudformation update-stack \
  --stack-name minha-ec2-simples \
  --template-body file://templates/EC2.yaml
```

> 💡 O CloudFormation calcula as diferenças (changeset) e aplica apenas o que mudou. Alguns recursos podem ser atualizados sem interrupção, outros precisam ser recriados.

---

## 7. Deletar uma Stack

```bash
aws cloudformation delete-stack \
  --stack-name minha-ec2-simples
```

> ⚠️ **IMPORTANTE:** Deletar a stack remove **todos** os recursos que ela criou. Sempre delete stacks de teste para evitar cobranças na AWS.

Para confirmar que foi deletada:

```bash
aws cloudformation describe-stacks \
  --stack-name minha-ec2-simples
# Deve retornar erro "Stack does not exist" ou status DELETE_COMPLETE
```

---

## 8. Listar Todas as Stacks

```bash
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE
```

---

## Alternativa: Console da AWS

Todos os passos acima também podem ser feitos pelo **Console Web da AWS**:

1. Acesse o [Console do CloudFormation](https://console.aws.amazon.com/cloudformation/)
2. Clique em **"Create stack"**
3. Faça upload do arquivo YAML
4. Defina os parâmetros (se houver)
5. Revise e confirme

O console é mais visual e amigável para quem está começando. A CLI é mais rápida e permite automação.

---

## Resumo dos Comandos

| Ação | Comando |
|---|---|
| Validar template | `aws cloudformation validate-template --template-body file://template.yaml` |
| Criar stack | `aws cloudformation create-stack --stack-name NOME --template-body file://template.yaml` |
| Ver status | `aws cloudformation describe-stacks --stack-name NOME` |
| Ver eventos | `aws cloudformation describe-stack-events --stack-name NOME` |
| Listar recursos | `aws cloudformation list-stack-resources --stack-name NOME` |
| Atualizar stack | `aws cloudformation update-stack --stack-name NOME --template-body file://template.yaml` |
| Deletar stack | `aws cloudformation delete-stack --stack-name NOME` |
| Listar stacks | `aws cloudformation list-stacks` |
