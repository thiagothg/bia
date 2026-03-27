🚀 Desafio Prático — Aula 2 | Imersão AWS & IA com Henrylle Maia

Alta Disponibilidade, Deploy sem Downtime e Pipeline CI/CD completo na AWS. Aqui está tudo que foi implementado, do zero ao HTTPS em produção.

---

🗺️ **Step by Step técnico:**

**1️⃣ RDS PostgreSQL**
- Provisionamos uma instância RDS PostgreSQL t3.micro
- Configuramos o Security Group `bia-db` com acesso restrito na porta 5432
- Banco isolado em subnet privada, sem acesso público

**2️⃣ Security Groups — Princípio de Menor Privilégio**
- `bia-alb` → inbound 80/443 de 0.0.0.0/0 (tráfego público)
- `bia-ec2` → inbound All TCP apenas do `bia-alb` (portas dinâmicas do ECS)
- `bia-db` → inbound 5432 apenas do `bia-ec2` (isolamento total do banco)

**3️⃣ Application Load Balancer + Target Group**
- ALB provisionado com listener HTTP (80) e HTTPS (443)
- Target Group configurado com health check em `/api/versao`
- Instâncias EC2 do cluster registradas automaticamente via ECS Service

**4️⃣ Cluster ECS com EC2**
- Cluster `cluster-bia-alb` com instâncias EC2 t3.micro
- Task Definition `task-def-bia-alb` referenciando a imagem no ECR
- Service `service-bia-alb` com desired count > 1 para alta disponibilidade
- Rolling update configurado: nova task sobe antes da antiga ser encerrada — zero downtime

**5️⃣ Pipeline CI/CD — CodePipeline + CodeBuild**
- Stage Source: integração com GitHub, trigger automático via webhook no push da `main`
- Stage Build: CodeBuild executa o `buildspec.yml` — build da imagem Docker, tag com commit hash e push para o ECR
- Stage Deploy: ECS Service atualiza automaticamente com a nova image URI via `imagedefinitions.json`

**6️⃣ Domínio personalizado + HTTPS**
- Certificado SSL/TLS emitido via AWS Certificate Manager (ACM) com validação DNS
- Listener HTTPS (443) no ALB associado ao certificado ACM
- Registro CNAME no provedor de domínio externo apontando para o DNS do ALB
- Redirect automático de HTTP → HTTPS configurado no listener 80

---

🔗 **Como tudo se conecta:**

```
GitHub push (main)
    → CodePipeline trigger
        → CodeBuild: docker build + push ECR
            → ECS Service rolling update
                → ALB distribui tráfego entre tasks saudáveis
                    → RDS acessível apenas pelas tasks via Security Group
                        → HTTPS com certificado ACM validado
```

Cada camada tem sua responsabilidade isolada. Segurança por Security Groups em cadeia, disponibilidade garantida pelo ALB + ECS, e entrega contínua pelo pipeline — sem intervenção manual.

Isso é infraestrutura pensada para produção. 🏗️

Obrigado ao Henrylle Maia pela estrutura didática e pelo nível técnico do evento!

#AWS #DevOps #ECS #EC2 #RDS #PostgreSQL #ALB #ApplicationLoadBalancer #CICD #CodePipeline #CodeBuild #ECR #Docker #HTTPS #ACM #CloudComputing #ImersaoAWS #NodeJS #React #InfraAsCode #CloudArchitecture
