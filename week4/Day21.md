# Day 22 - Handlers, Security Group, DaemonSet, Disk Monitor

## Ansible - Use Handler on Config Chane

---
- name: Configure Nginx for Pathnex
  hosts: all
  become: yes

  tasks:
    - name: Copy Nginx config
      copy:
        dest: /etc/nginx/nginx.conf
        content: |
          user nginx;
          worker_processes auto;
          pid /run/nginx.pid;
      notify: Restart nginx

    - name: Validate Nginx configuration
      command: nginx -t
      changed_when: false

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
        enabled: yes



## Terraform - Security Group


# Get latest Amazon Linux 2023 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }
}

# 1. Create VPC
resource "aws_vpc" "PathnexVPC" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "Pathnex-VPC"
  }
}

# 2. Create Public Subnet
resource "aws_subnet" "PathnexSubnet" {
  vpc_id                  = aws_vpc.PathnexVPC.id
  cidr_block              = var.subnet_cidr
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "Pathnex-Public-Subnet"
  }
}

# 3. Create Internet Gateway
resource "aws_internet_gateway" "PathnexIGW" {
  vpc_id = aws_vpc.PathnexVPC.id

  tags = {
    Name = "Pathnex-IGW"
  }
}

# 4. Create Route Table
resource "aws_route_table" "PathnexRT" {
  vpc_id = aws_vpc.PathnexVPC.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.PathnexIGW.id
  }

  tags = {
    Name = "Pathnex-Route-Table"
  }
}

# 5. Associate Route Table with Subnet
resource "aws_route_table_association" "PathnexAssociation" {
  subnet_id      = aws_subnet.PathnexSubnet.id
  route_table_id = aws_route_table.PathnexRT.id
}

# 6. Create Security Group
resource "aws_security_group" "PathnexSG" {
  name        = "pathnex-sg"
  description = "Allow HTTP and HTTPS traffic"
  vpc_id      = aws_vpc.PathnexVPC.id

  ingress {
    description = "Allow HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Allow HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  dynamic "ingress" {
    for_each = var.ssh_allowed_cidr == "" ? [] : [var.ssh_allowed_cidr]

    content {
      description = "Allow SSH from trusted IP"
      from_port   = 22
      to_port     = 22
      protocol    = "tcp"
      cidr_blocks = [ingress.value]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "Pathnex-SG"
  }
}

# 7. Create EC2 Instance
resource "aws_instance" "PathnexEC2" {
  ami                    = data.aws_ami.amazon_linux.id
  instance_type          = var.instance_type
  subnet_id              = aws_subnet.PathnexSubnet.id
  vpc_security_group_ids = [aws_security_group.PathnexSG.id]

  user_data = <<-EOF
    #!/bin/bash
    dnf install -y nginx
    systemctl enable nginx
    systemctl start nginx

    echo "<h1>Welcome to Pathnex DevOps Project!</h1>" \
      > /usr/share/nginx/html/index.html
  EOF

  depends_on = [
    aws_route_table_association.PathnexAssociation
  ]

  tags = {
    Name = "Pathnex-EC2"
  }
}


## Kubernetes - DaemonSet

apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: pathnex-daemon

spec:
  selector:
    matchLabels:
      app: ds

  template:
    metadata:
      labels:
        app: ds

    spec:
      containers:
        - name: monitor
          image: busybox
          command:
            - sh
            - -c
            - while true; do echo Pathnex Node Monitor; sleep 20; done


## Shell Script - Monitor Disk Usage

#!/bin/bash

USAGE=$(df -h / | tail -1 | awk '{print $5}')

echo "Disk Usage: $USAGE"

# Advanced Triggers
## Jenkins Pipeline - Trigger on Git Tag
You will learn how to **trigger builds on Git tags** with realistic tags.


pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
        TEAM = "DevOps"
        ENV = "prod"
    }

    triggers {
        pollSCM('H/15 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git',
                    branch: 'main'
            }
        }

        stage('Build') {
            when {
                buildingTag()
            }

            steps {
                sh '''
                    echo "Building $INSTITUTE_NAME app for $TEAM team in $ENV environment"
                '''
            }
        }
    }
}


## GitLab CI - Trigger on Tag
You will learn how to **trigger CI jobs when a tag is pushed**.


stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"
  TEAM: "DevOps"
  ENV: "Prod"

build:
  stage: build
  image: maven:3.8.1-jdk-17

  script:
    - echo "Building $INSTITUTE_NAME app for $TEAM team in $ENV environment"
    - mvn clean package

  only:
    - tags
