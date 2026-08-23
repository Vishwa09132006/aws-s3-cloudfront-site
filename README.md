# Static Portfolio Site — AWS S3 + CloudFront

A personal portfolio site deployed on AWS S3 and served globally via CloudFront with HTTPS. 

**Live URL:** https://d2h1uboex2xxrt.cloudfront.net

---

## Architecture

```
Browser → CloudFront (HTTPS + global CDN) → S3 Bucket (static files)
```

- **S3** stores the static HTML file
- **CloudFront** sits in front of S3 and handles HTTPS, caching, and global delivery
- **Bucket policy** allows CloudFront to read files from S3
- No server, no EC2, no backend — just static files served at scale

---

## What it does

- Serves a static HTML portfolio page over HTTPS from 400+ CloudFront edge locations worldwide
- Automatically cached globally so visitors get fast load times regardless of location
- Zero server maintenance — S3 and CloudFront are fully managed AWS services

---

## AWS Services Used

| Service | Purpose |
|---------|---------|
| S3 | Stores and hosts the static HTML file |
| CloudFront | CDN — adds HTTPS, caching, and global delivery |
| IAM | CLI user with least-privilege permissions (S3 + CloudFront only) |

---

## How it was built

Everything was provisioned using the AWS CLI from WSL2 on Windows.

**1. Create S3 bucket**
```bash
aws s3 mb s3://vp-cloud-portfolio-2026 --region us-east-1
```

**2. Upload the site**
```bash
aws s3 cp index.html s3://vp-cloud-portfolio-2026/
```

**3. Enable static website hosting**
```bash
aws s3 website s3://vp-cloud-portfolio-2026/ --index-document index.html
```

**4. Disable Block Public Access**
```bash
aws s3api put-public-access-block \
  --bucket vp-cloud-portfolio-2026 \
  --public-access-block-configuration \
  "BlockPublicAcls=false,IgnorePublicAcls=false,BlockPublicPolicy=false,RestrictPublicBuckets=false"
```

**5. Apply bucket policy**
```bash
aws s3api put-bucket-policy \
  --bucket vp-cloud-portfolio-2026 \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::vp-cloud-portfolio-2026/*"
      }
    ]
  }'
```

**6. Create CloudFront distribution**
```bash
aws cloudfront create-distribution \
  --origin-domain-name vp-cloud-portfolio-2026.s3-website-us-east-1.amazonaws.com \
  --default-root-object index.html
```

---

## Cost

| Service | Free Tier | Actual Usage |
|---------|-----------|--------------|
| S3 | 5GB storage, 20k GET requests/month | ~4KB |
| CloudFront | 1TB transfer, 10M requests/month | Minimal |

**Total cost: $0/month** within free tier limits

## Skills demonstrated

- AWS CLI (S3, CloudFront, IAM)
- Cloud infrastructure deployment without the console
- IAM least-privilege security practices
- Static site architecture on AWS
- Linux / WSL2 environment
