


Documentação Técnica – Arquitetura Híbrida XPTO

## 1. Visão Geral da Arquitetura
A arquitetura híbrida combina os recursos on-premises existentes com a escalabilidade da nuvem AWS. Essa estratégia preserva os investimentos legados enquanto incorpora automação, alta disponibilidade, segurança e controle de custos.
## 2. Componentes On-Premises
### 2.1 Proxy F5
- Função: Proxy Reverso, WAF
- Justificativa: Proteger o tráfego HTTP/S, inspecionar pacotes, prevenir ataques (ex: SQL Injection, DDOS)
- Camadas OSI: 3 (Redes), 4 (Firewall) e 7 (HTTP/S)
- Protocolo: TCP 443
- Sistema Operacional: TMOS (F5 OS)
### 2.2 Front-End (Apresentação)
- VMs: Xptofront1210 até Xptofront1213
- SO: Ubuntu 22.04 LTS
- vCPU: 2 | RAM: 4 GB | Disco: 50 GB SSD
- Função: Servidores Web (Nginx ou Apache) que fazem o frontend da aplicação legado
- Justificativa: Interface com usuários internos e externos com alta disponibilidade (4 instâncias)
### 2.3 Back-End (Aplicação)
- VMs: xptoback2223 até xptoback2226
- SO: Ubuntu 22.04 + Docker
- vCPU: 2 | RAM: 4 GB | Disco: 100 GB SSD
- Função: Lógica da aplicação principal (controle de lançamentos e consolidados)
- Justificativa: Mantém proximidade com o banco legado para baixa latência, separando o Back-end do Front-end ele fica seguro de acessos não autorizados, alta disponibilidade (4 instâncias)

### 2.4 Banco de Dados SQL Server
- VMs: xptodbsql33 até xptodbsql36
- SO: Windows Server 2022 | SQL Server 2019
- vCPU: 8 | RAM: 32 GB | Disco: 700 GB (RAID10)
- Função: Armazenamento de Dados
- Justificativa: Alta disponibilidade local, replicação entre instâncias, utilizado para sistemas legados, alta disponibilidade (4 instâncias)
### 2.5 ORY Hydra (Identidade)
- VM: xptoident44 / xptoident45
- SO: Ubuntu 22.04
- vCPU: 2 | RAM: 4 GB | Disco: 30 GB
- Função: OpenID Connect Server (Autenticação federada para serviços legados e cloud)
- Justificativa: Controle de identidade centralizado, SSO para aplicações internas
### 2.6 Dynatrace
- Função: APM e rastreamento de performance de ponta a ponta
- Justificativa: Observabilidade e métricas completas (camadas on-premises e cloud)
## 3. Componentes em Cloud AWS
### 3.1 VPCs e Subnets
- Redes hub e corporativa isoladas por subnets públicas e privadas
- Duas VPCs: xwsspto-xpto-sao-hubint (rede hub) e xwsspto-xpto-sao-corp (corporativo)
- Sub-redes privadas 
- Justificativa: Isolamento de tráfego, controle granular de ACLs e rotas
### 3.2 Transit Gateway
- Função: Ponto central para o tráfego de rede, conectando VPCs e redes locais por meio de um único gateway. 
- Justificativa: Roteamento centralizado, baixo tempo de resposta, escalável para múltiplas VPCs
### 3.3 API Gateway
- Função: Exposição de serviços REST da aplicação na AWS
- Justificativa: Escalável, seguro, com throttle e autenticação embutida
### 3.4 ECS (Fargate)
- Função: Containers controle de lançamentos e Consolidação diária
- vCPU (por container): 1 vCPU (controle), 0.5 vCPU (consolidação)
- RAM: 2 GB e 1 GB, respectivamente
Justificativa: Não requer gerenciamento de servidores diminuindo a sobrecarga operacional, auto-escalável, integração com CloudWatch

### 3.5 SQS
- Função: Serviço de fila de mensagens 
Justificativa: Permite que os componentes de um sistema se comuniquem sem dependências diretas, aumentando a flexibilidade e a escalabilidade.
### 3.6 Amazon RDS
Função: Banco de dados MySQL
- Instância: db.t3.medium | vCPU: 2 | RAM: 4 GB | Disco: 100 GB SSD
- Justificativa: Alta disponibilidade nativa, backups automáticos, compatibilidade com microsserviços
### 3.7 S3
- Função: Armazenamento de arquivos estáticos, logs 
Versionamento: Ciclo de vida (move para Glacier após 90 dias)
Criptografia SSE
Justificativa: Alta durabilidade, baixo custo, integração com RDS, ECS, backups
### 3.8 Segurança
- GuardDuty, AWS Config, CloudTrail, Shield, IAM: Compliance e detecção de ameaças, princípio de menor privilégio
- SGs e NACLs: Regras específicas por porta e subnet
- WAF + Shield: Proteção L7 e mitigação DDoS
### 3.9 Monitoramento
- CloudWatch: Logs, Metrics
### 4 Automação
- Terraform: Infraestrutura cloud - Modularização por ambientes, provisionamento de ECS, RDS, API Gateway, S3, etc.
- **Ansible**: Configuração on-prem
- **GitHub**: CI/CD e versionamento
## 5. Estratégia FinOps
- Auto-escalonamento no ECS e API Gateway
- S3 com ciclo de vida para otimização de armazenamento
- Uso de instâncias sob demanda e Reserved Instances no RDS
- CloudWatch Alarms para orçamentos
- Análise com Cost Explorer e relatórios periódicos
## 6. Disaster Recovery (DR)
- RDS Multi-AZ
- Backups diários automatizados
- S3 replicado entre regiões
- Infraestrutura reconstituível via Terraform
- RTO: 30 minutos | RPO: 5 minutos


## 7. Modelo OSI Aplicado
- Camada 7: WAF, ORY Hydra, API Gateway
- Camada 6: TLS (S3, RDS, API Gateway)
- Camada 4: SGs/NACLs
- Camada 3: VPC, rotas, TGW
- Camada 1–2: Direct Connect, switches físicos on-prem


## 8. Conclusão
- A arquitetura proposta garante uma transição segura e estratégica da XPTO para a nuvem, mantendo o legado com alta performance e preparando a empresa para escalabilidade futura, redução de custos, observabilidade avançada, e resiliência corporativa.
