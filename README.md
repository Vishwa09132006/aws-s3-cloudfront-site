# Static Portfolio Site — AWS S3 + CloudFront

My personal portfolio site, (which is not done yet, but I have done this to demonstrate the process I went through) I deployed on AWS S3 and served globally through CloudFront with HTTPS. You can upload any static files to the S3 bucket, set up the bucket permissions, enable static website hosting for that file and CloudFront will deliver it, but I chose to use my temporary cloud portfolio. 

**Live URL:** https://d2h1uboex2xxrt.cloudfront.net

---

## Architecture

```
Browser → CloudFront (HTTPS + global CDN) → S3 Bucket (static files)
```

## Screenshots

### Live Site
<img width="1912" height="1026" alt="s3-staticwebsite-cloudfront-hosted" src="https://github.com/user-attachments/assets/7b292a4f-4eba-425f-b398-5de15a8fa0b1" />

### S3 Bucket
<img width="1508" height="516" alt="aws-staticwebsite-cloudfront-s3bucket" src="https://github.com/user-attachments/assets/674e5ff6-77bf-4087-b66f-241ff7bce90e" />

### CloudFront Distribution
<img width="1894" height="762" alt="aws-s3-staticwebsite-cloudfront-distribution" src="https://github.com/user-attachments/assets/59d039b0-445a-40de-9a54-3f8213c8d729" />

- **S3** stores the static HTML file
- **CloudFront** sits in front of S3 and handles HTTPS, caching, and global delivery
- **Bucket policy** allows CloudFront to read files from S3

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
- IAM least-privilege security practices
- Static site architecture on AWS
- Linux / WSL2 environment
