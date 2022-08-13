# Dockerfile CI example

Example CI flow for building and pushing docker images from GitHub actions.

## Bootstrap IAM

```sh
AWS_ACCOUNT="your-account"
aws iam create-user --user-name github
cat << EOF > GitHubECRPolicy.json
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "AllowPushPull",
          "Effect": "Allow",
          "Action": [
              "ecr:GetAuthorizationToken",
              "ecr:BatchGetImage",
              "ecr:BatchCheckLayerAvailability",
              "ecr:CompleteLayerUpload",
              "ecr:GetDownloadUrlForLayer",
              "ecr:InitiateLayerUpload",
              "ecr:PutImage",
              "ecr:UploadLayerPart",
              "sts:AssumeRole",
              "sts:TagSession"
          ],
          "Resource": "*"
      }
  ]
}
EOF
aws iam create-policy --policy-name GitHubECRPolicy --policy-document file://GitHubECRPolicy.json
aws iam attach-user-policy --user-name github --policy-arn "arn:aws:iam::${AWS_ACCOUNT}:policy/GitHubECRPolicy"
aws iam list-attached-user-policies --user-name github
cat << EOF > GitHubTrustPolicy.json
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Principal": { "AWS": "arn:aws:iam::${AWS_ACCOUNT}:user/github" },
        "Action": "sts:AssumeRole"
    }
}
EOF
aws iam create-role --role-name GitHubECRActionRole --assume-role-policy-document file://GitHubTrustPolicy.json
aws iam attach-role-policy --role-name GitHubECRActionRole --policy-arn "arn:aws:iam::${AWS_ACCOUNT}:policy/GitHubECRPolicy"
aws iam list-attached-role-policies --role-name GitHubECRActionRole
aws iam create-access-key --user-name github
```

Configure the GitHub Action Secrets in your repo by setting `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables.
