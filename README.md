# Amazon Web Services (AWS) İnfrastrukturunun Arxitekturası

## Giriş

Amazon Web Services (AWS) dünyanın ən böyük və ən geniş istifadə olunan bulud platformasıdır. Bu sənəd AWS infrastrukturunun əsas arxitektura komponentlərini və onların necə işlədiyini izah edir.

## AWS-in Əsas Arxitektura Komponentləri

### 1. Regionlar (Regions)

AWS infrastrukturu dünya üzrə **regionlara** bölünmüşdür:

- **Region nədir?** - Müstəqil coğrafi ərazidə yerləşən data mərkəzlərinin toplusu
- **Xüsusiyyətləri:**
  - Hər region digər regionlardan tam izolə edilmişdir
  - Yüksək mövcudluq və davamlılıq təmin edir
  - Məlumatlar regionlar arasında avtomatik olaraq köçürülmür
  - Hər regionda minimum 3 Availability Zone var

```mermaid
graph TB
    subgraph "AWS Global Infrastructure"
        subgraph "US-East-1 (Virginia)"
            AZ1A[AZ-1a<br/>Data Center 1]
            AZ1B[AZ-1b<br/>Data Center 2]
            AZ1C[AZ-1c<br/>Data Center 3]
        end
        
        subgraph "EU-West-1 (Ireland)"
            AZ2A[AZ-1a<br/>Data Center 1]
            AZ2B[AZ-1b<br/>Data Center 2]
            AZ2C[AZ-1c<br/>Data Center 3]
        end
        
        subgraph "AP-Southeast-1 (Singapore)"
            AZ3A[AZ-1a<br/>Data Center 1]
            AZ3B[AZ-1b<br/>Data Center 2]
            AZ3C[AZ-1c<br/>Data Center 3]
        end
    end
    
    AZ1A -.->|High Speed Network| AZ1B
    AZ1B -.->|High Speed Network| AZ1C
    AZ1C -.->|High Speed Network| AZ1A
    
    AZ2A -.->|High Speed Network| AZ2B
    AZ2B -.->|High Speed Network| AZ2C
    AZ2C -.->|High Speed Network| AZ2A
    
    AZ3A -.->|High Speed Network| AZ3B
    AZ3B -.->|High Speed Network| AZ3C
    AZ3C -.->|High Speed Network| AZ3A
```

### 2. Mövcudluq Zonaları (Availability Zones - AZ)

- **AZ nədir?** - Bir region daxilində ayrı-ayrı data mərkəzləri
- **Xüsusiyyətləri:**
  - Hər AZ fiziki olaraq ayrıdır
  - Yüksək sürətli şəbəkə ilə bağlıdır
  - Təbii fəlakətlərə qarşı müqavimət
  - Redundancy və fault tolerance təmin edir

### 3. Edge Locations və CloudFront

- **Edge Location:** İstifadəçilərə yaxın məzmun çatdırma nöqtələri
- **CloudFront:** AWS-in Content Delivery Network (CDN) xidməti
- **Faydaları:**
  - Aşağı latency
  - Yüksək transfer sürəti
  - Global məzmun paylanması

## Əsas AWS Xidmətləri və Arxitektura

### 1. Hesablama Xidmətləri (Compute Services)

#### Amazon EC2 (Elastic Compute Cloud)
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Web Server    │    │  App Server     │    │   Database      │
│     (EC2)       │────│     (EC2)       │────│     (RDS)       │
│                 │    │                 │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

```mermaid
graph LR
    subgraph "Auto Scaling Group"
        EC2_1[🖥️ EC2 Instance 1<br/>Web Server]
        EC2_2[🖥️ EC2 Instance 2<br/>Web Server]
        EC2_3[🖥️ EC2 Instance 3<br/>Web Server]
    end
    
    subgraph "Load Balancer"
        ALB[⚖️ Application<br/>Load Balancer]
    end
    
    subgraph "Database Layer"
        RDS[(🗄️ RDS Database<br/>Multi-AZ)]
    end
    
    Users[👥 Users] --> ALB
    ALB --> EC2_1
    ALB --> EC2_2
    ALB --> EC2_3
    
    EC2_1 --> RDS
    EC2_2 --> RDS
    EC2_3 --> RDS
    
    CloudWatch[📊 CloudWatch] -.-> EC2_1
    CloudWatch -.-> EC2_2
    CloudWatch -.-> EC2_3
    
    style EC2_1 fill:#ff9900
    style EC2_2 fill:#ff9900
    style EC2_3 fill:#ff9900
    style RDS fill:#0066cc
```

