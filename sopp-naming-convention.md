# **Naming Convention SOPP \- Sistema Operativo Potencia Pragma**

# **Objetivo**

Establecer una estrategia estandarizada de nomenclatura para recursos de infraestructura cloud en SOPP (Sistema Operativo Potencia Pragma). Esta convención es agnóstica a proveedores (AWS, Azure, GCP) y garantiza consistencia, claridad y facilita la gestión, búsqueda y automatización de recursos.

# **Principios Fundamentales**

1. **Consistencia:** Mismo patrón en todos los proveedores y tipos de recursos  
2. **Claridad:** Nombres descriptivos que comunican el propósito del recurso  
3. **Escalabilidad:** Soporta crecimiento sin conflictos de nombres  
4. **Automatización:** Facilita scripting, búsqueda y filtrado  
5. **Legibilidad**: Fácil de leer y pronunciar  
6. **Brevedad:** Conciso sin sacrificar claridad  
7. **Agnóstico:** Funciona en AWS, Azure, GCP y otros proveedores

# **Estructura General de Nombres**

```
{client}-{project}-{environment}-{component}-{functionality}
```

### **Componentes**

| Componente | Descripción | Formato | Ejemplo |
| ----- | ----- | ----- | ----- |
| client | Nombre del cliente | {nombre-cliente} | pragma |
| project | Proyecto o iniciativa | {nombre-corto} | sopp |
| environment | Ambiente de despliegue | dev, qa, pre-pdn, pdn | pdn |
| component | Componente, Recurso | {nombre-corto} | **Componente:** api, db, cache, queue**Recurso:** ec2, rds, apigw |
| functionality | Funcionalidad específica del recurso | {nombre-corto} | billing, payments, auth |

## **Convenciones de Nomenclatura**

### **Reglas Generales**

1. ### Longitud: Máximo 63 caracteres (compatible con DNS)

2. ## Caracteres permitidos: Letras minúsculas (a-z), números (0-9), guiones (-)

3. ## Caracteres NO permitidos: Mayúsculas, puntos, guiones bajos, espacios, caracteres especiales

4. ## Inicio y fin: Debe comenzar y terminar con letra o número (no guión)

5. ## Guiones: Usar como separadores, máximo 1 guión consecutivo

6. ## Consistencia: Mismo recurso en diferentes ambientes debe tener mismo nombre base

### **Ejemplos de Nombres Válidos**

* ✅ pragma-sopp-pdn-api-billing   
* ✅ pragma-sopp-pdn-web-frontend   
* ✅ pragma-sopp-dev-s3-backup   
* ✅ pragma-sopp-pdn-vpc-shared  
* ✅ pragma-sopp-dev-rds-postgres  
* ✅ pragma-platform-pdn-role-api

### **Ejemplos de Nombres Inválidos**

* ❌ PRAGMA-sopp-pdn-api-billing (mayúsculas en client)   
* ❌ pragma\_sopp\_pdn\_web\_billing (guiones bajos)   
* ❌ pragma.sopp.pdn.api.billing (puntos)   
* ❌ \-pragma-sopp-pdn-api-billing (comienza con guión)   
* ❌ pragma-sopp-pdn-api-billing- (termina con guión)

## **Restricciones por Proveedor**

### **AWS**

**Restricciones Generales:**

* **Límite de caracteres:** Varía por servicio (típicamente 63-140 caracteres)  
* **Caracteres permitido**s: a-z, 0-9, guiones (-)  
* **Caracteres NO permitidos:** Mayúsculas, puntos, guiones bajos, espacios  
* **Inicio/Fin:** Debe comenzar y terminar con letra o número  
* **Caso:** Case-sensitive en algunos servicios

**Restricciones por Servicio:**

| Servicio | Límite | Caracteres | Notas |
| ----- | ----- | ----- | ----- |
| EC2 Instances | 255 | a-z, 0-9, guiones | Puede incluir puntos |
| RDS Instances | 63 | a-z, 0-9, guiones | Debe comenzar con letra |
| S3 Buckets | 63 | a-z, 0-9, guiones, puntos | Globalmente único, sin mayúsculas |
| Lambda Functions | 64 | a-z, 0-9, guiones | Debe comenzar con letra |
| VPC | 255 | a-z, 0-9, guiones, espacios | Puede incluir puntos |
| Security Groups | 255 | a-z, 0-9, guiones, espacios | Puede incluir puntos |
| IAM Roles | 64 | a-z, 0-9, guiones, puntos | Debe comenzar con letra |
| Step Functions | 80 | a-z, 0-9, guiones | Máximo 80 caracteres |

