# Kinesis Data Streams — CMK & SCP Demo

## Structure

```
Kinesis/
├── cloudformation/
│   ├── kinesis-cmk-demo.yaml       # Kinesis stream + CMK + producer/consumer IAM roles
│   └── kinesis-cwlogs-demo.yaml    # CloudWatch Logs → Kinesis → Lambda pipeline
└── scps/
    ├── scp-kinesis-deny-stop-encryption-fis.json   # Deny StopStreamEncryption + FIS
    ├── scp-kinesis-deny-cross-ou.json              # Deny cross-OU data plane access
    └── scp-kinesis-deny-prod-nonprod.json          # Deny prod <-> nonprod data sharing
```

## Deploy

### Prerequisites
- AWS CLI configured with SSO (`aws sso login --profile Work`)
- Permissions: CloudFormation, Kinesis, KMS, IAM, CloudWatch Logs, Lambda

### Stack 1 — Kinesis stream + CMK
```bash
aws cloudformation deploy \
  --template-file cloudformation/kinesis-cmk-demo.yaml \
  --stack-name kinesis-scp-demo \
  --parameter-overrides StreamName=scp-demo-stream Environment=nonprod \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile Work --region ap-southeast-2
```

### Stack 2 — CloudWatch Logs → Kinesis → Lambda pipeline
```bash
aws cloudformation deploy \
  --template-file cloudformation/kinesis-cwlogs-demo.yaml \
  --stack-name kinesis-cwlogs-demo \
  --parameter-overrides KinesisStackName=kinesis-scp-demo LogGroupName=/demo/kinesis-test \
  --capabilities CAPABILITY_NAMED_IAM \
  --profile Work --region ap-southeast-2
```

## Test the pipeline

```bash
# 1 — Send test log events
TIMESTAMP=$(date +%s000)
aws logs put-log-events \
  --log-group-name /demo/kinesis-test \
  --log-stream-name test-stream-1 \
  --log-events "[{\"timestamp\": $TIMESTAMP, \"message\": \"{\\\"service\\\":\\\"demo\\\",\\\"level\\\":\\\"INFO\\\",\\\"msg\\\":\\\"hello kinesis\\\"}\"}]" \
  --profile Work --region ap-southeast-2

# 2 — Watch Lambda decode the records
aws logs tail /aws/lambda/kinesis-log-consumer --follow \
  --profile Work --region ap-southeast-2

# 3 — Verify CMK encryption
aws kinesis describe-stream-summary \
  --stream-name scp-demo-stream \
  --query 'StreamDescriptionSummary.{Encryption:EncryptionType,KeyId:KeyId}' \
  --profile Work --region ap-southeast-2

# 4 — Test SCP deny (should fail once SCP is applied)
aws kinesis stop-stream-encryption \
  --stream-name scp-demo-stream \
  --encryption-type KMS \
  --key-id alias/scp-demo-stream-cmk \
  --profile Work --region ap-southeast-2
```

## SCPs

| File | Attach To | Purpose |
|---|---|---|
| `scp-kinesis-deny-stop-encryption-fis.json` | Root OU | Unconditionally deny disabling encryption and FIS |
| `scp-kinesis-deny-cross-ou.json` | Each OU | Deny cross-OU Kinesis data plane access |
| `scp-kinesis-deny-prod-nonprod.json` | Root OU | Deny prod <-> nonprod data sharing |

> **Note:** Replace all `<orgid>`, `<rootid>`, `<prod-ou-id>`, `<nonprod-ou-id>` placeholders in the SCP files with your actual AWS Organizations values before applying.
# Kinesis
