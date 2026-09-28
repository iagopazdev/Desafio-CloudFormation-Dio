# ☁️ Minha Primeira Stack com AWS CloudFormation

![AWS](https://img.shields.io/badge/AWS-CloudFormation-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![DIO](https://img.shields.io/badge/DIO-Desafio%20de%20Projeto-8B00FF?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Concluído-00C853?style=for-the-badge)

> **Desafio de Projeto** da [DIO](https://dio.me) — Documentação da minha primeira experiência criando stacks de infraestrutura com AWS CloudFormation.

---

## 📋 Sobre o Projeto

Este repositório documenta minha jornada ao criar **stacks de infraestrutura na AWS usando CloudFormation**. O objetivo é registrar anotações, insights e aprendizados adquiridos durante a prática, servindo como material de apoio para estudos e futuras implementações.

### O que é AWS CloudFormation?

O **AWS CloudFormation** é o serviço de **Infraestrutura como Código (IaC)** da Amazon. Ele permite que você defina toda a sua infraestrutura em arquivos de texto (templates YAML ou JSON) e provisione tudo de forma automatizada e repetível.

Em vez de criar recursos manualmente no console da AWS (clicando em botões), você **descreve** o que quer em um template e o CloudFormation **cria, configura e gerencia** tudo pra você.

**Principais vantagens:**
- ✅ **Repetibilidade** — o mesmo template cria a mesma infraestrutura sempre
- ✅ **Versionamento** — templates ficam no Git, com histórico de mudanças
- ✅ **Automação** — sem cliques manuais, menos erro humano
- ✅ **Rastreabilidade** — tudo que foi criado fica registrado na stack

---

## 🗂️ Estrutura do Repositório

```
dio-aws-cloudformation-stack/
│
├── 📄 README.md                           # Este arquivo — documentação principal
│
├── 📁 templates/                          # Templates CloudFormation (YAML)
│   ├── EC2.yaml                           # Stack 1: Instância EC2 simples
│   ├── Apache.yaml                        # Stack 2: EC2 + Servidor Apache
│   ├── Firewall.yaml                      # Stack 3: EC2 + Apache + Security Group
│   └── EC2_S3_UserGroup.yaml              # Stack 4: EC2 + S3 + IAM (completa)
│
├── 📁 docs/                               # Documentação complementar
│   ├── conceitos-cloudformation.md        # Conceitos fundamentais
│   ├── passo-a-passo.md                   # Guia prático com comandos AWS CLI
│   └── insights-e-dicas.md               # Lições aprendidas e dicas
│
└── 📁 imagens/                            # Capturas de tela (opcional)
```

---

## 🧱 Templates CloudFormation

Os templates seguem uma **progressão de complexidade**, do mais simples ao mais completo:

### 1️⃣ EC2.yaml — Instância EC2 Simples

> **Conceito:** Criação básica de um recurso na AWS via CloudFormation.

| Propriedade | Valor | Descrição |
|---|---|---|
| Tipo | `AWS::EC2::Instance` | Instância de máquina virtual |
| Região | `us-east-1a` | Zona de disponibilidade |
| AMI | `ami-0ed9277fb7eb570c9` | Amazon Linux 2 |
| Tipo de Instância | `t2.micro` | Elegível ao Free Tier |

**O que aprendi:** Este é o template mais básico possível. Ele mostra a estrutura mínima de um template CloudFormation — `Description` e `Resources`.

---

### 2️⃣ Apache.yaml — EC2 com Servidor Web Apache

> **Conceito novo:** `UserData` — executar comandos automaticamente quando a instância inicia.

Além de criar a instância EC2, este template **instala e configura o Apache HTTP Server** automaticamente usando um script bash no campo `UserData`. É o primeiro passo para automação de configuração.

**Comandos executados na inicialização:**
```bash
yum install -y httpd.x86_64        # Instala o Apache
systemctl start httpd.service       # Inicia o serviço
systemctl enable httpd.service      # Habilita na inicialização
echo "<h1>...</h1>" > /var/www/html/index.html  # Cria página inicial
```

**O que aprendi:** O `UserData` com `Fn::Base64` permite automatizar a configuração do servidor logo na criação. Isso elimina a necessidade de acessar via SSH para configurar manualmente.

---

### 3️⃣ Firewall.yaml — EC2 + Apache + Security Group

> **Conceito novo:** `AWS::EC2::SecurityGroup` — o firewall da AWS que controla o tráfego de rede.

Este template adiciona um **Security Group** (grupo de segurança) à instância, liberando a **porta 80 (HTTP)** para acesso externo. Sem isso, o Apache estaria rodando mas ninguém conseguiria acessar a página.

**Regra de entrada configurada:**
| Protocolo | Porta | Origem | Significado |
|---|---|---|---|
| TCP | 80 | `0.0.0.0/0` | HTTP liberado para qualquer IP |

**Conceito importante — `!Ref`:** A função `!Ref` cria uma referência entre recursos. No template, a instância EC2 referencia o Security Group usando `!Ref GrupoSeguranca`, criando a associação automaticamente.

**O que aprendi:** Security Groups são essenciais. Sem eles, os recursos ficam isolados. O `!Ref` é uma das funções mais usadas no CloudFormation para conectar recursos entre si.

---

### 4️⃣ EC2_S3_UserGroup.yaml — Template Completo

> **Conceitos novos:** `Parameters`, `Mappings`, `Outputs` e múltiplos serviços AWS integrados.

Este é o template mais completo, demonstrando a criação coordenada de **múltiplos recursos AWS**:

**Recursos criados:**
| Recurso | Tipo AWS | Descrição |
|---|---|---|
| Instância EC2 | `AWS::EC2::Instance` | Servidor virtual com Ubuntu |
| Bucket S3 | `AWS::S3::Bucket` | Armazenamento de objetos |
| Grupo IAM | `AWS::IAM::Group` | Grupo de permissões |
| Usuário IAM | `AWS::IAM::User` | Usuário vinculado ao grupo |
| Security Group | `AWS::EC2::SecurityGroup` | Firewall com SSH (porta 22) |

**Seções avançadas utilizadas:**

- **`Parameters`** — Permite que valores sejam definidos na hora de criar a stack (ex: tipo de instância). Torna o template reutilizável.
- **`Mappings`** — Tabela de lookup que seleciona a AMI correta com base na região. Usa a função `!FindInMap`.
- **`Outputs`** — Exibe informações úteis após a criação da stack (IDs dos recursos criados).

**O que aprendi:** Templates reais usam Parameters para flexibilidade, Mappings para lidar com variações regionais e Outputs para facilitar a consulta dos recursos criados. Esse template mostra como integrar vários serviços AWS em uma única stack.

---

## 🚀 Como Usar

### Pré-requisitos

- Conta AWS ativa (o [Free Tier](https://aws.amazon.com/free/) é suficiente para estes templates)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) instalado e configurado (`aws configure`)

### Criar uma Stack

```bash
aws cloudformation create-stack \
  --stack-name minha-primeira-stack \
  --template-body file://templates/EC2.yaml
```

### Verificar o Status

```bash
aws cloudformation describe-stacks --stack-name minha-primeira-stack
```

### Deletar a Stack

```bash
aws cloudformation delete-stack --stack-name minha-primeira-stack
```

> ⚠️ **Importante:** Sempre delete suas stacks de teste para evitar cobranças inesperadas na AWS.

Para mais detalhes e comandos, veja o [Guia Prático](docs/passo-a-passo.md).

---

## 📚 Documentação Complementar

| Documento | Descrição |
|---|---|
| [Conceitos do CloudFormation](docs/conceitos-cloudformation.md) | Fundamentos teóricos: IaC, templates, seções, funções intrínsecas |
| [Passo a Passo Prático](docs/passo-a-passo.md) | Comandos AWS CLI para o ciclo de vida completo de uma stack |
| [Insights e Dicas](docs/insights-e-dicas.md) | Lições aprendidas, erros comuns e boas práticas |

---

## 💡 Principais Aprendizados

1. **Infraestrutura como Código muda o jogo** — Definir infraestrutura em código traz consistência, versionamento e automação.
2. **Comece simples, evolua gradualmente** — A progressão EC2 → UserData → Security Group → Multi-serviço é um caminho natural de aprendizado.
3. **O CloudFormation gerencia dependências** — Usando `!Ref`, o CloudFormation resolve a ordem de criação automaticamente.
4. **Parameters tornam templates reutilizáveis** — Em vez de alterar o YAML, você passa valores diferentes na criação.
5. **Sempre limpe seus recursos** — Deletar stacks de teste evita surpresas na fatura da AWS.

---

## 🔗 Referências

- [AWS CloudFormation — Documentação Oficial](https://docs.aws.amazon.com/cloudformation/)
- [AWS CloudFormation — Template Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [AWS Free Tier](https://aws.amazon.com/free/)
- [DIO — Plataforma de Ensino](https://dio.me)

---

## 🏅 Sobre

Projeto desenvolvido como parte de um **Desafio de Projeto** na [DIO](https://dio.me), com o objetivo de praticar a criação de stacks com AWS CloudFormation e documentar o aprendizado adquirido.

---

<div align="center">
  Feito com 💜 durante o aprendizado na <a href="https://dio.me">DIO</a>
</div>
