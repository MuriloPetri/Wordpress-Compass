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


## 🚀 ETAPAS DE IMPLEMENTAÇÃO

1. **Criar a VPC**
   - Criar 2 subnets publicas
   - Criar 4 subnets privadas
   - IGW e NAT Gateway.

2. **Criando Security Groups**
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

### sg-EC2 (Instâncias EC2)
| Direção        | Tipo          | Protocolo | Porta | Origem/Destino |
|----------------|---------------|-----------|-------|----------------|
| Entrada        | HTTP          | TCP       | 80    | sg-ALB         |
| Entrada        | MySQL         | TCP       | 3306  | sg-RDS         |
| Saída          | NFS           | TCP       | 2049  | 0.0.0.0/0      |
| Saída          | Todo Tráfego  | TCP       | 80    | 0.0.0.0/0      |
| Saída          | MySQL         | TCP       | 3306  | 0.0.0.0/0      |
| Saída          | HTTP          | TCP       | 80    | 0.0.0.0/0      |



3. **Configurar o RDS**
   - Criar instância MySQL/MariaDB. (Qual voce preferir)
   - Selecionar opção de *Free-tier*
   - Tipo: db.t3.micro
   - Associe a VPC que criou para o projeto

4. **Configurar o EFS**
   - Criar sistema de arquivos NFS.
   - Personalize: Usando as subnets privadas de cada zona

5. **Criar Launch Template**
   - Sistema Operacional: Amazon Linux
   - Tipo: t2.micro
   - Selecione a VPC do projeto
   - Incluir script *user-data* para:
     - Instalar o WordPress.
     - Montar o EFS.
     - Conectar ao banco RDS.

6. **Criando o Target Group (Grupo de Destino)**
   -Tipo de destino: **Instâncias**
   -Caminho de verificação de integridade (Health Check Path): **/ ou /wp-admin/images/wordpress-logo.svg**

7. **Configurar Application Load Balancer**
   - Associar às **subnets públicas**.
   - Direcionar para o **Target Groupt**.
  
8. **Criar Auto Scaling Group**
   - Usar o Launch Template criado.
   - Configurar nas subnets privadas.
   - Associar ao ALB (Aplication Load Balancer).
  
9. **Resultados Finais**
   -Acesse a pagina **Load Balancers** na seção EC2
   - Copie o Nome DNS que irá aparecer
   - Pesquise o que colou, em alguns instantes irá aparecer a tela do Wordpress

10. **Wordpres**
    ![Wordpress](C:\Users\Muliro\Downloads/image)

     

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
