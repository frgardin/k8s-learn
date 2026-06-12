# Kubernetes Up and Running - Resumos

## Capítulo 1 - Introduction

### Visão Geral

O Capítulo 1 estabelece os fundamentos do Kubernetes como um orquestrador de contêineres. Explica por que Kubernetes foi criado, quais problemas ele resolve e os princípios arquiteturais que tornam possível construir sistemas distribuídos confiáveis, escaláveis e eficientes. O capítulo conecta conceitos abstratos (como imutabilidade e configuração declarativa) com benefícios práticos para desenvolvimento, operações e negócios.

### O que é Kubernetes?

- **Definição**: Orquestrador open source para implantar aplicações em contêineres
- **Origem**: Desenvolvido pelo Google, inspirado em uma década de experiência com sistemas em contêineres (Borg, Omega)
- **Lançamento**: 2014
- **Escopo**: Adequado desde clusters Raspberry Pi até datacenters com milhares de máquinas
- **Propósito**: Simplificar o desafio de construir sistemas distribuídos **confiáveis, escaláveis e disponíveis**

### Cinco Razões para Usar Kubernetes

1. **Development Velocity (Velocidade de Desenvolvimento)**
   - Antes: Software distribuído em CDs/DVDs, manutenção planejada, uptime não garantido
   - Agora: Serviços web, atualizações horárias, expectativa de uptime 24/7
   - Kubernetes permite iterar rapidamente mantendo confiabilidade alta

2. **Scaling (Escalabilidade de Software e Equipes)**
   - Desacoplamento de componentes via APIs e load balancers
   - Equipes pequenas ("two-pizza teams") podem escalar independentemente
   - Reduz complexidade de comunicação entre times

3. **Abstracting Your Infrastructure (Abstração da Infraestrutura)**
   - Desenvolvedores trabalham com APIs orientadas a aplicações, não máquinas virtuais
   - Portabilidade: mesmo código roda em on-prem, clouds públicas ou ambientes híbridos
   - Evita vendor lock-in

4. **Efficiency (Eficiência Operacional e Econômica)**
   - Consolidação: múltiplas aplicações compartilham máquinas sem impacto
   - Redução de desperdício (CPU ociosa)
   - Testes baratos: criar ambientes de teste em contêineres é rápido e barato
   - Autoscaling automático reduz custos sob demanda

5. **Cloud Native Ecosystem**
   - Kubernetes é extensível e aberto à comunidade
   - Ecossistema vibrante: ferramentas para observabilidade, segurança, storage, CI/CD
   - CNCF fornece maturidade e padronização (sandbox → incubating → graduated)

### Conceitos Arquiteturais Fundamentais

#### **Imutabilidade (Immutability)**

**Problema tradicional**: Infraestrutura mutável = estado desconhecido
- Atualizações incrementais (`apt-get update`) deixam resíduos e histórico não rastreável
- Rollback é complexo e caro
- Mudanças podem vir de upgrades do sistema, modificações de operadores, etc.

**Solução em contêineres**: Infraestrutura imutável
- Ao invés de modificar um contêiner existente, **constrói-se uma imagem nova**
- A imagem antiga permanece disponível para rollback (basta trocar o rótulo)
- Histórico completo de mudanças é visível
- Reproduzibilidade garantida: mesma imagem = mesmo comportamento

**Padrão anti**: Tentar modificar contêineres em produção (mesmo que tecnicamente possível)

#### **Configuração Declarativa (Declarative Configuration)**

**Imperativo vs Declarativo**:
- **Imperativo**: "Execute estes passos" → "rode A, rode B, rode C" (ordem importa, frágil a falhas)
- **Declarativo**: "Desejo que o estado seja X" → Kubernetes garante que o estado atual sempre corresponda

**Benefícios**:
- Pode ser armazenado em Git (infrastructure as code)
- Ferramentas padrão de desenvolvimento (code review, versionamento) aplicáveis
- GitOps: Git é a fonte de verdade
- Rollback: simplesmente revert o commit

**Padrão**: Toda configuração em Kubernetes é declarativa (manifests YAML descrevem desired state)

#### **Sistemas Auto-Reparáveis (Self-Healing Systems)**