- Virtual maşınlar (instances)
- Müxtəlif instance növləri
- Auto Scaling imkanları
- Load Balancing

#### AWS Lambda
```mermaid
graph TB
    subgraph "Event Sources"
        S3[📁 S3 Bucket]
        API[🌐 API Gateway]
        CloudWatch[⏰ CloudWatch Events]
        DynamoDB[🗄️ DynamoDB Streams]
    end
    
    subgraph "Lambda Functions"
        Lambda1[⚡ Lambda Function 1<br/>Image Processing]
        Lambda2[⚡ Lambda Function 2<br/>API Handler]
        Lambda3[⚡ Lambda Function 3<br/>Data Processing]
    end
    
    subgraph "Destinations"
        S3_Out[📁 S3 Output]
        DDB[🗄️ DynamoDB]
        SNS[📧 SNS]
    end
    
    S3 -->|Object Created| Lambda1
    API -->|HTTP Request| Lambda2
    CloudWatch -->|Scheduled Event| Lambda3
    DynamoDB -->|Stream Record| Lambda1
    
    Lambda1 --> S3_Out
    Lambda2 --> DDB
    Lambda3 --> SNS
    
    style Lambda1 fill:#ff9900
    style Lambda2 fill:#ff9900
    style Lambda3 fill:#ff9900
```

- Serverless hesablama
- Event-driven arxitektura
- Avtomatik miqyaslama
- Pay-per-use model

### 2. Yaddaş Xidmətləri (Storage Services)

#### Amazon S3 (Simple Storage Service)
```
┌─────────────────────────────────────────────────────────────┐
│                        S3 Bucket                           │
├─────────────────┬─────────────────┬─────────────────────────┤
│   Standard      │   Infrequent    │      Glacier            │
│   Storage       │   Access        │   (Archive)             │
└─────────────────┴─────────────────┴─────────────────────────┘
```

```mermaid
graph TB
    subgraph "S3 Storage Classes"
        Standard[📦 S3 Standard<br/>Frequent Access<br/>99.999999999% Durability]
        IA[📦 S3 Infrequent Access<br/>Less Frequent Access<br/>Lower Cost]
        Glacier[🧊 S3 Glacier<br/>Archive Storage<br/>Very Low Cost]
        DeepArchive[❄️ S3 Glacier Deep Archive<br/>Long-term Archive<br/>Lowest Cost]
    end
    
    subgraph "Lifecycle Management"
        Upload[📤 Upload Objects]
        Transition1[⏰ 30 Days]
        Transition2[⏰ 90 Days]
        Transition3[⏰ 365 Days]
    end
    
    subgraph "Access Patterns"
        WebApp[🌐 Web Applications]
        Backup[💾 Backup & Archive]
        Analytics[📊 Big Data Analytics]
        CDN[🌍 CloudFront CDN]
    end
    
    Upload --> Standard
    Standard -->|After 30 days| IA
    IA -->|After 90 days| Glacier
    Glacier -->|After 365 days| DeepArchive
    
    WebApp --> Standard
    Backup --> Glacier
    Analytics --> Standard
    CDN --> Standard
    
    Standard -.->|Lifecycle Rule| Transition1
    Transition1 -.-> Transition2
    Transition2 -.-> Transition3
    
    style Standard fill:#ff9900
    style IA fill:#ff6600
    style Glacier fill:#0066cc
    style DeepArchive fill:#003366
```

