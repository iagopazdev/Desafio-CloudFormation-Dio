# 💡 Insights e Dicas — Lições Aprendidas

> Este é um documento vivo. Atualize-o com suas próprias experiências ao longo da prática!

---

## 🐛 Erros Comuns e Como Resolver

### 1. Erro de validação do template

**Problema:** `Template format error` ou `Invalid template property`

**Causa provável:** erro de indentação no YAML, propriedade mal escrita ou seção no lugar errado.

**Como resolver:**
- Valide o template antes de usar: `aws cloudformation validate-template --template-body file://template.yaml`
- Confira a indentação — YAML é sensível a espaços (nunca use TAB)
- Consulte a [referência oficial do recurso](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

### 2. Conflito de nomes de recursos

**Problema:** `S3-FOUNDATION already exists` ou `GPO-ADMIN-LAB already exists`

**Causa:** o recurso com aquele nome já existe na AWS (de uma stack anterior ou criação manual).

**Como resolver:**
- Delete a stack anterior que criou o recurso, ou
- Mude o nome no template, ou
- Evite usar nomes fixos — deixe o CloudFormation gerar nomes automaticamente

### 3. Stack em ROLLBACK_COMPLETE

**Problema:** a stack falhou na criação e ficou presa em `ROLLBACK_COMPLETE`.

**Como resolver:**
1. Veja os eventos para entender a causa: `aws cloudformation describe-stack-events --stack-name NOME`
2. Delete a stack com falha: `aws cloudformation delete-stack --stack-name NOME`
3. Corrija o template
4. Crie novamente

### 4. Falta de permissões (CAPABILITY_NAMED_IAM)

**Problema:** `Requires capabilities: [CAPABILITY_NAMED_IAM]`

**Causa:** o template cria recursos IAM (usuários, grupos, roles) e a AWS exige confirmação explícita.

**Como resolver:**
```bash
aws cloudformation create-stack \
  --stack-name NOME \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## ✅ Boas Práticas

1. **Use Parameters em vez de valores hardcoded** — torna o template reutilizável em diferentes ambientes
2. **Sempre use Tags** — facilita a organização e o rastreamento de custos
3. **Delete stacks de teste** — recursos ativos geram custos, mesmo que pequenos
4. **Versione seus templates no Git** — mantenha um histórico de mudanças
5. **Valide antes de criar** — o comando `validate-template` economiza tempo
6. **Comece simples, evolua gradualmente** — a progressão EC2 → UserData → Security Group → Multi-recurso é o caminho natural
7. **Nunca altere recursos manualmente** — faça mudanças pelo template para evitar drift

---

## 📈 Ordem de Aprendizado Sugerida

```
1️⃣ EC2.yaml            → Conceito básico de recurso e template
2️⃣ Apache.yaml         → UserData e automação de configuração
3️⃣ Firewall.yaml       → Security Groups e !Ref
4️⃣ EC2_S3_UserGroup.yaml → Parameters, Mappings, Outputs e multi-serviço
```

Cada template adiciona uma camada de complexidade. Domine um antes de ir para o próximo.

---

## 📝 Minhas Anotações Pessoais

<!-- 
  Adicione aqui suas próprias observações durante a prática.
  Exemplos:
  - O que achou mais interessante?
  - O que foi diferente do que esperava?
  - Algum "momento eureka"?
-->

_Escreva suas anotações aqui..._

---

## 🚧 Dificuldades Encontradas

<!-- 
  Registre os problemas que você enfrentou e como resolveu.
  Isso é muito valioso para revisão futura!
  Exemplos:
  - "O template dava erro de indentação porque usei TAB em vez de espaços"
  - "A stack ficou presa em ROLLBACK porque o nome do bucket já existia"
-->

_Registre suas dificuldades aqui..._

---

## 🎯 Próximos Passos

<!-- 
  O que você quer aprender depois?
  Sugestões:
  - [ ] Usar CloudFormation com VPC e sub-redes
  - [ ] Explorar Nested Stacks (stacks dentro de stacks)
  - [ ] Aprender AWS CDK (CloudFormation com linguagens de programação)
  - [ ] Experimentar o Terraform como alternativa ao CloudFormation
-->

- [ ] _Adicione seus próximos passos aqui..._
