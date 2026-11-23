# Day 21 - SPEED TYPING (LEVEL2)

# Ansible
---
- name: Install and configure Nginx on Pathnex server
  hosts: pathnex-server
  become: yes

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Start and enable Nginx service
      systemd:
        name: nginx
        state: started
        enabled: yes

# Terraform

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "pathnex_server" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "c5.xlarge"

  tags = {
    Name  = "Pathnex-Server"
    Owner = "PathnexStudent"
  }
}


# Kubernetes

apiVersion: apps/v1
kind: Deployment
metadata:
  name: pathnex-nginx
  labels:
    app: pathnex-nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: pathnex-nginx

  template:
    metadata:
      labels:
        app: pathnex-nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80

# Shell Script

#!/bin/bash

echo "================================"
echo "     PATHNEX SERVER INFO"
echo "================================"

echo ""
echo "Current Date:"
date

echo ""
echo "System Uptime:"
uptime

echo ""
echo "Disk Usage:"
df -h

echo ""
echo "================================"
echo "   SERVER CHECK COMPLETED"
echo "================================"

# Multi-Environment Pipeline

## Jenkins Pipeline - Display to Multiple Environments
You will learn how to **define multiple environment deployments in one pipeline**.


pipeline {
    agent any

    environment {
        INSTITUTE_NAME = "Pathnex"
    }

    stages {

        stage('Deploy to Dev') {
            steps {
                input message: "Approve Dev Deployment?"
                sh 'echo "Deploying $INSTITUTE_NAME to Dev environment"'
            }
        }

        stage('Deploy to Staging') {
            steps {
                input message: "Approve Staging Deployment?"
                sh 'echo "Deploying $INSTITUTE_NAME to Staging environment"'
            }
        }

        stage('Deploy to Prod') {
            steps {
                input message: "Approve Production Deployment?"
                sh 'echo "Deploying $INSTITUTE_NAME to Production environment"'
            }
        }
    }
}



## GitLab CI - Multi-Environment
You will learn how to **define multiple environment deployments in GitLab CI**.


stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"

deploy-dev:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME to Dev environment"

deploy-staging:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME to Staging environment"
  when: manual

deploy-prod:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME to Prod environment"
  when: manual