**Como funciona**: 
- Você declara "quero 3 replicas deste Pod"
- Kubernetes **continuamente** verifica se há 3 replicas rodando
- Se um Pod falhar, Kubernetes cria um novo automaticamente
- Se um nó inteiro falhar, Pods são reschedulados

**Vantagem operacional**: Reduz dependência de oncall/operadores experientes
- Problemas são reparados automaticamente
- Operadores se focam em SLA, não em reparos manuais
- A confiabilidade não depende apenas de uptime de um nó

#### **Desacoplamento (Decoupling)**

**Problema**: Quando equipes e serviços crescem, comunicação entre camadas vira gargalo

**Solução**: Separar responsabilidades via APIs
- **Application Ops/SRE ↔ Kubernetes API**: App declares desired state, K8s garante
- **Cluster Ops/SRE ↔ Kernel SyscCall API**: K8s declares needed resources, kernel fornece
- **Hardware Ops/SRE ↔ CPU/Hardware**: Kernel gerencia recursos físicos

**Resultado** (Figura 1-1 do livro):
- Cada camada tem contrato bem definido (SLA)
- Times podem escalar independentemente
- Pequeno time de platform engineers pode suportar centenas de aplicações

### Padrões para Escala

#### **Escalando Aplicações**
- Configuração declarativa torna trivial: "aumentar replicas de 3 para 10" = mudar um número
- Kubernetes distribui contêineres automaticamente
- Horizontal scaling (mais contêineres) mais eficiente que vertical (máquinas maiores)

#### **Escalando Equipes com Microserviços**
- **Two-pizza team**: 6-8 pessoas, ideal para: bom compartilhamento de conhecimento, decisões rápidas, propósito compartilhado
- Cada time possui um microserviço desacoplado
- Kubernetes abstrações para integração:
  - **Pods**: Grupo de contêineres (mesmo que desenvolvidos por times diferentes)
  - **Services**: Load balancing, DNS, service discovery
  - **Namespaces**: Isolamento e controle de acesso
  - **Ingress**: Frontend unificado para múltiplos microserviços

**Tensão resolvida**: Quer agilidade (pequenas equipes) mas precisa de escala (muitas máquinas)?
→ Kubernetes desacopla equipes de máquinas específicas

### Considerações Práticas

#### **Quando usar Kubernetes**
- ✅ Aplicações cloud-native que precisam de velocidade e escalabilidade
- ✅ Múltiplas equipes colaborando
- ✅ Ambientes heterogêneos (on-prem, multi-cloud)

#### **Quando considerar alternativas**
- ❌ Aplicação monolítica simples em um único servidor
- ❌ Organização sem experiência em containers/DevOps
- ❌ Requisitos de ultrabaxa latência (mesmo que raro, overhead de K8s importa)

#### **Managed vs Self-Managed**
- **Kubernetes-as-a-Service (KaaS)**: GKE, AKS, EKS
  - ✅ Reduz overhead operacional
  - ❌ Menos flexibilidade (algumas features podem estar desabilitadas)
  - Recomendado para organizações pequenas/médias
- **Self-managed**: Mais flexibilidade, mas exige expertise em cluster ops

### Nível de Profundidade

**Intermediário**: O capítulo introduz conceitos fundamentais com rigor arquitetural, mas sem detalhes operacionais. Prepara para os capítulos seguintes que mergulham em Pods, Services, Deployments, etc.

### Conexões com Outros Contextos

- **DDIA (Kleppmann)**: 
  - Imutabilidade relacionada a `Write-Once, Read-Many` storage patterns
  - Declaração de estado vs. eventual consistency
  - Self-healing systems parallela a tolerância a falhas em distributed systems
- **Observabilidade/Prometheus**: Kubernetes fornece hooks nativos para liveness/readiness probes (que alimentam métricas)
- **DevOps/GitOps**: Infrastructure-as-code em Git é o padrão de deploy em Kubernetes

---

**Resumo executivo**: Kubernetes resolve o problema de escalar aplicações, equipes e infraestrutura simultaneamente através de três princípios: (1) imutabilidade elimina estado desconhecido, (2) declaratividade torna reversibilidade e reproduzibilidade viáveis, (3) self-healing reduz carga operacional. O resultado é velocidade no desenvolvimento sem sacrificar confiabilidade.
