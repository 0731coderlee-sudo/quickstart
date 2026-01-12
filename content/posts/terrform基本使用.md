+++
date = '2026-01-13T00:41:40+08:00'
draft = false
title = 'Terraform 基本使用（Docker + Nginx 示例）'
+++

用一个本地 Docker + Nginx 的示例，速览 Terraform 从 0 到 1 的基本用法。

## 1. 准备
- 安装 Terraform 与 Docker（本例直接使用 Docker provider）
- 建议新建目录 `terraform-docker-nginx/`，所有配置放在里面

## 2. 最小示例 main.tf
```hcl
terraform {
  required_version = ">= 1.6"
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {}

# 创建一个专用网络
resource "docker_network" "web" {
  name = "tf-web"
}

# 拉取 Nginx 镜像
resource "docker_image" "nginx" {
  name         = "nginx:1.27-alpine"
  keep_locally = true
}

# 运行 Nginx 容器
resource "docker_container" "nginx" {
  name  = "tf-nginx"
  image = docker_image.nginx.image_id
  ports {
    internal = 80
    external = 8080
  }
  networks_advanced {
    name = docker_network.web.name
  }
}
```

## 3. 工作流
在配置目录执行：
1) 初始化 provider 插件  
   `terraform init`
2) 预览变更计划  
   `terraform plan`
3) 应用并创建资源  
   `terraform apply` → 输入 `yes`  
   现在访问 <http://localhost:8080> 可看到 Nginx 欢迎页
4) 变更配置时重复 plan/apply；需要回滚时改回配置再 apply
5) 清理资源  
   `terraform destroy`

## 4. 小贴士
- 状态文件 `terraform.tfstate` 很重要，建议纳入备份或远端状态（本地实验可忽略远端）
- 变量与多环境：可用 `terraform.tfvars`、`-var-file` 管理端口、镜像版本等参数
- 想扩展：换成自定义 Nginx 配置 volume、增加多个容器、或接入云厂商 provider 都是类似写法

## 5. 变量与参数化示例
在 `variables.tf` 定义默认端口与镜像：
```hcl
variable "http_port" {
  type    = number
  default = 8080
}

variable "nginx_image" {
  type    = string
  default = "nginx:1.27-alpine"
}
```
在 `main.tf` 使用变量：
```hcl
resource "docker_image" "nginx" {
  name         = var.nginx_image
  keep_locally = true
}

resource "docker_container" "nginx" {
  # ...
  ports {
    internal = 80
    external = var.http_port
  }
}
```
切换端口/镜像：`terraform apply -var="http_port=8000" -var="nginx_image=nginx:1.26-alpine"`

## 6. 常见目录结构
```
terraform-docker-nginx/
├─ main.tf            # 核心资源
├─ variables.tf       # 变量声明
├─ terraform.tfvars   # 变量取值（可选）
├─ outputs.tf         # 输出值（如容器 IP）
├─ .terraform.lock.hcl# Provider 锁定版本
└─ terraform.tfstate  # 状态文件（不要手动改）
```
将 `terraform.tfstate` 加入 `.gitignore`，实验环境用本地状态即可；多人协作推荐远端状态后端（S3、OSS、GCS 等）。

## 7. 常见问题速查
- `Error: docker provider not found` → 确认 `terraform init` 已执行，或删除 `.terraform` 重新 init
- `port is already allocated` → 调整 `http_port`，避免与本机已有容器冲突
- 镜像下载慢 → 先手动拉取 `docker pull nginx:1.27-alpine`，再 `terraform apply`
- 想复用现有资源 → 可用 `terraform import docker_container.nginx tf-nginx` 把已存在容器纳入管理
