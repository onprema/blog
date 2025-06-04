# iam: aws vs gcp

*comparing iam concepts in aws vs gcp*

## Core Concepts

| AWS Concept | GCP Concept | Purpose |
|-------------|-------------|---------|
| **IAM Role** | **Service Account** | Identity that services can assume |
| **IAM Policy** | **IAM Role** (confusing, I know!) | Set of permissions |
| **Principal** | **Member** | Who gets the permissions |
| **Role ARN** | **Service Account Email** | How to reference the identity |

## The Confusing Part: "Role" Means Different Things!

### AWS Pattern:
```hcl
# 1. Create IAM Role (identity)
resource "aws_iam_role" "lambda_role" {
  name = "lambda-execution-role"
  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole"
      Principal = { Service = "lambda.amazonaws.com" }
    }]
  })
}

# 2. Create/attach policies (permissions)
resource "aws_iam_role_policy_attachment" "lambda_policy" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = "arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole"
}

# 3. Use the role
resource "aws_lambda_function" "my_function" {
  role = aws_iam_role.lambda_role.arn  # Reference by ARN
}
```

### GCP Pattern:
```hcl
# 1. Create Service Account (identity)
resource "google_service_account" "cloud_run_sa" {
  account_id = "cloud-run-service"
}

# 2. Grant IAM role (permissions) to member (identity)
resource "google_project_iam_member" "cloud_run_permissions" {
  project = var.project_id
  role    = "roles/pubsub.subscriber"  # This is the permission set
  member  = "serviceAccount:${google_service_account.cloud_run_sa.email}"  # This is the identity
}

# 3. Use the service account
resource "google_cloud_run_v2_service" "my_service" {
  template {
    service_account = google_service_account.cloud_run_sa.email  # Reference by email
  }
}
```

## Key Differences

### AWS:
- **IAM Role** = Identity (what assumes permissions)
- **IAM Policy** = Permissions (what actions are allowed)
- **Attach** policies to roles
- Reference roles by **ARN**

### GCP:
- **Service Account** = Identity (what has permissions)
- **IAM Role** = Permissions (predefined sets like "Pub/Sub Subscriber")
- **Grant** roles to members
- Reference service accounts by **email address**

## The "Member" Field Explained

In GCP, the `member` field specifies WHO gets the permissions:

```hcl
member = "serviceAccount:my-service@project.iam.gserviceaccount.com"  # Service account
member = "user:john@company.com"                                     # Individual user
member = "group:developers@company.com"                              # Group of users
member = "domain:company.com"                                        # Entire domain
```

Think of it as:
- **AWS**: "Attach this policy to this role"
- **GCP**: "Grant this role to this member"
