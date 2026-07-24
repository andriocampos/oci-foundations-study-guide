# 🖥️ MÓDULO 2 — Parte 2: Storage e Database

> Continuação do [Módulo 2 - Serviços Principais](modulo-02-servicos-principais.md)

---

## 2.3 Storage Services

### [TEORIA]

A OCI oferece 4 tipos principais de armazenamento, cada um para um caso de uso específico. O exame **sempre** testa qual tipo de storage usar em cada cenário.

---

### 2.3.1 Tipos de Storage — Visão Geral

| Tipo | Protocolo | Persistência | Caso de Uso |
|------|-----------|-------------|-------------|
| **Block Volume** | iSCSI/NVMe | Persistente (independente da VM) | Discos de dados, databases |
| **Boot Volume** | iSCSI | Persistente (disco de OS) | Sistema operacional |
| **Object Storage** | HTTP/REST | Persistente (ilimitado) | Backups, mídia, data lake |
| **File Storage** | NFSv3 | Persistente (compartilhado) | Filesystems compartilhados |

**Decisão rápida (frequente no exame):**

| Preciso de... | Usar |
|---------------|------|
| Disco para database (IOPS) | **Block Volume** |
| Armazenar backups/imagens/vídeos | **Object Storage** |
| Filesystem compartilhado entre VMs | **File Storage** |
| Dados que raramente acesso (compliance) | **Object Storage Archive** |
| Disco do sistema operacional | **Boot Volume** |

---

### 2.3.2 Block Volume — Tiers de Performance

| Tier | VPUs/GB | IOPS máx | Throughput máx | $/GB/mês | $/TB/mês |
|------|---------|----------|----------------|----------|----------|
| **Lower Cost** | 2 | 3.000 | 24 MB/s | $0,0255 | $26,11 |
| **Balanced** (default) | 10 | 25.000 | 480 MB/s | $0,0255 | $26,11 |
| **Higher Performance** | 20 | 35.000 | 480 MB/s | $0,0425 | $43,52 |
| **Ultra High Performance** | 30–120 | 225.000 | 2,68 GB/s | $0,0765 | $78,34 |

**Conceitos importantes:**
- **VPU (Volume Performance Unit):** Unidade que determina IOPS e throughput
- Block Volumes são **independentes** da instância (sobrevivem ao terminate)
- Podem ser **attached/detached** de instâncias
- Suportam **resize online** (aumentar sem downtime)
- **Multi-attach:** Um volume pode ser attached em múltiplas instâncias (read-only ou read-write com cluster)

**Boot Volume:**
- Preço: $0,0255/GB/mês (mesmo que Block Volume Balanced)
- Default: 47 GB para maioria das imagens
- Pode ser aumentado até 32 TB
- **Boot Volume Backup:** Cria backup completo da instância

> 💡 **Para a prova:** Lower Cost e Balanced têm o **mesmo preço** ($0,0255/GB). A diferença é que Balanced inclui 10 VPUs e oferece 25K IOPS vs 3K do Lower Cost. **Sempre escolha Balanced** a menos que precise economizar em VPUs.

---

### 2.3.3 Object Storage — Tiers

| Tier | Acesso | $/GB/mês | $/TB/mês | Mínimo | Restore |
|------|--------|----------|----------|--------|---------|
| **Standard** | Frequente (hot) | $0,0255 | $26,11 | Nenhum | Imediato |
| **Infrequent Access** | < 1x/mês | $0,0100 | $10,24 | 31 dias | Imediato |
| **Archive** | Raramente (cold) | $0,00255 | $2,61 | 90 dias | ~1 hora |

**Conceitos-chave:**
- **Buckets:** Container para objetos (namespace único global)
- **Namespace:** Identificador único do tenancy para Object Storage
- Objetos são acessados via **HTTP/HTTPS REST API** ou **Console/CLI**
- **Versionamento:** Mantém versões anteriores de objetos
- **Lifecycle Rules:** Move objetos automaticamente entre tiers
- **Replication:** Cross-region replication para DR
- **Pre-Authenticated Requests (PAR):** URL temporária com acesso sem autenticação
- Suporta objetos de até **10 TB** (multipart upload)

