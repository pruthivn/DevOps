# Terraform Scenarios

Document Terraform infrastructure scenarios, IaC examples, and troubleshooting cases here.

## 1. How to deploy resources in azure and aws using single tf file?
A. By defining both the aws and azurerm providers within the same configuration file we can achieve this.

```hcl
# 1. Define required providers and versions
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

# 2. Configure the AWS Provider
provider "aws" {
  region = "us-east-1"
}

# 3. Configure the Azure Provider
provider "azurerm" {
  features {} # Required block for azurerm
}

# 4. Define AWS Infrastructure
resource "aws_s3_bucket" "multi_cloud_bucket" {
  bucket = "my-unique-multicloud-bucket-name"
}

# 5. Define Azure Infrastructure
resource "azurerm_resource_group" "multi_cloud_rg" {
  name     = "multicloud-resources"
  location = "East US"
}

resource "azurerm_storage_account" "multi_cloud_sa" {
  name                     = "myuniquemcstorageacct"
  resource_group_name      = azurerm_resource_group.multi_cloud_rg.name
  location                 = azurerm_resource_group.multi_cloud_rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

## 2. Why we are not defining location details in provider block like AWS for Azure?
A. The Azure provider is globally scoped, whereas the AWS provider is regionally scoped.

Because an AWS provider instance can only execute API requests within a single region, you must specify the region at the provider level.

The Azure Resource Manager (ARM) API uses a single global endpoint (://azure.com). When Terraform sends API requests to Azure, it communicates at the Subscription level, not the geographical level.

Since a single Azure subscription natively spans all worldwide locations, the provider itself is region-agnostic.

## 3. How to deploy AWS resources into 2 differnt regions using single .tf file?
A. Using alias and provider directive in resource block we achieve this.

In the below code any resource that does not feature a provider = ... argument will automatically Deploy to the Default provider configuration.

```hcl
# 1. Default Provider Configuration (Primary Region)
provider "aws" {
  region = "us-east-1"
}

# 2. Secondary Provider Configuration (Using an Alias)
provider "aws" {
  alias  = "west"
  region = "us-west-2"
}

# 3. Resource deployed to the default region (us-east-1)
resource "aws_s3_bucket" "primary_bucket" {
  bucket = "my-company-primary-bucket-us-east-1"

  tags = {
    Environment = "Production"
    Region      = "us-east-1"
  }
}

# 4. Resource deployed to the aliased region (us-west-2)
resource "aws_s3_bucket" "secondary_bucket" {
  provider = aws.west # Explicitly references the secondary provider
  bucket   = "my-company-backup-bucket-us-west-2"

  tags = {
    Environment = "Production"
    Region      = "us-west-2"
  }
}
```

## 4. How to deploy AWS resources into 2 different Accounts using single .tf file?

1. using Acces keys

``hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

### 1. Default Provider: Connects to Account A (e.g., Development)
provider "aws" {
  region  = "us-east-1"
  profile = "development" # Uses the 'development' credentials from ~/.aws/credentials
}

### 2. Aliased Provider: Connects to Account B (e.g., Production)
provider "aws" {
  alias   = "prod"        # The unique identifier for this account instance
  region  = "us-east-1"
  profile = "production"  # Uses the 'production' credentials from ~/.aws/credentials
}

### =========================================================================
### Deploying Resources
### =========================================================================

### This bucket deploys into Account A because no explicit provider is declared
resource "aws_s3_bucket" "dev_bucket" {
  bucket = "my-company-dev-data-bucket"
}

### This bucket deploys into Account B using the explicit provider alias
resource "aws_s3_bucket" "prod_bucket" {
  provider = aws.prod # Routes this resource to the 'prod' aliased provider
  bucket   = "my-company-prod-data-bucket"
}
```

2. CI/CD point of view.

```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# 1. Default Provider: Connects to Account A (e.g., Development)
provider "aws" {
  region  = "us-east-1"
  profile = "development" # Uses the 'development' credentials from ~/.aws/credentials
}

# 2. Aliased Provider: Connects to Account B (e.g., Production)
provider "aws" {
  alias   = "prod"        # The unique identifier for this account instance
  region  = "us-east-1"
  profile = "production"  # Uses the 'production' credentials from ~/.aws/credentials
}

# =========================================================================
# Deploying Resources
# =========================================================================

# This bucket deploys into Account A because no explicit provider is declared
resource "aws_s3_bucket" "dev_bucket" {
  bucket = "my-company-dev-data-bucket"
}

# This bucket deploys into Account B using the explicit provider alias
resource "aws_s3_bucket" "prod_bucket" {
  provider = aws.prod # Routes this resource to the 'prod' aliased provider
  bucket   = "my-company-prod-data-bucket"
}
```

## 5. if we have 2 .auto.tfvars files which file it consider?
A. It considers both files, but if they contain the exact same variable, the file that comes last alphabetically (lexically) wins.

Ex: 

File 1: a_variables.auto.tfvars (instance_type = "t3.micro")

File 2: z_variables.auto.tfvars (instance_type = "m5.large")

Result: Terraform will use "m5.large" because z comes after a.

Special Naming StrategiesTo control exactly which auto-file wins without renaming your actual logical structures, use a numeric prefix

EX: 

01-defaults.auto.tfvars (Loaded first, lowest priority)

02-overrides.auto.tfvars (Loaded second, overrides the first)
