# 🌐 Projeto WordPress em Alta Disponibilidade na AWS

Bem-vindo ao repositório do projeto **WordPress em Alta Disponibilidade na AWS**!  
Este projeto demonstra como implantar a plataforma **WordPress** na **Amazon Web Services (AWS)** de forma escalável e tolerante a falhas.

A arquitetura utiliza serviços gerenciados da AWS para garantir **desempenho**, **resiliência** e **alta disponibilidade**, simulando um ambiente de produção real.

---

## ☁️ ARQUITETURA DO PROJETO

A infraestrutura foi projetada para ser **resiliente** e **escalável**:

- **Auto Scaling Group (ASG)** com múltiplas instâncias EC2 para executar o WordPress.
- **Application Load Balancer (ALB)** para balancear o tráfego de entrada.
- **Amazon EFS** para armazenamento compartilhado de arquivos do WordPress (mídias, plugins, etc.).
- **Amazon RDS (MySQL/MariaDB)** para armazenar dados como posts e informações de usuários.

---

## 🛠️ COMPONENTES NA AWS

### 🔹 VPC Personalizada
- **Subnets**: 2 públicas e 4 privadas, distribuídas em duas Zonas de Disponibilidade (AZs).
- **Internet Gateway (IGW)**: Permite acesso à internet pelas subnets públicas.
- **NAT Gateway**: Localizado nas subnets públicas, permitindo que as instâncias privadas acessem a internet.

- ## 🔐 Security Groups

### sg-ALB (Application Load Balancer)
| Direção        | Tipo   | Protocolo | Porta | Origem/Destino   |
|----------------|--------|-----------|-------|------------------|
| Entrada        | HTTP   |    TCP    | 80    |    0.0.0.0/0     |
| Saída          | HTTP   |    TCP    | 80    |     sg-EC2       |

---

### sg-RDS (Banco de Dados MySQL/Aurora)
| Direção        | Tipo          | Protocolo | Porta | Origem/Destino |
|----------------|---------------|-----------|-------|----------------|
| Entrada        | MySQL/Aurora  |    TCP    | 3306  |    sg-EC2      |
| Saída          | MySQL/Aurora  |    TCP    | 3306  |    sg-EC2      |

---

### sg-EFS (Armazenamento EFS)
| Direção        | Tipo  | Protocolo | Porta | Origem/Destino   |
|----------------|-------|-----------|-------|------------------|
| Entrada        | NFS   |    TCP    | 2049  |    sg-EC2        |
| Saída          | NFS   |    TCP    | 2049  |    sg-EC2        |

---

### sg-EC2 
| Direção        |      Tipo      | Protocolo | Porta | Origem/Destino   |
|------------ ---|----------------|-----------|-------|------------------|
| Entrada        |      HTTP      |    TCP    |   80  |    sg-ALB        |
| Entrada        |      MySQL     |    TCP    |  3306 |    sg-RDS        |
| Saída          |       NFS      |    TCP    |   80  |    0.0.0.0/0     |
| Saída          |  Todo Trafego  |    TCP    |   80  |    0.0.0.0/0     |
| Saída          |      MySQL     |    TCP    |  3306 |    0.0.0.0/0     |
| Saída          |      HTTP      |    TCP    |   80  |    0.0.0.0/0     |


### 🔹 Armazenamento e Banco de Dados
- **Amazon EFS**: Sistema de arquivos compartilhado e centralizado.
- **Amazon RDS**: Banco de dados relacional gerenciado (MySQL/MariaDB).

---

## 🚀 ETAPAS DE IMPLEMENTAÇÃO

1. **Criar a VPC**
   - Definir subnets, IGW e NAT Gateway.

2. **Configurar o RDS**
   - Criar instância MySQL/MariaDB.
   - Configurar grupo de segurança para acesso apenas das instâncias EC2.

3. **Configurar o EFS**
   - Criar sistema de arquivos NFS.
   - Configurar permissões para acesso pelas instâncias EC2.

4. **Criar Launch Template**
   - Incluir script *user-data* para:
     - Instalar o WordPress.
     - Montar o EFS.
     - Conectar ao banco RDS.

5. **Criar Auto Scaling Group**
   - Usar o Launch Template.
   - Configurar nas subnets privadas.
   - Associar ao ALB.

6. **Configurar Application Load Balancer**
   - Associar às subnets públicas.
   - Redirecionar tráfego para o ASG.

---

## ⚠️ CUIDADOS PARA CONTAS DE ESTUDO

- Monitore custos no **Cost Explorer** diariamente.
- **Remova recursos** ao finalizar (incluindo VPCs).
- EC2:
  - Tipos compatíveis com *free tier*.
  - Adicionar *tags*: `Name`, `CostCenter`, `Project`.
- RDS:
  - Tipo: `db.t3g.micro`.
  - **Sem Multi-AZ**.

---

## 📜 LICENÇA
Este projeto é para fins educacionais e segue as boas práticas de implantação em nuvem.
