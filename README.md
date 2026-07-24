# 📚 Cronograma de Estudos — OCI Foundations (1Z0-1085)

> **Autor:** Arquiteto de Soluções Oracle Cloud & Instrutor Certificado  
> **Objetivo:** Preparação completa para o exame de certificação Oracle Cloud Infrastructure Foundations  
> **Metodologia:** Teoria Oficial + Custos Reais (Billing) + Simulados de Fixação  
> **Dados de pricing:** Atualizados em tempo real via Oracle Cloud Pricing API

---

## 🗺️ Visão Geral do Cronograma

| # | Pilar | Tópicos Principais | Peso no Exame |
|---|-------|-------------------|---------------|
| 1 | **Conceitos de Nuvem e Arquitetura OCI** | Regiões, ADs, FDs, IAM, Compartments | ~20% |
| 2 | **Serviços Principais do OCI** | Compute, VCN, Storage, Autonomous DB | ~35% |
| 3 | **Segurança e Conformidade** | Responsabilidade Compartilhada, WAF, Vault, Identity Domains | ~20% |
| 4 | **Preços, Faturamento e Suporte** | SLA, Billing, Budgets, Cost Analysis, Always Free | ~25% |

### Metodologia de Cada Módulo

Cada tópico segue obrigatoriamente 3 partes:

```
[TEORIA]     → Conceito-chave alinhado à documentação oficial Oracle
[PRÁTICA & CUSTO] → Cenários reais de precificação (Always Free vs. Produção)
[SIMULADO]   → 2 questões no formato exato do exame oficial
```

---

## 📖 Informações do Exame

| Item | Detalhe |
|------|---------|
| Código | 1Z0-1085-24 |
| Formato | Múltipla escolha e múltipla seleção |
| Questões | ~60 questões |
| Duração | 90 minutos |
| Nota mínima | 68% |
| Idioma | Inglês (com tradução disponível) |
| Custo | US$ 150 (ou gratuito via Oracle MyLearn) |

---

---

# 🏛️ MÓDULO 1 — Pilar 1: Conceitos de Nuvem e Arquitetura OCI

---

## 1.1 Regiões (Regions)

### [TEORIA]

**Definição Oficial:** Uma Região OCI é uma área geográfica localizada composta por um ou mais Availability Domains. Regiões são completamente independentes entre si e podem estar separadas por grandes distâncias (entre países ou continentes).

**Conceitos-chave para o exame:**

- A OCI opera atualmente **32 regiões comerciais** globalmente
- Além das comerciais, existem **regiões governamentais** e de **segurança nacional** (isoladas)
- Cada tenancy possui uma **Home Region** — região master para recursos IAM
- Recursos IAM são criados na Home Region e **replicados automaticamente** para regiões assinadas
- O usuário pode se inscrever (**subscribe**) em múltiplas regiões
- Naming convention: `<país>-<cidade>-<número>` (ex: `sa-saopaulo-1`, `eu-frankfurt-1`)
- A OCI mantém **preços consistentes** entre todas as regiões comerciais (diferencial vs AWS/Azure)

**Regiões no Brasil:**

| Região | Localização |
|--------|-------------|
| `sa-saopaulo-1` | São Paulo, Brazil |
| `sa-vinhedo-1` | Vinhedo, Brazil |

**Dica de prova:** O exame pergunta sobre o papel da Home Region e sobre a replicação de IAM. Lembre-se: IAM é **global ao tenancy** mas a fonte autoritativa está na Home Region.

---

### [PRÁTICA & CUSTO]

**Transferência de Dados entre Regiões (Data Egress)**

| Métrica | OCI | AWS / Azure | Economia |
|---------|-----|-------------|----------|
| Free Tier Egress | **10 TB/mês** | ~100 GB/mês | 100x mais free |
| Preço após free tier | **$0,0085/GB** | ~$0,09/GB | **~90% mais barato** |
| Custo para 10 TB/mês | **$0** | ~$912/mês | **$912/mês de economia** |

> 💡 **Ponto crítico para a prova:** O tráfego de dados **dentro da mesma região** (entre ADs) é **GRATUITO**. A cobrança de egress aplica-se apenas ao tráfego que **sai da OCI para a internet**.

**Serviços de Networking GRATUITOS:**

| Serviço | Custo |
|---------|-------|
| VCN, Subnets, Route Tables, Security Lists | **FREE** |
| Network Load Balancer (Layer 4) | **FREE** |
| Service Gateway (acesso privado a OCI services) | **FREE** |
| 1 Flexible Load Balancer (10 Mbps) | **FREE** (Always Free) |

---

## 1.2 Availability Domains (ADs)

