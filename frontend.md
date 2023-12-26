

# frontend deployment

1. s3----S3 Module (s3-frontend-module):

```
s3-frontend-module/
|-- main.tf
|-- variables.tf
|-- outputs.tf

```
main.tf:
```
provider "aws" {
  region = var.region
}

resource "aws_s3_bucket" "frontend_bucket" {
  bucket = var.bucket_name
  acl    = "public-read"

  website {
    index_document = "index.html"
    error_document = "error.html"
  }
}

output "bucket_name" {
  value = aws_s3_bucket.frontend_bucket.bucket
}

```

variables.tf:

```
variable "region" {
  description = "AWS region"
}

variable "bucket_name" {
  description = "Name for the S3 bucket"
}

```

outputs.tf:
```
output "website_endpoint" {
  value = aws_s3_bucket.frontend_bucket.website_endpoint
}

```




2. acm---ACM Module (acm-module):
Directory Structure:
```
acm-module/
|-- main.tf
|-- variables.tf
|-- outputs.tf

```
main.tf:
```
provider "aws" {
  region = var.region
}

resource "aws_acm_certificate" "frontend_cert" {
  domain_name       = var.domain_name
  validation_method = "DNS"
}

output "acm_certificate_arn" {
  value = aws_acm_certificate.frontend_cert.arn
}

```
variables.tf:
```
variable "region" {
  description = "AWS region"
}

variable "domain_name" {
  description = "Domain name for the ACM certificate"
}

```
outputs.tf:
```
output "acm_certificate_arn" {
  value = aws_acm_certificate.frontend_cert.arn
}

```


3. cloudfront---CloudFront Module (cloudfront-frontend-module):
```
cloudfront-frontend-module/
|-- main.tf
|-- variables.tf
|-- outputs.tf

```
main.tf
```
provider "aws" {
  region = var.region
}

resource "aws_cloudfront_distribution" "frontend_distribution" {
  origin {
    domain_name = var.origin_domain_name
    s3_origin_config {
      origin_access_identity = ""
    }
  }

  enabled             = true
  is_ipv6_enabled     = true
  comment             = "Frontend distribution"
  default_root_object = "index.html"

  aliases = [var.domain_name]

  default_cache_behavior {
    target_origin_id = aws_s3_bucket.frontend_bucket.bucket_regional_domain_name
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods = ["GET", "HEAD", "OPTIONS"]
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }
}

output "cloudfront_domain_name" {
  value = aws_cloudfront_distribution.frontend_distribution.domain_name
}

output "cloudfront_distribution_id" {
  value = aws_cloudfront_distribution.frontend_distribution.id
}

```
variables.tf:

```
variable "region" {
  description = "AWS region"
}

variable "domain_name" {
  description = "Domain name for the CloudFront distribution"
}

variable "origin_domain_name" {
  description = "Domain name of the S3 bucket serving as the origin"
}

```
outputs.tf:
```
output "cloudfront_domain_name" {
  value = aws_cloudfront_distribution.frontend_distribution.domain_name
}

output "cloudfront_distribution_id" {
  value = aws_cloudfront_distribution.frontend_distribution.id
}

```

4. route53---Route 53 Module (route53-module):

```
route53-module/
|-- main.tf
|-- variables.tf
|-- outputs.tf

```

```
provider "aws" {
  region = var.region
}

resource "aws_route53_zone" "frontend_zone" {
  name = var.domain_name
}

resource "aws_route53_record" "frontend_record" {
  zone_id = aws_route53_zone.frontend_zone.zone_id
  name    = var.domain_name
  type    = "A"

  alias {
    name                   = var.cloudfront_domain_name
    zone_id                = var.cloudfront_distribution_id
    evaluate_target_health = false
  }
}

output "route53_zone_id" {
  value = aws_route53_zone.frontend_zone.zone_id
}

```
variables.tf:
```
variable "region" {
  description = "AWS region"
}

variable "domain_name" {
  description = "Domain name for the Route 53 record"
}

variable "cloudfront_domain_name" {
  description = "Domain name associated with the CloudFront distribution"
}

variable "cloudfront_distribution_id" {
  description = "ID of the CloudFront distribution"
}

```
outputs.tf:
```
output "route53_zone_id" {
  value = aws_route53_zone.frontend_zone.zone_id
}

```




5. Main Configuration:

Directory Structure:
```
frontend-deployment/
|-- main.tf
|-- variables.tf
|-- terraform.tfvars

```

main.tf:
```
provider "aws" {
  region = "us-east-1"
}

module "s3" {
  source = "./s3-frontend-module"

  region      = "us-east-1"
  bucket_name = "your-unique-frontend-bucket-name"
}

module "cloudfront" {
  source = "./cloudfront-frontend-module"

  region             = "us-east-1"
  domain_name        = "your-domain.com"
  origin_domain_name = module.s3.bucket_name
}

module "acm" {
  source = "./acm-module"

  region      = "us-east-1"
  domain_name = "your-domain.com"
}

module "route53" {
  source = "./route53-module"

  region                   = "us-east-1"
  domain_name              = "your-domain.com"
  cloudfront_domain_name   = module.cloudfront.cloudfront_domain_name
  cloudfront_distribution_id = module.cloudfront.cloudfront_distribution_id
}

```
variables.tf:

```
variable "region" {
  description = "AWS region"
}

```
terraform.tfvars:
```
region = "us-east-1"
```


Deploying:
Navigate to the frontend-deployment directory in your terminal.
Run terraform init to initialize the configuration.
Run terraform apply to apply the configuration.
Make sure to replace placeholder values with your actual values.
This example provides a modular approach to deploying a React frontend code with a custom domain on AWS using S3, CloudFront, ACM, and Route 53.
Adjust the configurations based on your specific project structure and requirements.
   
