# 🖥️ MÓDULO 2 — Pilar 2: Serviços Principais do OCI

> **Peso no exame:** ~35% das questões  
> **Tópicos:** Compute, Networking (VCN), Storage, Database  
> **Dados de pricing:** Obtidos em tempo real via Oracle Cloud Pricing API (Julho 2026)

---

## 📋 Índice do Módulo 2

1. [Compute Shapes](#21-compute-shapes)
2. [Virtual Cloud Network (VCN)](#22-virtual-cloud-network-vcn)
3. [Storage Services](#23-storage-services)
4. [Database Services](#24-database-services)
5. [Simulado de Fixação](#-simulado-de-fixação--módulo-2)

---

---

## 2.1 Compute Shapes

### [TEORIA]

**Definição Oficial:** Um Compute Shape determina o número de OCPUs, a quantidade de memória e outros recursos alocados a uma instância de Compute. A OCI oferece shapes flexíveis (configuráveis), fixos, bare metal e GPU.

---

### 2.1.1 Conceitos Fundamentais

**OCPU vs vCPU (MUITO cobrado no exame):**

| Conceito | Definição |
|----------|-----------|
| **OCPU** | 1 core físico com hyper-threading. Equivale a **2 vCPUs** em x86 |
| **vCPU** | Thread virtual. AWS/Azure usam vCPU; OCI usa OCPU |
| **Conversão** | 1 OCPU (OCI) = 2 vCPUs (AWS/Azure) |

> 💡 **Para a prova:** Quando o exame compara OCI com outros clouds, lembre que 4 OCPUs OCI = 8 vCPUs AWS.

**Tipos de Instâncias:**

| Tipo | Característica | Uso |
|------|---------------|-----|
| **VM (Virtual Machine)** | Compartilha hardware com outras VMs | Workloads gerais |
| **Bare Metal (BM)** | Hardware dedicado, sem hypervisor | Performance máxima, compliance |
| **Dedicated VM Host** | Host dedicado com VMs isoladas | Licenciamento, regulação |

**Tipos de Shapes:**

| Categoria | Descrição |
|-----------|-----------|
| **Flex (Flexible)** | OCPU e memória configuráveis independentemente |
| **Standard** | Uso geral, balanceado |
| **Optimized** | Clock mais alto para workloads single-threaded |
| **DenseIO** | NVMe local de alta performance |
| **GPU** | Aceleradores NVIDIA para AI/ML |
| **HPC** | High Performance Computing com RDMA |

---

### 2.1.2 Shapes Flexíveis (Flex) — O mais cobrado!

**Conceito:** Shapes Flex permitem escolher **independentemente** o número de OCPUs e a quantidade de memória (dentro dos limites do shape).

**Regra de memória:** De **1 GB até 64 GB por OCPU** (proporção configurável).

**Tabela completa de Flex Shapes (dados reais):**

| Shape | Processador | OCPU $/hr | Memória $/GB/hr | OCPUs | RAM máx |
|-------|-------------|-----------|-----------------|-------|---------|
| **VM.Standard.E4.Flex** | AMD EPYC Milan | $0,025 | $0,0015 | 1–64 | 1.024 GB |
| **VM.Standard.E5.Flex** | AMD EPYC Genoa | $0,030 | $0,0020 | 1–94 | 1.049 GB |
| **VM.Standard.E6.Flex** | AMD EPYC Turin | $0,030 | $0,0020 | 1–94 | 1.049 GB |
| **VM.Standard3.Flex** | Intel Xeon Ice Lake | $0,040 | $0,0024 | 1–32 | 512 GB |
| **VM.Standard.A1.Flex** | Ampere Altra (ARM) | **$0,010** | **$0,0006** | 1–80 | 512 GB |
| **VM.Standard.A2.Flex** | Ampere Altra Max (ARM) | $0,014 | $0,0020 | 1–80 | 512 GB |
| **VM.Standard.A4.Flex** | NVIDIA Grace (ARM) | $0,0138 | $0,0027 | 1–72 | 512 GB |
| **VM.Optimized3.Flex** | Intel Xeon (High freq) | $0,054 | $0,0032 | 1–18 | 256 GB |
| **VM.DenseIO.E4.Flex** | AMD EPYC + NVMe | $0,034 | $0,0020 | 8–32 | 512 GB |

> 🏆 **Melhor custo-benefício:** VM.Standard.A1.Flex (ARM) a $0,01/OCPU/hr — **60% mais barato** que E4.

---

### 2.1.3 Bare Metal Shapes

**Conceito:** Instâncias dedicadas sem hypervisor. O servidor físico inteiro é do cliente.

**Quando usar Bare Metal:**
- Workloads que exigem **compliance** (sem multi-tenancy)
- Performance máxima **sem overhead de virtualização**
- Licenciamento que exige contagem de **cores físicos**
- Databases de alta performance (Oracle RAC, etc.)

| Shape | Processador | OCPUs | RAM | $/hora | $/mês |
|-------|-------------|-------|-----|--------|-------|
| BM.Standard.E4.128 | AMD EPYC Milan | 128 | 2.048 GB | $3,84 | ~$2.803 |
| BM.Standard.E5.192 | AMD EPYC Genoa | 192 | 2.304 GB | $5,76 | ~$4.205 |
| BM.Standard.A1.160 | Ampere Altra (ARM) | 160 | 1.024 GB | $1,60 | ~$1.168 |

---

### 2.1.4 GPU Shapes

**Conceito:** Instâncias com aceleradores NVIDIA para AI/ML, HPC e inferência.

| Shape | GPUs | Tipo GPU | OCPUs | RAM | $/hora | $/mês |
|-------|------|----------|-------|-----|--------|-------|
| BM.GPU.A10.4 | 4 | NVIDIA A10 | 64 | 1.024 GB | $2,95 | ~$2.154 |
| BM.GPU.A100-v2.8 | 8 | NVIDIA A100 80GB | 128 | 2.048 GB | $17,00 | ~$12.410 |
| BM.GPU.H100.8 | 8 | NVIDIA H100 80GB | 112 | 2.048 GB | $36,00 | ~$26.280 |

**Casos de uso por GPU:**

| GPU | Uso Principal |
|-----|--------------|
| A10 | Inferência, rendering, workloads gráficos |
| A100 | Treinamento de modelos grandes, HPC |
| H100 | LLMs, treinamento de última geração, GenAI |

---

### 2.1.5 Preemptible Instances e Capacity Reservations

**Preemptible (Preemptíveis):**
- **50% de desconto** sobre o preço regular
- Podem ser **reclamadas pela OCI** com aviso de 30 segundos
- Ideais para: batch processing, CI/CD, workloads tolerantes a falha
- **NÃO** têm SLA de disponibilidade

**Capacity Reservations:**
- Reserva capacidade dedicada em um AD/FD específico
- Garante disponibilidade para **scale-out planejado**
- Cobrado mesmo se não utilizado (pay for reserved, not used)

**Burstable Instances:**
- Baseline CPU de 12.5% ou 50% (shapes E3/E4 Micro)
- Pode "burst" para 100% por períodos curtos
- Ideal para workloads com picos esporádicos

---

### [PRÁTICA & CUSTO]

**Cenário 1 — Aplicação Web Pequena (Always Free):**

| Recurso | Configuração | Custo |
|---------|-------------|-------|
| VM.Standard.A1.Flex | 4 OCPUs, 24 GB RAM | **$0 (Always Free)** |
| Boot Volume | 47 GB | **$0 (dentro dos 200 GB free)** |
| **Total mensal** | | **$0** |

**Cenário 2 — API Server Produção (E5 Flex):**

| Recurso | Configuração | Custo/mês |
|---------|-------------|-----------|
| VM.Standard.E5.Flex | 4 OCPUs, 32 GB RAM | |
| → OCPUs | 4 × $0,030 × 730h | $87,60 |
| → Memória | 32 × $0,002 × 730h | $46,72 |
| Block Volume (500 GB) | Balanced tier | $12,75 |
| **Total mensal** | | **$147,07** |

**Cenário 3 — ML Training (GPU H100):**

| Recurso | Configuração | Custo/mês |
|---------|-------------|-----------|
| BM.GPU.H100.8 | 8× H100, 112 OCPUs | $26.280,00 |
| Block Volume (2 TB) | High Performance | $85,00 |
| **Total mensal (24/7)** | | **$26.365,00** |
| **Total (8h/dia, 20 dias)** | 160h × $36 | **$5.760,00** |

> 💡 **Dica de economia:** Para ML training, desligue instâncias GPU quando não estiver treinando. 160h vs 730h = **78% de economia**.

**Always Free Compute:**

| Shape | Configuração Gratuita |
|-------|----------------------|
| VM.Standard.A1.Flex (ARM) | 4 OCPUs + 24 GB (divisível em até 4 VMs) |
| VM.Standard.E4.Flex (AMD) | 1/8 OCPU + 1 GB (Micro instance) |

---
