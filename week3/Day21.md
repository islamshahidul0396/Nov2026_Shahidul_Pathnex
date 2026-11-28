# Day 21 — SPEED TYPING (LEVEL 2)

You must rewrite all FOUR from scratch:


---
- name: Install and configure Nginx
  hosts: all
  become: yes

  tasks:
    - name: Install Nginx
      yum:
        name: nginx
        state: present

    - name: Start and enable Nginx service
      service:
        name: nginx
        state: started
        enabled: yes



provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "pathnex_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t3.micro"

  tags = {
    Name  = "Pathnex-EC2"
    Owner = "PathnexStudent"
  }
}



apiVersion: apps/v1
kind: Deployment
metadata:
  name: pathnex-nginx
  labels:
    app: nginx

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80



#!/bin/bash

echo "================================"
echo "       SYSTEM INFORMATION"
echo "================================"

echo "Current Date and Time:"
date

echo ""
echo "System Uptime:"
uptime

echo ""
echo "Disk Usage:"
df -h

echo "================================"

⏱ TIME LIMIT: 15 minutes

# Multi-Environment Pipeline

## 🔹 Jenkins Pipeline — Deploy to Multiple Environments
You will learn how to **define multiple environment deployments in one pipeline**.

pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Deploy to Dev') {
            steps {
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
                input message: "Approve Prod Deployment?"
                sh 'echo "Deploying $INSTITUTE_NAME to Prod environment"'
            }
        }
    }
}

## 🔹 GitLab CI — Multi-Environment
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