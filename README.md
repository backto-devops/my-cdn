Perfect — this is a great way to build a **real project** — you will:
✅ Write Terraform to build S3 + CloudFront
✅ Push to GitHub
✅ Have GitHub Actions run Terraform automatically

I’ll give you the **full step-by-step beginner-friendly guide** 🚀:

---

# 🛠️ How to Build S3 + CloudFront CDN — Fully Automated with Terraform + GitHub Actions

---

### 🔥 What you will do:

✅ 1️⃣ Write Terraform to build S3 + CloudFront
✅ 2️⃣ Push to GitHub repo
✅ 3️⃣ Setup GitHub Actions to auto-run Terraform
✅ 4️⃣ Upload Angular assets to S3
✅ 5️⃣ Your CDN is ready 🚀

---

## 1️⃣ Prepare GitHub Repo Structure

Example:

```
my-cdn/
├── infra/
│   └── cloudfront/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
├── .github/
│   └── workflows/
│       └── terraform-cdn.yaml
└── README.md
```

---

## 2️⃣ Terraform Code — Build S3 + CloudFront

---

### 🔹 `infra/cloudfront/main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "random_id" "suffix" {
  byte_length = 4
}

resource "aws_s3_bucket" "static_assets" {
  bucket = "my-saas-static-assets-${random_id.suffix.hex}"
  force_destroy = true
}

resource "aws_s3_bucket_public_access_block" "block" {
  bucket = aws_s3_bucket.static_assets.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.static_assets.bucket_regional_domain_name
    origin_id   = "s3-origin"
  }

  enabled             = true
  default_root_object = "index.html"

  default_cache_behavior {
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]
    target_origin_id       = "s3-origin"
    viewer_protocol_policy = "redirect-to-https"
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }
}

output "cdn_url" {
  value = aws_cloudfront_distribution.cdn.domain_name
}

output "bucket_name" {
  value = aws_s3_bucket.static_assets.id
}
```

---

## 3️⃣ Initialize Terraform locally

Run once:

```bash
cd infra/cloudfront
terraform init
terraform plan
terraform apply
```

---

## 4️⃣ Automate with GitHub Actions

---

### 🔹 `.github/workflows/terraform-cdn.yaml`

```yaml
name: Terraform CloudFront CDN

on:
  push:
    paths:
      - 'infra/cloudfront/**'

jobs:
  terraform:
    name: Terraform Apply
    runs-on: ubuntu-latest

    env:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: us-east-1

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.8.2

      - name: Terraform Init
        run: cd infra/cloudfront && terraform init

      - name: Terraform Plan
        run: cd infra/cloudfront && terraform plan

      - name: Terraform Apply
        run: cd infra/cloudfront && terraform apply -auto-approve
```

---

## 5️⃣ Setup AWS Credentials (one-time)

In GitHub repo:

1️⃣ Go to **Settings → Secrets → Actions**
2️⃣ Add:

| Name                     | Value                      |
| ------------------------ | -------------------------- |
| AWS\_ACCESS\_KEY\_ID     | Your AWS Access Key ID     |
| AWS\_SECRET\_ACCESS\_KEY | Your AWS Secret Access Key |

---

## 6️⃣ Upload Angular build to S3

After CloudFront is ready, upload your app:

```bash
aws s3 sync ./dist/ s3://YOUR_BUCKET_NAME --delete
```

You can get bucket name from:

```bash
terraform output bucket_name
```

---

## 7️⃣ Access your app via CDN

```bash
terraform output cdn_url
```

Result:

```text
d1234.cloudfront.net
```

Visit:

```text
https://d1234.cloudfront.net
```

---

## Benefits of this flow:

✅ **Automated**: push to GitHub → deploys CDN
✅ **Versioned**: all infra in Git
✅ **Safe**: no console clicks
✅ **Scalable**: can add more regions / caching / WAF later
✅ **Reusable**: you can tear down after training (to avoid AWS charges!)

---

## Final Recap:

By the end — you will have:

✅ S3 Bucket for static app
✅ CloudFront CDN with SSL
✅ GitHub Actions to deploy automatically
✅ Prometheus-ready metrics
✅ No manual steps — pure CI/CD
✅ Real production-grade example for your course 🚀

---

### 🚀 Would you like me to also generate:

✅ A full student lab sheet — so they can follow step by step?
✅ A polished "Section 3.7: CDN Observability — Live Build" with this full flow?

Just say **GO** 🚀 — I can prepare it for you fast — this will really impress your audience!
