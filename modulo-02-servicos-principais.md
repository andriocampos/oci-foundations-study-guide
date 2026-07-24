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

| Shape | GPUs | Tipo GPU | OCPUs | RAM | $/hora | $/mês |
|-------|------|----------|-------|-----|--------|-------|
| BM.GPU.A10.4 | 4 | NVIDIA A10 | 64 | 1.024 GB | $2,95 | ~$2.154 |
| BM.GPU.A100-v2.8 | 8 | NVIDIA A100 80GB | 128 | 2.048 GB | $17,00 | ~$12.410 |
| BM.GPU.H100.8 | 8 | NVIDIA H100 80GB | 112 | 2.048 GB | $36,00 | ~$26.280 |

---

### 2.1.5 Preemptible Instances e Capacity Reservations

| Recurso | Benefício | Consideração |
|---------|-----------|--------------|
| **Preemptible** | 50% desconto | Pode ser reclamada com 30s de aviso |
| **Capacity Reservation** | Garante disponibilidade | Paga mesmo sem usar |
| **Burstable** | Baseline CPU 12.5%/50% | Bursts temporários para 100% |

---

### [PRÁTICA & CUSTO — Compute]

**Cenário 1 — Always Free:**

| Recurso | Configuração | Custo |
|---------|-------------|-------|
| VM.Standard.A1.Flex | 4 OCPUs, 24 GB RAM | **$0** |

**Cenário 2 — API Server Produção (E5 Flex, 4 OCPU, 32 GB):**

| Item | Cálculo | Custo/mês |
|------|---------|-----------|
| OCPUs | 4 × $0,030 × 730h | $87,60 |
| Memória | 32 × $0,002 × 730h | $46,72 |
| **Total** | | **$134,32** |

**Cenário 3 — GPU H100 (ML Training, 8h/dia × 20 dias):**

| Item | Cálculo | Custo/mês |
|------|---------|-----------|
| BM.GPU.H100.8 | 160h × $36,00 | **$5.760,00** |
| vs 24/7 | 730h × $36,00 | $26.280,00 |
| **Economia** | | **78%** |

---

---

## 2.2 Virtual Cloud Network (VCN)

### [TEORIA]

**Definição Oficial:** Uma Virtual Cloud Network (VCN) é uma rede privada definida por software que você configura no OCI. Ela se assemelha a uma rede de data center tradicional, com regras de firewall e tipos específicos de gateways de comunicação.

---

### 2.2.1 Componentes da VCN (Todos cobrados no exame!)

| Componente | Descrição | Custo |
|------------|-----------|-------|
| **VCN** | Rede virtual (CIDR block, ex: 10.0.0.0/16) | **FREE** |
| **Subnet** | Subdivisão da VCN (pública ou privada) | **FREE** |
| **Route Table** | Regras de roteamento de tráfego | **FREE** |
| **Security List** | Firewall stateful no nível da subnet | **FREE** |
| **NSG (Network Security Group)** | Firewall no nível da VNIC (mais granular) | **FREE** |
| **Internet Gateway** | Acesso bidirecional à internet (public subnet) | **FREE** |
| **NAT Gateway** | Acesso unidirecional à internet (private subnet → internet) | $8,25/mês |
| **Service Gateway** | Acesso privado a OCI services (sem internet) | **FREE** |
| **DRG (Dynamic Routing Gateway)** | Conecta VCN a on-premises ou peering | **FREE** |
| **Local Peering Gateway (LPG)** | Conecta VCNs na mesma região | **FREE** |
| **Remote Peering** | Conecta VCNs em regiões diferentes (via DRG) | **FREE** |

---

### 2.2.2 Subnets — Pública vs Privada

| Tipo | Acesso Internet | Uso Típico |
|------|----------------|------------|
| **Pública** | Sim (via Internet Gateway) | Web servers, bastion hosts |
| **Privada** | Não direto (via NAT Gateway para saída) | Databases, app servers, backends |

**Escopo da Subnet:**
- **Regional** (recomendado): span all ADs na região
- **AD-specific** (legacy): limitada a um Availability Domain

> 💡 **Para a prova:** Subnets regionais são a **recomendação atual**. Permitem alta disponibilidade sem criar subnet por AD.

---

### 2.2.3 Security Lists vs Network Security Groups (NSG)