**Recomendación para AWS:** Usar máximo 63 caracteres para compatibilidad universal.

# 

# **Tabla de Servicios AWS y Abreviaturas para Componentes**

## **Servicios AWS \- Abreviaturas Estándar** 

| Abreviatura | Servicio | Descripción | Tipo | Ejemplo de Uso |
| ----- | ----- | ----- | ----- | ----- |
| alb-ext | Elastic Load Balancer External | Load Balancer | Networking | pragma-sopp-pdn-alb-ext-api |
| alb-int | Elastic Load Balancer Internal | Load Balancer | Networking | pragma-sopp-pdn-alb-int-app |
| apigw | API Gateway | API Gateway | Networking | pragma-sopp-pdn-apigw-billing |
| apigw-cd | API Gateway Custom Domain | API Gateway Custom Domain | Networking | pragma-sopp-pdn-apigw-cd-main |
| asg | Auto Scaling Group | Auto Scaling | Compute | pragma-sopp-pdn-asg-workers |
| athena | Athena | Query Engine | Analytics | pragma-sopp-pdn-athena-queries |
| cf | CloudFront | CDN | Networking | pragma-sopp-pdn-cf-distribution |
| cloudtrail | CloudTrail | Audit Logging | Monitoring | pragma-sopp-pdn-cloudtrail-logs |
| code-pipeline | CodePipeline | CI/CD Pipeline | CI/CD | pragma-sopp-pdn-code-pipeline-main |
| codebuild | CodeBuild | Build Service | CI/CD | pragma-sopp-pdn-codebuild-app |
| codecommit | CodeCommit Repository | Source Control | CI/CD | pragma-sopp-pdn-codecommit-repo |
| codedeploy | CodeDeploy | Deployment Service | CI/CD | pragma-sopp-pdn-codedeploy-app |
| cognito | Cognito | Identity Management | Security | pragma-sopp-pdn-cognito-auth |
| cw-alarm | CloudWatch Alarm | Monitoring Alert | Monitoring | pragma-sopp-pdn-cw-alarm-cpu |
| cw-canaries | CloudWatch Synthetics Canaries | Synthetic Monitoring | Monitoring | pragma-sopp-pdn-cw-canaries-health |
| cw-dashboard | CloudWatch Dashboard | Monitoring Dashboard | Monitoring | pragma-sopp-pdn-cw-dashboard-perf |
| cw-log-group | CloudWatch Log Group | Log Storage | Monitoring | pragma-sopp-pdn-cw-log-group-api |
| dynamodb | DynamoDB | NoSQL Database | Database | pragma-sopp-pdn-dynamodb-sessions |
| ec2 | EC2 | Virtual Machine | Compute | pragma-sopp-pdn-ec2-web |
| ecr | ECR Repository | Container Registry | Compute | pragma-sopp-pdn-ecr-images |
| ecs-cluster | ECS Cluster | Container Orchestration | Compute | pragma-sopp-pdn-ecs-cluster-main |
| ecs-service | ECS Service | Container Service | Compute | pragma-sopp-pdn-ecs-service-api |
| ecs-td | ECS Task Definition | Container Task | Compute | pragma-sopp-pdn-ecs-td-worker |
| efs | EFS | File Storage | Storage | pragma-sopp-pdn-efs-shared |
| eks | EKS | Kubernetes | Compute | pragma-sopp-pdn-eks-cluster |
| elasticache | ElastiCache | Cache Service | Caching | pragma-sopp-pdn-elasticache-redis |
| emr-cluster | EMR Cluster | Big Data | Analytics | pragma-sopp-pdn-emr-cluster-jobs |
| event-bridge-rule | Event Bridge Rule | Event Routing | Integration | pragma-sopp-pdn-event-bridge-rule-trigger |
| fsx | FSx | File System | Storage | pragma-sopp-pdn-fsx-windows |
| glb | Gateway Load Balancer | Load Balancer | Networking | pragma-sopp-pdn-glb-appliance |
| glue-connection | Glue Connection | Data Integration | Integration | pragma-sopp-pdn-glue-connection-db |
| glue-job | Glue Job | ETL Job | Integration | pragma-sopp-pdn-glue-job-etl |
| igw | Internet Gateway | Network Gateway | Networking | pragma-sopp-pdn-igw-main |
| ip-sets | IP Sets WAF | WAF IP List | Security | pragma-sopp-pdn-ip-sets-blocked |
| kms | KMS | Key Management | Security | pragma-sopp-pdn-kms-key |
| lambda | Lambda Function | Serverless Function | Compute | pragma-sopp-pdn-lambda-processor |
| lc | Launch Configuration | EC2 Launch Config | Compute | pragma-sopp-pdn-lc-web |
| memcache | MEMCached | Cache Service | Caching | pragma-sopp-pdn-memcache-sessions |
| ngw | NAT Gateway | Network Gateway | Networking | pragma-sopp-pdn-ngw-outbound |
| nlb | Network Load Balancer | Load Balancer | Networking | pragma-sopp-pdn-nlb-tcp |
| parameter-store | Parameter Store | Configuration | Configuration | pragma-sopp-pdn-parameter-store-config |
| quicksight | QuickSight | BI Tool | Analytics | pragma-sopp-pdn-quicksight-dashboard |
| rds | RDS | Relational Database | Database | pragma-sopp-pdn-rds-postgres |
| rds-cluster | RDS Aurora Cluster | Database Cluster | Database | pragma-sopp-pdn-rds-cluster-aurora |
| rds-instance | RDS Aurora Instance | Database Instance | Database | pragma-sopp-pdn-rds-instance-primary |
| rds-proxy | RDS Aurora Proxy | Database Proxy | Database | pragma-sopp-pdn-rds-proxy-app |
| rds-serverless | RDS Aurora Serverless | Serverless Database | Database | pragma-sopp-pdn-rds-serverless-app |
| redshift | Redshift | Data Warehouse | Analytics | pragma-sopp-pdn-redshift-warehouse |
| rtb | Route Tables | Network Routing | Networking | pragma-sopp-pdn-rtb-private |
| s3 | S3 | Object Storage | Storage | pragma-sopp-pdn-s3-data |
| sgr | Security Group | Network Security | Networking | pragma-sopp-pdn-sgr-api |
| sm | Secret Manager | Secrets Storage | Security | pragma-sopp-pdn-sm-database |
| sns | SNS | Notification Service | Messaging | pragma-sopp-pdn-sns-alerts |
| storage-gw | Storage Gateway | Hybrid Storage | Storage | pragma-sopp-pdn-storage-gw-cache |
| subnet-private | Subnet Private | Private Subnet | Networking | pragma-sopp-pdn-subnet-private-1a |
| subnet-public | Subnet Public | Public Subnet | Networking | pragma-sopp-pdn-subnet-public-1a |
| tg | Target Group | Load Balancer Target | Networking | pragma-sopp-pdn-tg-api |
| transit-gw | Transit Gateway | Network Transit | Networking | pragma-sopp-pdn-transit-gw-main |
| vpc | VPC | Virtual Private Cloud | Networking | pragma-sopp-pdn-vpc-main |
| vpc-peering | VPC Peering | VPC Connection | Networking | pragma-sopp-pdn-vpc-peering-remote |
| vpce | VPC Endpoint | VPC Endpoint | Networking | pragma-sopp-pdn-vpce-s3 |
| wacl | Web ACL WAF | WAF Rules | Security | pragma-sopp-pdn-wacl-api |

