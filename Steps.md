
# 🧭 COMPLETE WORKFLOW 

---

## 🟢 STEP-1 : Create VPC

![Image](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2023/02/02/2023-vpc-resource-map-1.jpg)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/subnet-cidrs.png)

1. AWS Console open karo → search **VPC**
2. Click 👉 **Create VPC**
3. Choose:

   * VPC only
4. Fill:

   * Name: `ThreeTier-VPC`
   * CIDR: `10.0.0.0/16`
5. Create.

**Purpose:**
Ye tumhara ghar hai. Saare tiers isi boundary me rahenge.

---

## 🟢 STEP-2 : Create Subnets (8)

![Image](https://kodekloud.com/kk-media/image/upload/v1752863367/notes-assets/images/AWS-Networking-Fundamentals-Subnets-Demo/aws-management-console-subnets-vpc-details.jpg)

![Image](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/region-with-wavelength-zones.png)

Create these EXACT:

* Public-Subnet-A → 10.0.0.0/20 → AZ-A

* Public-Subnet-B → 10.0.16.0/20 → AZ-B

* Web-Subnet-A → 10.0.32.0/20

* Web-Subnet-B → 10.0.48.0/20

* App-Subnet-A → 10.0.64.0/20

* App-Subnet-B → 10.0.80.0/20

* DB-Subnet-A → 10.0.96.0/20

* DB-Subnet-B → 10.0.112.0/20

**Important AWS rule:**
RDS ko minimum 2 AZ ke subnets chahiye.

---

## 🟢 STEP-3 : Internet Gateway

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2Agftv4LSqU_12kRqNwYISJw.png)