**Always Free:**
- 10 GB Standard tier
- 10 GB Archive tier
- 50.000 API requests/mês

> 💡 **Para a prova:** Object Storage é **regional** (replicado entre ADs automaticamente). Archive é **10x mais barato** que Standard mas exige restore de ~1 hora.

---

### 2.3.4 File Storage (FSS)

| Característica | Detalhe |
|---------------|---------|
| Protocolo | NFSv3 |
| Preço | $0,0425/GB/mês ($43,52/TB) |
| Capacidade | Elástica (cresce automaticamente) |
| Compartilhamento | Múltiplas instâncias simultâneas |
| Replicação | Multi-AD disponível |
| Snapshot | Suportado |

**Quando usar File Storage:**
- Aplicações que precisam de **filesystem POSIX compartilhado**
- Migração lift-and-shift de NFS on-premises
- Ambientes de desenvolvimento compartilhados
- Content Management Systems (CMS)

> 💡 **Para a prova:** File Storage é o **único** tipo de storage NFS-compatível. Se a questão menciona "NFS" ou "filesystem compartilhado entre instâncias", a resposta é File Storage.

---

### [PRÁTICA & CUSTO — Storage]

**Comparativo de custo para 1 TB:**

| Tipo | $/TB/mês | Caso de uso |
|------|----------|-------------|
| Block Volume (Balanced) | $26,11 | Discos de dados |
| Object Storage Standard | $26,11 | Backups, mídia |
| Object Storage Infrequent | $10,24 | Acesso raro |
| Object Storage Archive | **$2,61** | Compliance, long-term |
| File Storage | $43,52 | NFS compartilhado |
| Block Volume Ultra High | $78,34 | Databases críticas |

**Cenário — Empresa com 10 TB de dados:**

| Dados | Tipo | Custo/mês |
|-------|------|-----------|
| 2 TB databases (alta IOPS) | Block Ultra High | $156,68 |
| 3 TB arquivos compartilhados | File Storage | $130,56 |
| 3 TB backups | Object Standard | $78,33 |
| 2 TB arquivo morto (compliance) | Object Archive | **$5,22** |
| **Total 10 TB** | | **$370,79/mês** |

vs. tudo em Block Balanced: 10 TB × $26,11 = $261,10/mês (mais barato mas sem otimização por caso de uso)

---

---

## 2.4 Database Services

### [TEORIA]

A OCI oferece serviços de banco de dados gerenciados que vão desde NoSQL serverless até Exadata dedicado. O exame foca especialmente no **Autonomous Database**.

---

### 2.4.1 Autonomous Database — O carro-chefe da OCI!

**Definição Oficial:** O Oracle Autonomous Database é um serviço de banco de dados em nuvem que usa machine learning para automatizar tuning, segurança, backups, updates e outras tarefas rotineiras de gerenciamento.

**Tipos de workload:**

| Tipo | Sigla | Otimizado para |
|------|-------|---------------|
| **Autonomous Transaction Processing** | ATP | OLTP, aplicações transacionais |
| **Autonomous Data Warehouse** | ADW | Analytics, BI, data warehouse |
| **Autonomous JSON Database** | AJD | Documentos JSON, APIs |
| **APEX Application Development** | APEX | Low-code development |

**Características "Self" (MUITO cobrado!):**

| Feature | O que faz |
|---------|-----------|
| **Self-Driving** | Provisiona, patching, upgrade, tuning automáticos |
| **Self-Securing** | Criptografia automática, patching de segurança |
| **Self-Repairing** | Detecção e correção automática de falhas |

**Modelos de deployment:**