### **Azure**

**Restricciones Generales:**

* Límite de caracteres: Varía por recurso (1-90 caracteres típicamente)  
* Caracteres permitidos: a-z, 0-9, guiones (-), puntos (.), guiones bajos (\_)  
* Caracteres NO permitidos: Mayúsculas (case-insensitive), espacios, caracteres especiales  
* Inicio/Fin: Varía por recurso  
* Caso: Case-insensitive (pero se preserva)

**Restricciones por Servicio:**

| Servicio | Límite | Caracteres | Notas |
| ----- | ----- | ----- | ----- |
| Virtual Machines (Windows) | 15 | a-z, 0-9, guiones | Máximo 15 para Windows |
| Virtual Machines (Linux) | 64 | a-z, 0-9, guiones, puntos | Máximo 64 para Linux |
| Storage Accounts | 24 | a-z, 0-9 | Solo minúsculas y números, sin guiones |
| App Services | 60 | a-z, 0-9, guiones | Globalmente único |
| Virtual Networks | 64 | a-z, 0-9, guiones, puntos, guiones bajos | Dentro del resource group |
| Network Security Groups | 80 | a-z, 0-9, guiones, puntos, guiones bajos | Dentro del resource group |
| Key Vault | 24 | a-z, 0-9, guiones | Globalmente único |
| Cosmos DB | 44 | a-z, 0-9, guiones | Globalmente único |
| SQL Database | 128 | a-z, 0-9, guiones, puntos, guiones bajos | Dentro del servidor |