| Aspecto | Security List | NSG |
|---------|--------------|-----|
| Nível | **Subnet** (todas as VNICs) | **VNIC** (instância específica) |
| Granularidade | Menor | **Maior** (recomendado) |
| Associação | 1 subnet → múltiplas SLs | 1 VNIC → até 5 NSGs |
| Regras | Ingress + Egress | Ingress + Egress |
| Stateful | Sim (default) | Sim (default) |

> 💡 **Para a prova:** Oracle **recomenda NSGs** sobre Security Lists para controle mais granular.

---

### 2.2.4 Gateways — Qual usar e quando

```
Internet (público)
    │
    ▼
┌─────────────────┐
│ Internet Gateway │ ← para subnets PÚBLICAS
└────────┬────────┘
         │
    ┌────┴────┐
    │   VCN   │
    └────┬────┘
         │
┌────────┴────────┐
│  NAT Gateway    │ ← subnet privada → internet (só saída)
└─────────────────┘
         │
┌────────┴────────┐
│ Service Gateway │ ← subnet privada → OCI Services (sem internet)
└─────────────────┘
         │
┌────────┴────────┐
│      DRG        │ ← VCN ↔ On-premises (VPN/FastConnect)
└─────────────────┘
```

**Tabela de decisão (frequente no exame):**

| Cenário | Gateway |
|---------|---------|
| Web server acessível pela internet | **Internet Gateway** |
| DB server precisa baixar patches da internet | **NAT Gateway** |
| App server acessa Object Storage sem internet | **Service Gateway** |
| Conexão com data center on-premises | **DRG + VPN ou FastConnect** |
| Conectar duas VCNs na mesma região | **Local Peering Gateway (LPG)** |
| Conectar duas VCNs em regiões diferentes | **DRG + Remote Peering** |

---

### 2.2.5 Load Balancers

| Tipo | Camada | Protocolos | Custo |
|------|--------|-----------|-------|
| **Network Load Balancer** | Layer 4 | TCP/UDP | **FREE** |
| **Flexible Load Balancer** | Layer 7 | HTTP/HTTPS | $8,25/mês + bandwidth |

**Flexible Load Balancer — Detalhes:**
- Base: $0,0113/hora (~$8,25/mês)
- Bandwidth: $0,0001/Mbps/hora ($7,30/mês para 100 Mbps)
- **Always Free:** 1 instância + 10 Mbps

**Network Load Balancer:**
- **100% gratuito** (sem cobrança por instância ou dados)
- Ideal para tráfego TCP/UDP de alta performance
- Suporta milhões de conexões simultâneas

> 💡 **Dica de prova:** NLB é GRATUITO e opera em Layer 4. Flex LB opera em Layer 7 (HTTP/HTTPS) e tem custo. O exame testa essa diferença.

---

### 2.2.6 Conectividade Híbrida

| Serviço | Bandwidth | Custo/mês | Uso |
|---------|-----------|-----------|-----|
| **Site-to-Site VPN** | Até 250 Mbps | $18,25 | Conexão criptografada básica |
| **FastConnect 1 Gbps** | 1 Gbps dedicado | $219 | Produção, latência previsível |
| **FastConnect 10 Gbps** | 10 Gbps dedicado | $496 | Alta demanda |
| **FastConnect 100 Gbps** | 100 Gbps dedicado | $3.723 | Data centers massivos |

**FastConnect vs VPN:**

| Aspecto | VPN | FastConnect |
|---------|-----|-------------|
| Tipo | Internet (criptografado) | Dedicado (privado) |
| Latência | Variável | **Consistente e baixa** |
| Bandwidth | Até 250 Mbps | 1–100 Gbps |
| Custo | $18/mês | $219–$3.723/mês |
| SLA | Sem SLA de latência | **SLA de latência** |

---

### [PRÁTICA & CUSTO — VCN]

**Cenário — Arquitetura Web 3-Tier:**

| Componente | Custo/mês |
|------------|-----------|
| 1 VCN (10.0.0.0/16) | **$0** |
| 3 Subnets (public + 2 private) | **$0** |
| 1 Internet Gateway | **$0** |
| 1 NAT Gateway | $8,25 |
| 1 Service Gateway | **$0** |
| 1 Network Load Balancer (L4) | **$0** |
| Route Tables + Security Lists | **$0** |
| Egress (5 TB/mês) | **$0** (dentro do free tier) |
| **Total Networking** | **$8,25/mês** |

> 💡 **Para a prova:** Uma arquitetura VCN completa com LB pode custar apenas **$8,25/mês** (só o NAT Gateway). Quase tudo é gratuito!

---