![Image](https://help.yeastar.com/en/p-series-cloudpbx-deployment/images/aws-create-and-attach-igw-to-vpc.png)

1. Internet Gateway banao → `ThreeTier-IGW`
2. Attach to `ThreeTier-VPC`
3. Public RT me route add karo:

```
0.0.0.0/0 → IGW
```

---

## 🟢 STEP-4 : Route Tables

![Image](https://campus.barracuda.com/resources/attachments/image/100369028/1/rd_ass.png)

![Image](https://cloudiofy.com/wp-content/uploads/2022/07/route-table.png)

### Public-RT

* Associate: Public subnets only

### Private-RT

* Associate: Web, App, DB

❌ Private me IGW mat daalna.

---

## 🟢 STEP-5 : Security Groups Chain

![Image](https://website-assets.atlan.com/img/set-up-inbound-outbound-rules.webp)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/security-group-details.png)

Create:

1. **ALB-SG**

   * HTTP 80 → 0.0.0.0/0

2. **Web-SG**

   * HTTP 80 → ALB-SG
   * SSH 22 → Your IP

3. **App-SG**

   * HTTP 80 → Web-SG
   * SSH 22 → Bastion-SG (abhi create karenge)

4. **DB-SG**

   * 3306 → App-SG

This is firewall poetry — layers like fort walls.

---

## 🟢 STEP-6 : Create DB Subnet Group (MANUALLY FIRST)

![Image](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/con-VPC-sec-grp.png)

![Image](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/vpc-example-web-database.png)

RDS dashboard → Subnet Groups → Create:

```
Name: db-subnet-group
VPC: ThreeTier-VPC
Select: DB-Subnet-A & DB-Subnet-B only
```

---

## 🟢 STEP-7 : Create RDS MySQL

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2ALD5ek0V70cGWS3oH1GoW5g.png)

![Image](https://docs.aws.amazon.com/images/AmazonRDS/latest/UserGuide/images/endpoint-port.png)

Configure STRICT:

* Engine: MySQL
* VPC: ThreeTier-VPC
* Subnet group: `db-subnet-group`
* Public access: **No**
* SG: `DB-SG`

Create and WAIT → available.

---

## 🟢 STEP-8 : Launch Bastion (Jump Host)

![Image](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2017/03/30/ssm-1024x699.png)

![Image](https://d2908q01vomqb2.cloudfront.net/22d200f8670dbdb3e253a90eee5098477c95c23d/2021/05/02/EC2-Instance-Connect-SSH-Access-2.png)

EC2 with:

* Subnet: Public
* Key: `private.pem`
* SG: `Bastion-SG`

### Windows CMD concept we used

From laptop:

```powershell
scp -i "C:\Users\user5\Downloads\private.pem" "C:\Users\user5\Downloads\private.pem" ec2-user@<BASTION_PUBLIC_IP>:/home/ec2-user/
```

Inside bastion:

```bash
chmod 400 private.pem
```

---

## 🟢 STEP-9 : SSH FROM PUBLIC → PRIVATE IP

![Image](https://miro.medium.com/1%2AF7zFRVkKxqzYCDZ7ktv9vQ.png)

![Image](https://cdn.prod.website-files.com/58fe8f93dc9e750ca84ebb16/5ac64b5e9912212018f15f6a_aws-image2.png)

FROM BASTION TERMINAL:

```bash
ssh -i private.pem ec2-user@10.0.70.234
```

✔ This is the command path
❌ Without this key, permission impossible

---

## 🟢 STEP-10 : Launch App Server BASE

![Image](https://tonythomas.in/wp-content/uploads/2024/05/install-nginx-web-server-on-an-aws-ec2-instance-using-user-data-1-1024x480.webp)

![Image](https://media.geeksforgeeks.org/wp-content/uploads/20230206194929/14.png)

Use:

* AMI: Amazon Linux 2023
* Subnet: App
* Key: private.pem
* SG: App-SG

Install:

```bash
sudo yum update -y
sudo yum install nginx php php-fpm php-mysqlnd mariadb105 -y
sudo systemctl start nginx php-fpm
sudo systemctl enable nginx php-fpm
```

Create php page and test.

---

## 🟢 STEP-11 : Launch Web Server BASE

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2A6sPGsOKGY6tfVRRCpmyLDA.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2AB2Vs3jydEmwc_pnKbAFx1A.png)

```bash
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

## 🟢 STEP-12 : Create AMIs

![Image](https://kodekloud.com/kk-media/image/upload/v1752868962/notes-assets/images/Amazon-Elastic-Compute-Cloud-EC2-Demo-Create-AMI-from-AWS-Console/aws-ec2-console-ami-details.jpg)

![Image](https://kodekloud.com/kk-media/image/upload/v1752868963/notes-assets/images/Amazon-Elastic-Compute-Cloud-EC2-Demo-Create-AMI-from-AWS-Console/aws-ec2-snapshot-details-console.jpg)

* WebTier-AMI
* AppTier-AMI

Terminate base instances after AMI.

---

## 🟢 STEP-13 : Create Launch Templates

![Image](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/launch-template-ami-alias.png)

![Image](https://media.amazonwebservices.com/blog/2018/lt_prod_1.png)

Templates for:

* Web → WebTier-AMI
* App → AppTier-AMI

---

## 🟢 STEP-14 : Auto Scaling Groups

![Image](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2018/01/12/AS-group-resources-1.png)

![Image](https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2018/02/14/elb-hc-nc.png)

For both tiers:

* Min 1
* Desired 1
* Max 4
* Attach TG

---

## 🟢 STEP-15 : Create Target Groups

![Image](https://images.ctfassets.net/zsv3d0ugroxu/IhdBFJpNKgwHl-6kcJR88/fab06edefd4edf2baa086954471d15df/elb1.png)

![Image](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2023/04/10/figure_3-1024x688.png)

* Web-TG → 80 → /
* App-TG → 80

---

## 🟢 STEP-16 : Create Load Balancers

![Image](https://d2908q01vomqb2.cloudfront.net/5b384ce32d8cdef02bc3a139d4cac0a22bb029e8/2018/08/23/image-1.1-1024x655.png)

![Image](https://docs.aws.amazon.com/images/elasticloadbalancing/latest/classic/images/internal_load_balancer.png)

1. Public ALB → Public subnets → Web-TG
2. Internal ALB → App subnets → App-TG

---

## 🟢 STEP-17 : FINAL TEST

Open ALB DNS → browser.

Kill instance → ASG replace → check.

---

# 🎯 YOU ARE DONE

---