- Object storage
- 99.999999999% (11 9's) davamlılıq
- Müxtəlif storage class-ları
- Versioning və lifecycle management

#### Amazon EBS (Elastic Block Store)
- EC2 üçün persistent storage
- Snapshot imkanları
- Müxtəlif performans səviyyələri

### 3. Verilənlər Bazası Xidmətləri (Database Services)

#### Amazon RDS (Relational Database Service)
- Managed relational databases
- Multi-AZ deployment
- Read replicas
- Automated backups

#### Amazon DynamoDB
- NoSQL database
- Serverless
- Global tables
- Auto scaling

### 4. Şəbəkə və Məzmun Çatdırma

#### Amazon VPC (Virtual Private Cloud)
```
┌─────────────────────────────────────────────────────────────┐
│                        VPC                                  │
│  ┌─────────────────┐              ┌─────────────────┐      │
│  │  Public Subnet  │              │ Private Subnet  │      │
│  │                 │              │                 │      │
│  │  ┌───────────┐  │              │  ┌───────────┐  │      │
│  │  │    EC2    │  │              │  │    RDS    │  │      │
│  │  └───────────┘  │              │  └───────────┘  │      │
│  └─────────────────┘              └─────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

```mermaid
graph TB
    subgraph "Internet"
        Internet[🌐 Internet]
    end
    
    subgraph "VPC (10.0.0.0/16)"
        subgraph "Availability Zone A"
            subgraph "Public Subnet A (10.0.1.0/24)"
                WebServer1[🖥️ Web Server<br/>EC2 Instance]
                NAT1[🔄 NAT Gateway]
            end
            
            subgraph "Private Subnet A (10.0.3.0/24)"
                AppServer1[⚙️ App Server<br/>EC2 Instance]
                DB1[(🗄️ RDS Primary<br/>Database)]
            end
        end
        
        subgraph "Availability Zone B"
            subgraph "Public Subnet B (10.0.2.0/24)"
                WebServer2[🖥️ Web Server<br/>EC2 Instance]
                NAT2[🔄 NAT Gateway]
            end
            
            subgraph "Private Subnet B (10.0.4.0/24)"
                AppServer2[⚙️ App Server<br/>EC2 Instance]
                DB2[(🗄️ RDS Standby<br/>Database)]
            end
        end
        
        IGW[🚪 Internet Gateway]
        ALB[⚖️ Application Load Balancer]
        
        subgraph "Security"
            SG_Web[🛡️ Web Security Group<br/>Port 80, 443]
            SG_App[🛡️ App Security Group<br/>Port 8080]
            SG_DB[🛡️ DB Security Group<br/>Port 3306]
        end
    end
    
    Internet --> IGW
    IGW --> ALB
    ALB --> WebServer1
    ALB --> WebServer2
    
    WebServer1 --> AppServer1
    WebServer2 --> AppServer2
    
    AppServer1 --> DB1
    AppServer2 --> DB1
    DB1 -.->|Replication| DB2
    
    AppServer1 --> NAT1
    AppServer2 --> NAT2
    NAT1 --> IGW
    NAT2 --> IGW
    
    SG_Web -.-> WebServer1
    SG_Web -.-> WebServer2
    SG_App -.-> AppServer1
    SG_App -.-> AppServer2
    SG_DB -.-> DB1
    SG_DB -.-> DB2
    
    style WebServer1 fill:#ff9900
    style WebServer2 fill:#ff9900
    style AppServer1 fill:#66cc00
    style AppServer2 fill:#66cc00
    style DB1 fill:#0066cc
    style DB2 fill:#0066cc
```

### 2. Microservices Arxitekturası
```
┌─────────────────┐    ┌─────────────────┐
│   API Gateway   │────│   Lambda        │
└─────────┬───────┘    └─────────────────┘
          │
┌─────────▼───────┐    ┌─────────────────┐
│   ECS/EKS       │────│   DynamoDB      │
│  (Containers)   │    │                 │
└─────────────────┘    └─────────────────┘
```

```mermaid
graph TB
    subgraph "Client Layer"
        WebApp[🌐 Web Application]
        MobileApp[📱 Mobile App]
        ThirdParty[🔗 Third Party APIs]
    end
    
    subgraph "API Gateway Layer"
        APIGateway[🚪 API Gateway<br/>Authentication<br/>Rate Limiting<br/>Request Routing]
    end
    
    subgraph "Microservices"
        subgraph "User Service"
            UserLambda[⚡ User Lambda<br/>Serverless]
            UserDB[(🗄️ User DynamoDB)]
        end
        
        subgraph "Product Service"
            ProductECS[🐳 Product Service<br/>ECS Container]
            ProductDB[(🗄️ Product RDS)]
        end
        
        subgraph "Order Service"
            OrderEKS[☸️ Order Service<br/>EKS Kubernetes]
            OrderDB[(🗄️ Order DynamoDB)]
        end
        
        subgraph "Payment Service"
            PaymentLambda[⚡ Payment Lambda<br/>Serverless]
            PaymentDB[(🗄️ Payment RDS)]
        end
        
        subgraph "Notification Service"
            NotificationECS[🐳 Notification Service<br/>ECS Container]
            SNS[📧 SNS Topics]
            SQS[📬 SQS Queues]
        end
    end
    
    subgraph "Data & Analytics"
        S3Analytics[📊 S3 Data Lake]
        Kinesis[🌊 Kinesis Streams]
        ElasticSearch[🔍 ElasticSearch]
    end
    
    subgraph "Monitoring & Logging"
        CloudWatch[📈 CloudWatch]
        XRay[🔍 X-Ray Tracing]
    end
    
    WebApp --> APIGateway
    MobileApp --> APIGateway
    ThirdParty --> APIGateway
    
    APIGateway --> UserLambda
    APIGateway --> ProductECS
    APIGateway --> OrderEKS
    APIGateway --> PaymentLambda
    
    UserLambda --> UserDB
    ProductECS --> ProductDB
    OrderEKS --> OrderDB
    PaymentLambda --> PaymentDB
    
    OrderEKS --> PaymentLambda
    PaymentLambda --> NotificationECS
    
    NotificationECS --> SNS
    NotificationECS --> SQS
    
    UserLambda --> Kinesis
    ProductECS --> Kinesis
    OrderEKS --> Kinesis
    
    Kinesis --> S3Analytics
    Kinesis --> ElasticSearch
    
    CloudWatch -.-> UserLambda
    CloudWatch -.-> ProductECS
    CloudWatch -.-> OrderEKS
    CloudWatch -.-> PaymentLambda
    
    XRay -.-> APIGateway
    XRay -.-> UserLambda
    XRay -.-> ProductECS
    XRay -.-> OrderEKS
    
    style UserLambda fill:#ff9900
    style PaymentLambda fill:#ff9900
    style ProductECS fill:#0066cc
    style NotificationECS fill:#0066cc
    style OrderEKS fill:#66cc00
    style APIGateway fill:#cc6600
```

## Disaster Recovery Strategiyaları

```mermaid
graph TB
    subgraph "Primary Region (us-east-1)"
        PrimaryVPC[🏢 Primary VPC]
        PrimaryEC2[🖥️ Production EC2]
        PrimaryRDS[(🗄️ Primary RDS)]
        PrimaryS3[📁 Primary S3]
    end
    
    subgraph "Secondary Region (us-west-2)"
        SecondaryVPC[🏢 Secondary VPC]
        SecondaryEC2[🖥️ Standby EC2]
        SecondaryRDS[(🗄️ Secondary RDS)]
        SecondaryS3[📁 Secondary S3]
    end
    
    subgraph "DR Strategies"
        BackupRestore[💾 Backup & Restore<br/>RTO: Hours<br/>RPO: Hours<br/>💰 Lowest Cost]
        PilotLight[💡 Pilot Light<br/>RTO: 10-30 min<br/>RPO: Minutes<br/>💰💰 Medium Cost]
        WarmStandby[🔥 Warm Standby<br/>RTO: Minutes<br/>RPO: Seconds<br/>💰💰💰 High Cost]
        MultiSite[🌍 Multi-Site Active/Active<br/>RTO: Real-time<br/>RPO: Real-time<br/>💰💰💰💰 Highest Cost]
    end
    
    PrimaryRDS -.->|Cross-Region Replication| SecondaryRDS
    PrimaryS3 -.->|Cross-Region Replication| SecondaryS3
    
    PrimaryVPC -.->|VPC Peering| SecondaryVPC
    
    BackupRestore -.->|Scheduled Backups| SecondaryS3
    PilotLight -.->|Minimal Resources| SecondaryEC2
    WarmStandby -.->|Scaled Down| SecondaryEC2
    MultiSite -.->|Full Capacity| SecondaryEC2
    
    style BackupRestore fill:#66cc00
    style PilotLight fill:#ffcc00
    style WarmStandby fill:#ff9900
    style MultiSite fill:#ff6600
```

## AWS Well-Architected Framework

```mermaid
mindmap
  root((AWS Well-Architected Framework))
    Operational Excellence
      Monitoring & Logging
        CloudWatch
        CloudTrail
        X-Ray
      Automation
        Infrastructure as Code
        CI/CD Pipelines
        Auto Scaling
      Continuous Improvement
        Performance Reviews
        Lessons Learned
        Best Practices
    
    Security
      Identity & Access Management
        IAM Users/Roles
        Multi-Factor Auth
        Least Privilege
      Data Protection
        Encryption at Rest
        Encryption in Transit
        Key Management
      Network Security
        Security Groups
        NACLs
        WAF & Shield
    
    Reliability
      Multi-AZ Deployment
        Availability Zones
        Load Balancing
        Auto Scaling
      Backup & Recovery
        Automated Backups
        Cross-Region Replication
        Disaster Recovery
      Fault Tolerance
        Circuit Breakers
        Retry Logic
        Graceful Degradation
    
    Performance Efficiency
      Right-sizing Resources
        Instance Types
        Storage Classes
        Database Sizing
      Caching Strategies
        ElastiCache
        CloudFront
        Application Caching
      Content Delivery
        CDN
        Edge Locations
        Global Acceleration
    
    Cost Optimization
      Reserved Instances
        1-year/3-year Terms
        Payment Options
        Instance Families
      Spot Instances
        Batch Processing
        Fault-tolerant Apps
        Cost Savings
      Resource Monitoring
        Cost Explorer
        Budgets & Alerts
        Rightsizing Recommendations
```

### 5 Əsas Prinsip:

1. **Operational Excellence (Əməliyyat Mükəmməlliyi)**
   - Monitoring və logging
   - Automation
   - Continuous improvement

2. **Security (Təhlükəsizlik)**
   - Identity and Access Management (IAM)
   - Data encryption
   - Network security

3. **Reliability (Etibarlılık)**
   - Multi-AZ deployment
   - Backup və disaster recovery
   - Auto scaling

4. **Performance Efficiency (Performans Səmərəliliyi)**
   - Right-sizing resources
   - Caching strategies
   - Content delivery optimization

5. **Cost Optimization (Xərc Optimallaşdırması)**
   - Reserved instances
   - Spot instances
   - Resource monitoring

## Təhlükəsizlik Arxitekturası

### Identity and Access Management (IAM)
- Users, Groups, Roles
- Policies və permissions
- Multi-Factor Authentication (MFA)
- Cross-account access

### Şəbəkə Təhlükəsizliyi
- Security Groups (Stateful firewall)
- Network ACLs (Stateless firewall)
- AWS WAF (Web Application Firewall)
- AWS Shield (DDoS protection)

### Data Encryption
- Encryption at rest
- Encryption in transit
- AWS KMS (Key Management Service)
- CloudHSM

## Monitoring və Logging

### Amazon CloudWatch
- Metrics və alarms
- Log aggregation
- Dashboards
- Auto scaling triggers

### AWS CloudTrail
- API call logging
- Compliance və audit
- Security analysis

### AWS X-Ray
- Distributed tracing
- Performance analysis
- Debugging

### 1. Backup və Restore
- RTO: Saatlar
- RPO: Dəqiqələr/saatlar
- Ən aşağı xərc

### 2. Pilot Light
- RTO: 10-30 dəqiqə
- RPO: Dəqiqələr
- Orta xərc

### 3. Warm Standby
- RTO: Dəqiqələr
- RPO: Saniyələr
- Yüksək xərc

### 4. Multi-Site Active/Active
- RTO: Real-time
- RPO: Real-time
- Ən yüksək xərc

## Best Practices

### 1. Design Principles
- **Automate everything** - Mümkün olan hər şeyi avtomatlaşdırın
- **Treat infrastructure as code** - Infrastructure as Code (IaC) istifadə edin
- **Design for failure** - Uğursuzluq üçün dizayn edin
- **Implement security at every layer** - Hər səviyyədə təhlükəsizlik

### 2. Cost Optimization
- Reserved Instances istifadə edin
- Spot Instances-dan faydalanın
- Right-sizing edin
- Unused resources-ları silin
- CloudWatch ilə monitoring edin

### 3. Performance
- Caching strategiyalarından istifadə edin
- CDN (CloudFront) istifadə edin
- Database indexing
- Auto Scaling konfiqurasiya edin

## Nəticə

AWS infrastrukturu yüksək mövcudluq, miqyaslanabilirlik və təhlükəsizlik təmin edən mürəkkəb bir sistemdir. Düzgün arxitektura dizaynı ilə:

- **Yüksək performans** əldə edə bilərsiniz
- **Xərcləri optimize** edə bilərsiniz  
- **Təhlükəsizliyi** təmin edə bilərsiniz
- **Davamlılığı** artıra bilərsiniz

Bu sənəd AWS infrastrukturunun əsas komponentlərini əhatə edir. Daha ətraflı məlumat üçün AWS rəsmi sənədlərinə müraciət edin.

---

**Qeyd:** Bu sənəd AWS infrastrukturunun ümumi baxışını təqdim edir. Konkret layihələr üçün daha ətraflı arxitektura dizaynı tələb oluna bilər.