| Modelo | Descrição |
|--------|-----------|
| **Serverless (Shared)** | Infraestrutura compartilhada, scale automático |
| **Dedicated** | Exadata dedicado, isolamento completo |

**ECPU vs OCPU:**
- O Autonomous Database agora usa **ECPUs** (Elastic CPUs)
- ECPUs oferecem granularidade mais fina que OCPUs
- Mínimo: **2 ECPUs** para Autonomous Serverless
- Auto-scaling pode **triplicar** os ECPUs base (sem intervenção)

---

### 2.4.2 Pricing — Autonomous Database

| Tipo | Compute (ECPU/hr) | Storage (GB/mês) | Exemplo 2 ECPU + 100 GB |
|------|-------------------|------------------|--------------------------|
| ATP/ADW (License Included) | $0,336 | $0,0255 | ~$493/mês |
| ATP/ADW (BYOL) | $0,0807 | $0,0255 | ~$120/mês |
| JSON Database | $0,0807 | $0,0255 | ~$120/mês |

**BYOL (Bring Your Own License):**
- Economia de **~76%** em compute se possui licenças Oracle existentes
- Storage mantém o mesmo preço ($0,0255/GB/mês)

> 💡 **Para a prova:** BYOL é a opção mais econômica para empresas que **já possuem** licenças Oracle Database. O exame testa cenários de escolha entre License Included vs BYOL.

---

### 2.4.3 MySQL HeatWave

| Característica | Detalhe |
|---------------|---------|
| Compatibilidade | MySQL 8.0 |
| Diferencial | **HeatWave** — engine in-memory para analytics em tempo real |
| Compute | $0,0336/OCPU/hora |
| Storage | $0,0255/GB/mês |
| Exemplo (2 OCPU, 100 GB) | ~$52/mês |

**HeatWave — O diferencial:**
- Acelera queries analíticas **400x** vs MySQL padrão
- ML integrado (HeatWave ML) sem mover dados
- Sem custo extra para o engine HeatWave em queries

---

### 2.4.4 PostgreSQL (OCI Database with PostgreSQL)

| Característica | Detalhe |
|---------------|---------|
| Versões | PostgreSQL 14, 15, 16 |
| Compute | $0,0336/OCPU/hora |
| Storage | $0,0255/GB/mês |
| HA | Multi-AD replication disponível |
| Exemplo (2 OCPU, 100 GB) | ~$52/mês |

---

### 2.4.5 NoSQL Database

| Característica | Detalhe |
|---------------|---------|
| Modelo | Key-value, document, column-family |
| Pricing | On-demand (pay per request) |
| Read | $0,000001/request |
| Write | $0,000001/request |
| Storage | $0,025/GB/mês |
| **Always Free** | **25 GB + 200M reads + 50M writes/mês** |

---

### 2.4.6 Base Database Service

| Modelo | Compute (OCPU/hr) | Storage | Uso |
|--------|-------------------|---------|-----|
| VM (License Included) | $0,168 | $0,0255/GB/mês | Oracle DB on VM |
| VM (BYOL) | $0,025 | $0,0255/GB/mês | Traga sua licença |

**Edições disponíveis:**
- Standard Edition
- Enterprise Edition
- Enterprise Edition High Performance
- Enterprise Edition Extreme Performance (inclui RAC, Data Guard)

---

### 2.4.7 Exadata Cloud Service

| Característica | Detalhe |
|---------------|---------|
| Performance | Máxima (hardware otimizado para Oracle DB) |
| Compute | $0,336/OCPU/hora |
| Deployment | Quarter Rack mínimo |
| Uso | Mission-critical OLTP e analytics |

---

### [PRÁTICA & CUSTO — Database]

**Comparativo para workload OLTP (2 unidades de compute + 100 GB):**