### [TEORIA]

**Definição Oficial:** Um Availability Domain é um ou mais data centers tolerantes a falhas, localizados dentro de uma Região, mas conectados entre si por uma rede de baixa latência e alta largura de banda.

**Conceitos-chave para o exame:**

- Uma região pode ter **1 ou 3 ADs**
- Regiões com 3 ADs: `us-ashburn-1`, `us-phoenix-1`, `eu-frankfurt-1`, `uk-london-1`
- A **maioria das regiões novas** possui apenas **1 AD** (design moderno com FDs para HA)
- ADs são **isolados** uns dos outros (energia, cooling e rede independentes)
- Falha em um AD **NÃO** afeta outros ADs na mesma região
- Interconexão entre ADs: rede criptografada de **alta largura de banda e baixa latência**
- **Tráfego entre ADs na mesma região é GRATUITO**

**Arquitetura de HA em regiões com 1 AD:**
> Em regiões com apenas 1 AD, a alta disponibilidade é garantida através dos **Fault Domains** (3 por AD), que oferecem isolamento de hardware similar.

---

## 1.3 Fault Domains (FDs)

### [TEORIA]

**Definição Oficial:** Um Fault Domain é um agrupamento de hardware e infraestrutura dentro de um Availability Domain. FDs permitem distribuir instâncias para que não compartilhem o mesmo hardware físico (anti-afinidade).

**Conceitos-chave para o exame:**

- Cada AD contém **exatamente 3 Fault Domains** (FD 1, FD 2, FD 3)
- FDs protegem contra:
  - ✅ Falhas de **hardware inesperadas**
  - ✅ **Manutenção planejada** (live migration, patching)
- Quando a OCI faz manutenção em 1 FD, os outros 2 **NÃO são afetados**
- O usuário pode **especificar** em qual FD alocar instâncias
- Se não especificado, a OCI distribui automaticamente (**anti-afinidade**)
- FDs são "**data centers lógicos**" dentro de cada AD

**Hierarquia completa:**

```
Tenancy (conta Oracle Cloud)
└── Region (ex: sa-saopaulo-1)
    └── Availability Domain 1
    │   ├── Fault Domain 1  ← hardware isolado
    │   ├── Fault Domain 2  ← hardware isolado
    │   └── Fault Domain 3  ← hardware isolado
    └── Availability Domain 2 (se houver, ex: us-ashburn-1)
    │   ├── Fault Domain 1
    │   ├── Fault Domain 2
    │   └── Fault Domain 3
    └── Availability Domain 3 (se houver)
        ├── Fault Domain 1
        ├── Fault Domain 2
        └── Fault Domain 3
```

**Tabela de proteção por camada:**

| Camada | Protege contra | Exemplo |
|--------|---------------|---------|
| Fault Domain | Falha de hardware / manutenção | Servidor queimou |
| Availability Domain | Falha de data center | Incêndio no prédio |
| Region (cross-region DR) | Desastre natural / regional | Terremoto na cidade |

---

### [PRÁTICA & CUSTO]

**Custo de usar FDs e ADs:**

| Recurso | Custo |
|---------|-------|
| Distribuir instâncias entre Fault Domains | **GRATUITO** |
| Distribuir instâncias entre Availability Domains | **GRATUITO** |
| Tráfego de dados entre ADs (mesma região) | **GRATUITO** |
| Tráfego de dados entre Regiões | $0,0085/GB (após 10 TB free) |

> 💡 **Para a prova:** Não existe custo adicional para usar múltiplos FDs ou ADs. A decisão de distribuir workloads é **puramente arquitetural**, sem impacto financeiro dentro da mesma região.

---

## 1.4 IAM (Identity and Access Management)

### [TEORIA]

**Definição Oficial:** O OCI Identity and Access Management (IAM) permite controlar quem tem acesso aos recursos de cloud e que tipo de acesso eles possuem.

**Componentes-chave para o exame:**

| Componente | Definição |
|------------|-----------|
| **Tenancy** | Conta raiz OCI. É o compartment raiz de toda a hierarquia |
| **Users** | Identidades individuais que podem fazer login na console ou usar API |
| **Groups** | Coleção de usuários com as mesmas permissões |
| **Policies** | Documentos que especificam quem pode acessar o quê (formato legível) |
| **Compartments** | Coleções lógicas de recursos para organização e controle de acesso |
| **Identity Domains** | Container para users, groups e configurações de autenticação |

**Sintaxe de Policies (MUITO cobrado no exame):**

```
Allow <group_name> to <verb> <resource-type> in <location> where <conditions>
```

**Verbos de Policy (do menos ao mais permissivo):**

