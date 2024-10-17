#Terraform을 사용하여 3-Tier 아키텍처를 구성

1. VPC 생성
2. 각 티어별 서브넷 생성 (공개 및 프라이빗)
3. 인터넷 게이트웨이 및 NAT 게이트웨이 설정
4. 라우팅 테이블 구성
5. 각 티어별 보안 그룹 설정

## VPC 생성
vpc.tf
```HCL
# CIDR 블록 변수 선언
variable "app-subnet-cidr" {
  description = "CIDR block for the first application subnet"
  type        = string
  default     = "10.0.0.0/16"  # 예시 CIDR 블록
}

resource "aws_vpc" "vpc"{
    cidr_block = var.app-subnet-cidr

    tags = {
        Name = "ce14-2.terraform-vpc"
    }
}
```
![image](https://github.com/user-attachments/assets/0ea21135-c00b-4081-a952-6a738cb883fe)

## 각 티어별 서브넷 생성
web-subnets.tf
```HCL
# CIDR 블록 변수 선언
variable "web-subnet1-cidr" {
  description = "CIDR block for the first application subnet"
  type        = string
  default     = "10.0.1.0/24"  # 예시 CIDR 블록
}

variable "web-subnet2-cidr" {
  description = "CIDR block for the second application subnet"
  type        = string
  default     = "10.0.2.0/24"  # 예시 CIDR 블록
}


# 첫 번째 애플리케이션 서브넷 (web-subent1) 생성
resource "aws_subnet" "web-subent1" {
  vpc_id                  = aws_vpc.vpc.id  
  cidr_block              = var.web-subnet1-cidr
  
  availability_zone       = "ap-northeast-2a" 

  map_public_ip_on_launch = true

  tags = {
    Name = "ce14-terraform-app_subnet1"
  }
}

# 두 번째 애플리케이션 서브넷 (web-subnet2) 생성
resource "aws_subnet" "web-subnet2" {
  vpc_id                  = aws_vpc.vpc.id  #vpc.tf 파일의 resource 자원 참조
  cidr_block              = var.web-subnet2-cidr  
  availability_zone       = "ap-northeast-2b"    

  #인스턴스가 이 서브넷에 생성될 때 자동으로 퍼블릭 IP 할당 여부
  map_public_ip_on_launch = true

  tags = {
    Name = "ce14-terraform-app_subnet2"
  }
}
```
![image](https://github.com/user-attachments/assets/5bf6f88d-5bf3-4e59-a203-fcbeb5f4b554)

## 인터넷 게이트웨이 및 NAT 게이트웨이 설정

```bash
# 인터넷 게이트웨이 생성
resource "aws_internet_gateway" "internet-gw" {
  # 이 인터넷 게이트웨이가 연결될 VPC의 ID. aws_vpc 리소스에서 생성된 VPC를 참조.
  vpc_id = aws_vpc.vpc.id

  # 인터넷 게이트웨이의 태그. AWS 콘솔에서 식별을 쉽게 하기 위해 Name 태그 설정.
  tags = {
    Name = "ce14-terraform-igw"  # 인터넷 게이트웨이 이름
  }
}
```

```bash
# NAT 게이트웨이 생성
resource "aws_nat_gateway" "nat-gw" {
  # NAT 게이트웨이에 연결할 Elastic IP의 ID. aws_eip 리소스에서 생성된 EIP를 참조.
  allocation_id     = aws_eip.eip.id
  
  # NAT 게이트웨이의 연결 유형. public으로 설정하여 퍼블릭 NAT 게이트웨이로 만듦.
  connectivity_type = "public"
  
  # NAT 게이트웨이가 속할 서브넷의 ID. aws_subnet 리소스에서 생성된 서브넷을 참조.
  subnet_id         = aws_subnet.web-subnet1.id
  
  # NAT 게이트웨이의 태그. AWS 콘솔에서 식별을 쉽게 하기 위해 Name 태그 설정.
  tags = {
    Name = "ce00-terraform-ngw"  # NAT 게이트웨이의 이름
  }

  # 이 NAT 게이트웨이는 인터넷 게이트웨이에 의존하므로 depends_on 사용.
  depends_on = [aws_internet_gateway.internet-gw]
}
```

### 라우팅 테이블 구성
``` bash
# 퍼블릭 라우트 테이블 생성
resource "aws_route_table" "private-route-table" {
  # 라우트 테이블이 속할 VPC의 ID. aws_vpc 리소스에서 생성된 VPC를 참조.
  vpc_id = aws_vpc.vpc.id

  # 기본 라우트 설정. 모든 트래픽을 인터넷 게이트웨이로 라우팅.
  route {
    cidr_block = "0.0.0.0/0"  # 모든 IP 주소에 대한 트래픽
    gateway_id = aws_nat_gateway.nat-gw.id  # 인터넷 게이트웨이를 통해 라우팅
  }

  # 라우트 테이블의 태그. AWS 콘솔에서 식별을 쉽게 하기 위해 Name 태그 설정.
  tags = {
    Name = "ce14-terraform-private-rt"  # 퍼블릭 라우트 테이블의 이름
  }
}

# 첫 번째 퍼블릭 라우트 테이블과 서브넷의 연결
resource "aws_route_table_association" "private-rt-association-1" {
  # 연결할 서브넷의 ID. aws_subnet 리소스에서 생성된 서브넷을 참조.
  subnet_id      = aws_subnet.db-subnet1.id
  
  # 연결할 라우트 테이블의 ID. aws_route_table 리소스에서 생성된 라우트 테이블을 참조.
  route_table_id = aws_route_table.private-route-table.id
}

# 두 번째 퍼블릭 라우트 테이블과 서브넷의 연결
resource "aws_route_table_association" "private-rt-association-2" {
  subnet_id      = aws_subnet.db-subnet2.id
  route_table_id = aws_route_table.private-route-table.id
}
```

```bash
# 퍼블릭 라우트 테이블 생성
resource "aws_route_table" "public-route-table" {
  # 라우트 테이블이 속할 VPC의 ID. aws_vpc 리소스에서 생성된 VPC를 참조.
  vpc_id = aws_vpc.vpc.id

  # 기본 라우트 설정. 모든 트래픽을 인터넷 게이트웨이로 라우팅.
  route {
    cidr_block = "0.0.0.0/0"  # 모든 IP 주소에 대한 트래픽
    gateway_id = aws_internet_gateway.internet-gw.id  # 인터넷 게이트웨이를 통해 라우팅
  }

  # 라우트 테이블의 태그. AWS 콘솔에서 식별을 쉽게 하기 위해 Name 태그 설정.
  tags = {
    Name = "ce14-terraform-public-rt"  # 퍼블릭 라우트 테이블의 이름
  }
}

# 첫 번째 퍼블릭 라우트 테이블과 서브넷의 연결
resource "aws_route_table_association" "pub-rt-association-1" {
  # 연결할 서브넷의 ID. aws_subnet 리소스에서 생성된 서브넷을 참조.
  subnet_id      = aws_subnet.web-subnet1.id
  
  # 연결할 라우트 테이블의 ID. aws_route_table 리소스에서 생성된 라우트 테이블을 참조.
  route_table_id = aws_route_table.public-route-table.id
}

# 두 번째 퍼블릭 라우트 테이블과 서브넷의 연결
resource "aws_route_table_association" "pub-rt-association-2" {
  subnet_id      = aws_subnet.web-subnet2.id
  route_table_id = aws_route_table.public-route-table.id
}
```

![image](https://github.com/user-attachments/assets/e8458a86-78e4-4e74-a8dd-7f7bdf78415c)

`terraform init`
`terraform apply`
