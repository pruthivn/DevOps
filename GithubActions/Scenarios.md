# GitHub Actions Scenarios

## i want to create a S3 bucket with Github actions but i don't want ot hardcode the bucket name and i need to give the bucket name while running the pipline?
A. we can achieve this using workflow_dispatch(we manually need to trigger the pipeline).

```yaml
name: Create S3 Buckets

on:
  workflow_dispatch:
    inputs:
      bucket_name:
        description: 'Enter Name of the bucket you want to create'
        required: true # defintely provide the bucket name
        type: string
permissions:
  id-token: write
  contents: read
jobs:
  s3-create:
    runs-on: ubuntu-latest
    steps:
      - name: configure aws credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v6.1.0
        with:
          aws-region: ap-south-1
          role-to-assume: arn:aws:iam::655700896650:role/GHA-Role
      - name: Create S3 bucket
        run: |                              
          aws s3 mb s3://${{ github.event.inputs.bucket_name }} --region ap-south-1 
          aws s3 ls | grep ${{ github.event.inputs.bucket_name }}
          aws s3 ls
```

![alt text](image-3.png)