**Recomendación para Azure:** Usar máximo 24 caracteres para compatibilidad universal (Storage Accounts es el más restrictivo).

### **GCP**

**Restricciones Generales:**

* **Límite de caracteres:** 1-63 caracteres (RFC 1035 compatible)  
* **Caracteres permitidos:** a-z, 0-9, guiones (-)  
* **Caracteres NO permitidos**: Mayúsculas, puntos, guiones bajos, espacios, caracteres especiales  
* **Inicio:** Debe comenzar con letra minúscula  
* **Fin:** No puede terminar con guión  
* **Patrón Regex:** ^\[a-z\](\[-a-z0-9\]\*\[a-z0-9\])?$  
* **Caso:** Case-sensitive

**Restricciones por Servicio:**

| Servicio | Límite | Caracteres | Notas |
| ----- | ----- | ----- | ----- |
| Compute Engine Instances | 63 | a-z, 0-9, guiones | RFC 1035 compatible |
| Cloud Functions | 63 | a-z, 0-9, guiones | RFC 1035 compatible |
| Cloud Storage Buckets | 63 | a-z, 0-9, guiones, puntos | Globalmente único, sin mayúsculas |
| Cloud SQL Instances | 63 | a-z, 0-9, guiones | RFC 1035 compatible |
| VPC Networks | 63 | a-z, 0-9, guiones | RFC 1035 compatible |
| Firewall Rules | 63 | a-z, 0-9, guiones | RFC 1035 compatible |
| Service Accounts | 30 | a-z, 0-9, guiones | Máximo 30 caracteres |
| Cloud Pub/Sub Topics | 255 | a-z, 0-9, guiones, puntos, guiones bajos | Más flexible que Compute |

**Recomendación para GCP:** Usar máximo 30 caracteres para compatibilidad universal (Service Accounts es el más restrictivo).

## **Validación de Nombres**

## **Validación Regex**

```
^[a-z]([a-z0-9]{1,6}[a-z0-9])?-[a-z0-9]([a-z0-9-]{0,18}[a-z0-9])?$
```

## **Desglose:**

* ^\[a-z\] \- Comienza con letra minúscula  
* (\[a-z0-9\]{1,6}\[a-z0-9\])? \- Client name (3-8 caracteres)  
* \- \- Separador  
* \[a-z0-9\] \- Comienza con letra o número  
* (\[a-z0-9-\]{0,18}\[a-z0-9\])? \- Resto del nombre (máximo 20 caracteres)  
* $ \- Fin de string

## **Excepciones**

Las excepciones requieren aprobación del Arquitecto de Soluciones/Arquitecto CloudOps:

* Recursos heredados que no pueden renombrarse  
* Integraciones con sistemas externos que requieren nombres específicos  
* Limitaciones técnicas del proveedor

**Documentar excepciones en:**

* Registro de cambio  
* Comentario en IaC  
* Registro centralizado de excepciones

## **Referencias**

* [AWS Naming Best Practices](https://docs.aws.amazon.com/general/latest/gr/aws-naming-conventions.html)  
* [Azure Naming Conventions](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging)  
* [GCP Resource Naming](https://cloud.google.com/docs/naming-conventions)  
* [RFC 1123 \- DNS Naming](https://tools.ietf.org/html/rfc1123)