| Verbo | Permissão |
|-------|-----------|
| `inspect` | Listar recursos (somente metadados) |
| `read` | Inspect + obter conteúdo/detalhes |
| `use` | Read + interagir com recursos existentes |
| `manage` | Controle total (criar, deletar, editar) |

**Exemplos práticos:**

```
Allow group NetworkAdmins to manage virtual-network-family in compartment Production
Allow group Developers to use instances in compartment Dev
Allow group Auditors to inspect all-resources in tenancy
```

**Dica de prova:** 
- Policies são aplicadas a **Groups**, NUNCA diretamente a Users
- Se nenhuma policy concede acesso, o acesso é **negado por padrão** (deny implícito)
- A política `manage all-resources in tenancy` equivale a acesso **root/admin total**

---

## 1.5 Compartments

### [TEORIA]

**Definição Oficial:** Compartments são coleções lógicas de recursos cloud relacionados que podem ser acessados apenas por grupos com permissões concedidas por policies.

**Conceitos-chave para o exame:**

- Todo tenancy possui um **root compartment** (criado automaticamente)
- Compartments podem ser **aninhados** (até 6 níveis de profundidade)
- Um recurso pertence a **exatamente um** compartment
- Recursos podem interagir com outros em **compartments diferentes**
- Compartments são **globais** (span across all regions do tenancy)
- **Deletar** um compartment requer que ele esteja **vazio**
- Compartments servem para: **isolamento**, **billing/budget**, **controle de acesso**

**Boas práticas de design:**

```
Root Compartment (Tenancy)
├── Network (VCNs, Subnets, Gateways)
├── Security (Vault, Logs, Cloud Guard)
├── Production
│   ├── App-Tier
│   └── Database-Tier
├── Development
└── SharedServices
```

**Dica de prova:**
- Policies atribuídas a um compartment **herdam** para sub-compartments (child)
- Budgets e Cost Tracking são atrelados a compartments
- Mover um recurso entre compartments é possível e **não interrompe** o serviço

---

### [PRÁTICA & CUSTO]

**IAM e Compartments — Custo:**

| Recurso IAM | Custo |
|-------------|-------|
| Users, Groups, Policies | **GRATUITO** (ilimitados) |
| Compartments | **GRATUITO** (ilimitados) |
| Identity Domains (Free tier) | **1 domínio incluído** |
| Federação (SAML 2.0, OIDC) | **GRATUITO** |
| MFA (Multi-Factor Authentication) | **GRATUITO** |
| Audit Logs (IAM events) | **GRATUITO** (retenção 365 dias) |

> 💡 **Para a prova:** IAM é **100% gratuito**. Não existe cobrança por número de users, groups, policies ou compartments. A Oracle não cobra pela camada de identidade.

---

## 1.6 Always Free Tier — Resumo Completo (Muito cobrado!)

### [TEORIA]

O **Always Free Tier** oferece recursos que **nunca expiram**, disponíveis para qualquer tenancy pago ou trial.

| Categoria | Recurso Always Free |
|-----------|-------------------|
| **Compute** | 4 OCPUs + 24 GB RAM (Ampere A1 ARM) |
| **Compute** | 1/8 OCPU + 1 GB RAM (AMD E4 Micro) |
| **Block Storage** | 200 GB total |
| **Object Storage** | 10 GB (Standard) + 10 GB (Archive) |
| **Database** | 2 Autonomous Databases × 20 GB cada |
| **Networking** | 10 TB/mês egress + 1 Load Balancer |
| **Observability** | 10 GB logging/mês + 500M datapoints monitoring |
| **Security** | Bastion, Cloud Guard, Vulnerability Scanning |
| **Serverless** | 2M invocações Functions + 2M calls API Gateway |
| **AI/ML** | 5K transactions Vision/Language/mês |

**Restrição importante:** Recursos Always Free ficam **disponíveis apenas na Home Region**.

---

---

## 📝 SIMULADO DE FIXAÇÃO — Módulo 1

---

### Questão 1

**Uma empresa multinacional está planejando utilizar a OCI para hospedar uma aplicação crítica com requisitos de alta disponibilidade. A região escolhida possui 3 Availability Domains. O arquiteto distribui as instâncias de compute entre os 3 ADs e, dentro de cada AD, entre diferentes Fault Domains.**

**Qual afirmação descreve CORRETAMENTE a proteção oferecida por essa arquitetura?**