| Serviço | Custo/mês | Melhor para |
|---------|-----------|-------------|
| Autonomous ATP (License Included) | ~$493 | Oracle features completas, zero DBA |
| Autonomous ATP (BYOL) | ~$120 | Já possui licença Oracle |
| MySQL HeatWave | ~$52 | MySQL com analytics |
| PostgreSQL | ~$52 | Workloads open-source |
| NoSQL | ~$2,55 | Key-value / documentos |
| Base DB VM (BYOL) | ~$39 | Controle total, traga licença |

**Always Free Database:**

| Recurso | Limite |
|---------|--------|
| Autonomous Database | **2 instâncias × 20 GB cada** |
| NoSQL | 25 GB + 200M reads + 50M writes/mês |

> 💡 **Para a prova:** O Autonomous Database **Always Free** é uma das respostas mais testadas. 2 databases com 20 GB cada, disponíveis na Home Region, sem expiração.

---

---

## 📝 SIMULADO DE FIXAÇÃO — Módulo 2

---

### Questão 1

**Uma empresa precisa provisionar uma instância de compute na OCI para rodar um servidor web nginx. A equipe quer flexibilidade para ajustar CPU e memória independentemente, e precisa do menor custo possível. A aplicação é compatível com processadores ARM.**

**Qual shape é o mais adequado?**

- **A)** VM.Standard.E5.Flex — pois é a geração mais recente de AMD
- **B)** VM.Standard.A1.Flex — pois é ARM com o menor custo por OCPU ($0,01/hr)
- **C)** BM.Standard.A1.160 — pois ARM bare metal oferece máxima performance
- **D)** VM.Optimized3.Flex — pois tem o clock mais alto para web servers

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: B**

**Justificativa:**
- **A)** E5 Flex é excelente mas custa $0,030/OCPU/hr (3x mais caro que A1)
- **B)** ✅ CORRETA — A1 Flex é ARM, flexível, e custa apenas $0,01/OCPU/hr. nginx roda perfeitamente em ARM. Além disso, 4 OCPUs + 24 GB entram no **Always Free**.
- **C)** Bare Metal é overkill para um web server e não permite ajuste flexível
- **D)** Optimized3 custa $0,054/OCPU/hr (5,4x mais caro) e é para workloads single-thread intensivos

</details>

---

### Questão 2

**Um arquiteto está desenhando a rede para uma aplicação 3-tier na OCI. O tier web precisa ser acessível pela internet. O tier de aplicação precisa acessar a internet para baixar pacotes. O tier de banco de dados precisa acessar o Object Storage sem tráfego passando pela internet pública.**

**Quais gateways devem ser configurados?**

- **A)** Internet Gateway + NAT Gateway + Service Gateway
- **B)** Internet Gateway + Internet Gateway + NAT Gateway
- **C)** NAT Gateway + NAT Gateway + Service Gateway
- **D)** Internet Gateway + Service Gateway + DRG

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: A**

**Justificativa:**
- **Web tier** (acessível da internet) → **Internet Gateway** (acesso bidirecional)
- **App tier** (acessa internet para downloads) → **NAT Gateway** (saída sem exposição)
- **DB tier** (acessa Object Storage sem internet) → **Service Gateway** (path privado para OCI services)
- **B)** Incorreta — dois Internet Gateways não fazem sentido, e IG expõe o app tier
- **C)** Incorreta — NAT não permite acesso **inbound** para o web tier
- **D)** Incorreta — DRG é para on-premises, não para acesso a OCI services

</details>

---

### Questão 3

**Uma empresa precisa armazenar 5 TB de logs de auditoria por 7 anos para compliance regulatória. Os dados serão acessados no máximo 1-2 vezes por ano.**

**Qual tipo de armazenamento é o mais econômico?**

- **A)** Block Volume (Lower Cost tier) — $0,0255/GB/mês
- **B)** Object Storage Standard — $0,0255/GB/mês
- **C)** Object Storage Archive — $0,00255/GB/mês
- **D)** File Storage — $0,0425/GB/mês

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: C**

