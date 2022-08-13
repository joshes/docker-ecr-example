# Dockerfile CI example

Example CI flow for building and pushing docker images from GitHub actions.

| Branch / Tag     | Docker Tag |
|------------------|------------|
| develop          | latest-dev |
| main             | latest     |
| tag (e.g., 1.0)  | 1.0        |

## Bootstrap IAM

```sh
AWS_ACCOUNT="XXXXXXXXXX"
aws iam create-user --user-name github
cat << EOF > GitHubECRPublisherPolicy.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPush",
            "Effect": "Allow",
            "Action": [
                "ecr:CompleteLayerUpload",
                "ecr:GetAuthorizationToken",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:BatchCheckLayerAvailability",
                "ecr:PutImage",
                "sts:AssumeRole",
                "sts:TagSession"
            ],
            "Resource": "*"
        }
    ]
}
EOF
aws iam create-policy --policy-name GitHubECRPublisherPolicy --policy-document file://GitHubECRPublisherPolicy.json
aws iam attach-user-policy --user-name github --policy-arn "arn:aws:iam::${AWS_ACCOUNT}:policy/GitHubECRPublisherPolicy"
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
aws iam create-role --role-name GitHubECRPublisherRole --assume-role-policy-document file://GitHubTrustPolicy.json
aws iam attach-role-policy --role-name GitHubECRPublisherRole --policy-arn "arn:aws:iam::${AWS_ACCOUNT}:policy/GitHubECRPublisherPolicy"
aws iam list-attached-role-policies --role-name GitHubECRPublisherRole
aws iam create-access-key --user-name github
```

### Finalize the configuration.

- Configure the GitHub Action Secrets in your repo by setting `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables.
- Configure the defaults in `.github/actions/ci/action.yml` for your environment.

## Future work

- Make this work for monorepos.