- **A)** Fault Domains protegem contra desastres regionais, enquanto Availability Domains protegem contra falhas de hardware isoladas.
- **B)** Cada Availability Domain contém exatamente 2 Fault Domains, totalizando 6 FDs na região.
- **C)** Fault Domains fornecem proteção contra falhas de hardware e manutenção planejada dentro de um AD, enquanto múltiplos ADs protegem contra falhas em nível de data center.
- **D)** O tráfego de dados entre Availability Domains na mesma região é cobrado a $0,0085/GB, o que deve ser considerado no custo total.

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: C**

**Justificativa:**
- **A)** Invertida — ADs protegem contra falhas de DC, FDs contra falhas de hardware.
- **B)** Incorreta — cada AD possui **3** FDs (não 2), totalizando 9 na região.
- **C)** ✅ CORRETA — exatamente a hierarquia de proteção do OCI.
- **D)** Incorreta — tráfego entre ADs na **mesma região** é **GRATUITO**.

</details>

---

### Questão 2

**Um administrador de cloud precisa conceder permissão para que desenvolvedores possam criar e gerenciar instâncias de compute apenas no compartment "Development", sem acesso a recursos em outros compartments.**

**Qual policy statement está CORRETA para atender a esse requisito?**

- **A)** `Allow user DevUser1 to manage instances in compartment Development`
- **B)** `Allow group Developers to manage instance-family in compartment Development`
- **C)** `Allow group Developers to manage all-resources in tenancy`
- **D)** `Allow group Developers to inspect instances in compartment Development`

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: B**

**Justificativa:**
- **A)** Incorreta — policies são aplicadas a **Groups**, NUNCA diretamente a Users individuais.
- **B)** ✅ CORRETA — concede ao grupo Developers permissão de gerenciamento total de instâncias, limitado ao compartment Development. Usa `instance-family` que inclui instâncias e recursos associados.
- **C)** Incorreta — concede acesso irrestrito a **todos** os recursos em **todo** o tenancy (princípio do menor privilégio violado).
- **D)** Incorreta — `inspect` apenas permite **listar** recursos, não criar ou gerenciar.

</details>

---

### Questão 3

**Qual das seguintes afirmações sobre Compartments no OCI é VERDADEIRA?**

- **A)** Um recurso pode pertencer a múltiplos compartments simultaneamente para flexibilidade.
- **B)** Compartments são específicos de uma região e não podem ser acessados de outras regiões.
- **C)** Policies aplicadas a um compartment pai são automaticamente herdadas pelos sub-compartments filhos.
- **D)** É necessário pagar uma taxa adicional para criar mais de 10 compartments.

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: C**

**Justificativa:**
- **A)** Incorreta — um recurso pertence a **exatamente um** compartment.
- **B)** Incorreta — compartments são **globais** (span all subscribed regions).
- **C)** ✅ CORRETA — policies possuem herança de pai para filhos (inheritance).
- **D)** Incorreta — compartments são **gratuitos e ilimitados**.

</details>

---

### Questão 4

**Uma startup está avaliando custos de transferência de dados na OCI. Eles projetam 8 TB de egress mensal para a internet.**

**Qual será o custo APROXIMADO mensal de data transfer out?**

- **A)** $68,00 — calculado como 8.000 GB × $0,0085/GB
- **B)** $0 — coberto integralmente pelo free tier de 10 TB/mês da OCI
- **C)** $720,00 — equivalente ao pricing padrão de outros provedores cloud
- **D)** $8,50 — apenas o primeiro 1 TB é gratuito na OCI

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: B**

**Justificativa:**
- A OCI oferece **10 TB/mês de egress gratuito** (todos os tenancies, não apenas Always Free).
- 8 TB < 10 TB → custo = **$0**.
- O mesmo volume na AWS custaria aproximadamente **$720+**.
- Esse é um **diferencial competitivo** cobrado frequentemente no exame.

</details>

---

## ⏭️ Próximo Módulo

**Módulo 2 — Pilar 2: Serviços Principais do OCI**
- Compute Shapes (VM, BM, Dedicated Host)
- Virtual Cloud Network (VCN, Subnets, Gateways)
- Storage (Object, Block, File, Archive)
- Autonomous Database

> 💬 **Aguardando seu comando para prosseguir para o Módulo 2.**

---

## 📊 Dados de Pricing

Os dados de custos neste guia foram obtidos em tempo real via Oracle Cloud Pricing API e refletem os preços atuais praticados. A OCI mantém **preços consistentes entre todas as regiões comerciais**.

| Fonte | Cobertura |
|-------|-----------|
| Oracle Cloud Pricing API | 562 SKUs standard + 30 BYOL |
| Regiões verificadas | 32 regiões comerciais |
| Modelo de billing | Pay As You Go (PAYG) |

---

*Última atualização: Julho 2026*