**Justificativa:**
- 5 TB × 7 anos = armazenamento de longo prazo
- Acesso 1-2x/ano = dados frios (cold)
- **C)** ✅ Archive = $0,00255/GB/mês = **$13,05/mês** para 5 TB
- **A)** Block Volume = $130/mês (10x mais caro e inadequado para logs)
- **B)** Standard = $130/mês (desnecessário para acesso raro)
- **D)** File Storage = $217/mês (mais caro e não adequado para archival)
- Archive tem restore de ~1 hora, aceitável para 1-2 acessos/ano

</details>

---

### Questão 4

**Qual afirmação sobre o Oracle Autonomous Database está CORRETA?**

- **A)** O Autonomous Database requer que o DBA execute patching e tuning manualmente
- **B)** O modelo BYOL (Bring Your Own License) é mais caro que License Included, mas oferece mais features
- **C)** O Autonomous Database oferece auto-scaling que pode triplicar os ECPUs base automaticamente, sem intervenção do administrador
- **D)** O Always Free tier inclui 5 Autonomous Databases com 100 GB cada

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: C**

**Justificativa:**
- **A)** Incorreta — o Autonomous é "Self-Driving": patching, tuning e upgrades são **automáticos**
- **B)** Incorreta — BYOL é **~76% mais barato** (você traz sua licença existente)
- **C)** ✅ CORRETA — Auto-scaling pode escalar até 3x os ECPUs base de forma automática
- **D)** Incorreta — Always Free inclui **2** databases com **20 GB** cada (não 5 com 100 GB)

</details>

---

### Questão 5

**Uma empresa transfere 15 TB de dados por mês para a internet a partir da OCI. Qual é o custo mensal de data transfer?**

- **A)** $0 — todo tráfego de saída é gratuito na OCI
- **B)** $127,50 — calculado como 15.000 GB × $0,0085/GB
- **C)** $42,50 — apenas os 5 TB excedentes são cobrados (15 TB - 10 TB free = 5 TB × $0,0085)
- **D)** $1.350,00 — equivalente ao preço de mercado dos concorrentes

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: C**

**Justificativa:**
- OCI: primeiros **10 TB são gratuitos**
- 15 TB - 10 TB = **5 TB excedentes** (5.120 GB)
- 5.120 GB × $0,0085 = **$43,52/mês**
- **A)** Incorreta — nem todo tráfego é grátis, apenas os primeiros 10 TB
- **B)** Incorreta — não considera o free tier de 10 TB
- **C)** ✅ CORRETA — $42,50 ≈ $43,52 (5 TB × $0,0085/GB)
- **D)** Incorreta — isso seria o preço na AWS, não na OCI

</details>

---

### Questão 6

**Qual é a relação CORRETA entre OCPU e vCPU na OCI?**

- **A)** 1 OCPU = 1 vCPU (são equivalentes)
- **B)** 1 OCPU = 2 vCPUs (1 core físico com hyper-threading)
- **C)** 1 OCPU = 4 vCPUs (quad-threading)
- **D)** A relação varia dependendo do shape escolhido

<details>
<summary>🔑 Clique para ver a resposta</summary>

**Resposta: B**

**Justificativa:**
- 1 OCPU = 1 core físico com hyper-threading = **2 vCPUs**
- Isso significa que 4 OCPUs OCI = 8 vCPUs (equivalente em AWS/Azure)
- Essa relação é **fixa** para shapes x86 (AMD/Intel)
- Para ARM (Ampere), 1 OCPU = 1 core (sem hyper-threading), mas Oracle ainda conta como 1 OCPU

</details>

---

## ⏭️ Próximo Módulo

**Módulo 3 — Pilar 3: Segurança e Conformidade**
- Modelo de Responsabilidade Compartilhada
- WAF (Web Application Firewall)
- Vault (Key Management)
- Identity Domains
- Cloud Guard
- Data Safe

> 💬 **Aguardando seu comando para prosseguir para o Módulo 3.**

---

*Dados de pricing verificados em Julho 2026 via Oracle Cloud Pricing API*
