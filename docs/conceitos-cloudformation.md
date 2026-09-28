# 📖 Conceitos Fundamentais do AWS CloudFormation

## O que é Infraestrutura como Código (IaC)?

**Infraestrutura como Código** é a prática de definir e gerenciar infraestrutura (servidores, redes, bancos de dados) usando **arquivos de texto** em vez de processos manuais.

**Analogia simples:** imagine que, em vez de montar um móvel peça por peça seguindo instruções na sua cabeça, você tivesse um robô que lê um arquivo com as instruções e monta tudo automaticamente, sempre igual. Isso é IaC.

**Por que é importante?**
- Você pode **versionar** sua infraestrutura no Git, como faz com código
- Elimina erros humanos de configuração manual
- Permite **replicar ambientes** facilmente (dev, staging, produção)
- Qualquer pessoa do time pode entender a infraestrutura lendo o código

---

## O que é o AWS CloudFormation?

O **CloudFormation** é o serviço de IaC nativo da AWS. Você escreve um **template** (arquivo YAML ou JSON) descrevendo os recursos que quer criar, e o CloudFormation cuida de:

1. **Criar** todos os recursos na ordem correta
2. **Resolver dependências** entre recursos automaticamente
3. **Atualizar** recursos quando o template muda
4. **Deletar** tudo de forma limpa quando você não precisa mais

O conjunto de recursos criados a partir de um template é chamado de **Stack** (pilha).

---

## Anatomia de um Template CloudFormation

Um template CloudFormation pode ter as seguintes seções:

```yaml
AWSTemplateFormatVersion: '2010-09-09'   # Versão do formato (opcional)
Description: Texto descritivo             # Descrição da stack (opcional)

Parameters:     # Valores configuráveis pelo usuário (opcional)
  ...

Mappings:       # Tabelas de lookup estáticas (opcional)
  ...

Resources:      # Recursos AWS a criar (⚠️ OBRIGATÓRIO)
  ...

Outputs:        # Informações exibidas após a criação (opcional)
  ...
```

### AWSTemplateFormatVersion

```yaml
AWSTemplateFormatVersion: '2010-09-09'
```
Declara a versão do formato do template. A única versão válida é `'2010-09-09'`. É opcional, mas boa prática incluir.

### Description

```yaml
Description: Minha stack de exemplo
```
Texto que aparece no console da AWS ao lado da stack. Ajuda a identificar o propósito rapidamente.

### Parameters

Valores que o usuário pode definir na hora de criar a stack, tornando o template **reutilizável**.

```yaml
Parameters:
  TipoInstancia:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro
    Description: Tipo da instância EC2
```

- **Type:** tipo do parâmetro (String, Number, etc.)
- **Default:** valor padrão se o usuário não informar
- **AllowedValues:** restringe as opções válidas

### Resources (obrigatório)

A **única seção obrigatória**. Lista todos os recursos AWS que serão criados.

```yaml
Resources:
  NomeLogicoDoRecurso:        # Você escolhe o nome
    Type: AWS::Servico::Recurso  # Tipo do recurso AWS
    Properties:                # Configurações do recurso
      Propriedade1: valor1
      Propriedade2: valor2
```

### Mappings

Tabelas de lookup para valores que variam conforme região, ambiente, etc.

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0123456789
    us-west-2:
      AMI: ami-9876543210
```

Use a função `!FindInMap` para buscar valores:
```yaml
ImageId: !FindInMap [RegionMap, !Ref "AWS::Region", AMI]
```

### Outputs

Informações exibidas após a criação da stack. Úteis para consultar IDs e nomes dos recursos criados.

```yaml
Outputs:
  IdDaInstancia:
    Description: ID da instância EC2
    Value: !Ref MinhaInstancia
```

---

## Funções Intrínsecas Mais Comuns

O CloudFormation oferece funções especiais para tornar templates dinâmicos:

| Função | O que faz | Exemplo |
|---|---|---|
| `!Ref` | Referencia um recurso ou parâmetro | `!Ref MinhaInstancia` |
| `!Sub` | Substitui variáveis dentro de uma string | `!Sub "Olá ${NomeDoParametro}"` |
| `Fn::Base64` | Converte texto para Base64 (necessário para UserData) | `Fn::Base64: !Sub \|` |
| `!FindInMap` | Busca valor em um Mapping | `!FindInMap [Map, Chave1, Chave2]` |
| `!GetAtt` | Obtém um atributo específico de um recurso | `!GetAtt MinhaInstancia.PublicIp` |

### !Ref em detalhe

`!Ref` é a função mais usada. O que ela retorna depende do tipo de recurso:

- **Em um Parameter:** retorna o valor informado pelo usuário
- **Em `AWS::EC2::Instance`:** retorna o Instance ID
- **Em `AWS::S3::Bucket`:** retorna o nome do bucket
- **Em `AWS::EC2::SecurityGroup`:** retorna o ID do security group

---

## O que é uma Stack?

Uma **Stack** é o conjunto de recursos AWS criados a partir de um template. Quando você faz `create-stack`, o CloudFormation:

1. Lê o template
2. Resolve dependências entre recursos
3. Cria tudo na ordem correta
4. Se algo falhar, faz **rollback** automático (desfaz tudo)

**Ciclo de vida de uma stack:**
```
Criar (CREATE) → Atualizar (UPDATE) → Deletar (DELETE)
```

Ao **deletar** uma stack, o CloudFormation remove **todos** os recursos que criou — por isso é seguro usar para testes.

---

## Drift Detection (Detecção de Desvio)

E se alguém modificar um recurso **manualmente** no console da AWS depois que a stack já foi criada? O recurso ficaria diferente do que o template define. Isso é chamado de **drift** (desvio).

O CloudFormation oferece o recurso de **Drift Detection**, que compara o estado atual dos recursos com o que está definido no template e mostra as diferenças.

```bash
aws cloudformation detect-stack-drift --stack-name minha-stack
```

**Dica:** evite alterar recursos manualmente. Sempre faça alterações pelo template para manter a consistência.
