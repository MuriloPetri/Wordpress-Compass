## ☁️ ARQUITETURA DO PROJETO

A infraestrutura foi projetada para ser **resiliente** e **escalável**:

- **Auto Scaling Group (ASG)** com múltiplas instâncias EC2 para executar o WordPress.  
- **Application Load Balancer (ALB)** para balancear o tráfego de entrada.  
- **Amazon EFS** para armazenamento compartilhado de arquivos do WordPress (mídias, plugins, etc.).  
- **Amazon RDS (MySQL/MariaDB)** para armazenar dados como posts e informações de usuários.

---

## 🛠️ COMPONENTES NA AWS

### 🔹 VPC Personalizada
- **Subnets**: 2 públicas e 4 privadas, distribuídas em duas Zonas de Disponibilidade (AZs)  
- **Internet Gateway (IGW)**: Permite acesso à internet pelas subnets públicas  
- **NAT Gateway**: Localizado nas subnets públicas, permitindo que as instâncias privadas acessem a internet  

---

## 🚀 ETAPAS DE IMPLEMENTAÇÃO

### 1. Criar a VPC
- Criar 2 subnets públicas  
- Criar 4 subnets privadas  
- Criar IGW e NAT Gateway
<img width="1404" height="880" alt="Image" src="https://github.com/user-attachments/assets/14954835-7c4d-49b8-bc43-909385cf85ed" />

### 2. Criando Security Groups

#### sg-ALB (Application Load Balancer)
| Direção | Tipo | Protocolo | Porta | Origem/Destino |
|---------|------|-----------|-------|----------------|
| Entrada | HTTP | TCP       | 80    | 0.0.0.0/0      |
| Saída   | HTTP | TCP       | 80    | sg-EC2         |

#### sg-RDS (Banco de Dados MySQL/MariaDB)
| Direção | Tipo         | Protocolo | Porta | Origem/Destino |
|---------|--------------|-----------|-------|----------------|
| Entrada | MySQL/Aurora | TCP       | 3306  | sg-EC2         |
| Saída   | MySQL/Aurora | TCP       | 3306  | sg-EC2         |

#### sg-EFS (Armazenamento EFS)
| Direção | Tipo | Protocolo | Porta | Origem/Destino |
|---------|------|-----------|-------|----------------|
| Entrada | NFS  | TCP       | 2049  | sg-EC2         |
| Saída   | NFS  | TCP       | 2049  | sg-EC2         |

#### sg-EC2 (Instâncias EC2)
| Direção | Tipo        | Protocolo | Porta | Origem/Destino |
|---------|------------|-----------|-------|----------------|
| Entrada | HTTP       | TCP       | 80    | sg-ALB         |
| Entrada | MySQL      | TCP       | 3306  | sg-RDS         |
| Saída   | NFS        | TCP       | 2049  | 0.0.0.0/0      |
| Saída   | Todo Tráfego | TCP     | 80    | 0.0.0.0/0      |
| Saída   | MySQL      | TCP       | 3306  | 0.0.0.0/0      |
| Saída   | HTTP       | TCP       | 80    | 0.0.0.0/0      |

---

### 3. Configurar o RDS
- Criar instância MySQL ou MariaDB (qual preferir)  
- Selecionar opção de *Free-tier*  
- Tipo: `db.t3.micro`  
- Associar à VPC criada
- **IMPORTANTE ->** Em configuração adicional escolha o mesmo nome do **Identificador da instância de banco de dados** para evitar possíveis erros
<img width="1332" height="167" alt="Image" src="https://github.com/user-attachments/assets/fb725fae-a211-40c9-a8e3-0c288a526045" />
<img width="889" height="166" alt="Image" src="https://github.com/user-attachments/assets/3d0f4120-6df8-4315-95db-8e6547a6a7ef" />

### 4. Configurar o EFS
- Criar sistema de arquivos NFS
- Clicar em **PERSONALIZAR**
<img width="676" height="684" alt="Image" src="https://github.com/user-attachments/assets/d0b071e2-a5f6-42c6-97f0-483338c207a8" />
- Associar subnets privadas de cada zona
<img width="1559" height="342" alt="Image" src="https://github.com/user-attachments/assets/1851271f-2a8d-47b9-bc55-4d4ce96b04d4" />
- E criar a EFS

### 5. Criar Launch Template
- Sistema Operacional: Amazon Linux  
- Tipo: `t2.micro`  
- Selecionar VPC do projeto
- Não precisa incluir a sub-rede
- Incluir o Security Group da EC2  
- Incluir script *user-data* para:
  - Instalar WordPress  
  - Montar EFS  
  - Conectar ao RDS  

### 6. Criar Target Group (Grupo de Destino)
- Tipo de destino: **Instâncias**
- Incluir a VPC criada para o projeto
- Selecionar a subnets públicas  
- Caminho de verificação de integridade (Health Check Path): `/` ou `/wp-admin/images/wordpress-logo.svg`
<img width="1073" height="260" alt="Image" src="https://github.com/user-attachments/assets/9e5ff602-7b35-4d4f-a4cc-1d75943d2d28" />

### 7. Configurar Application Load Balancer
- Selecionar o tipo Application Load Balamcer
- Voltado para Internet
- Selecionar a VPC criada para o projeto
- Associar às **subnets públicas**  
- Direcionar tráfego para o **Target Group**  


### 8. Criar Auto Scaling Group
- Usar o Launch Template criado  
- Configurar nas subnets privadas  
- Associar ao ALB (Application Load Balancer)  

### 9. Resultados Finais
- Acesse a página **Load Balancers** na seção EC2  
- Copie o DNS e abra no navegador  
- Em instantes, aparecerá a tela do WordPress  

### 10. WordPress
<img width="1917" height="918" alt="Image" src="https://github.com/user-attachments/assets/3179a93c-a83b-46e7-820d-b855e306d63d" />
