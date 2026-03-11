1. Stream Management

CreateStream

aws kinesis create-stream \
  --stream-name test-stream \
  --shard-count 1
DescribeStream

aws kinesis describe-stream \
  --stream-name test-stream
DescribeStreamSummary

aws kinesis describe-stream-summary \
  --stream-name test-stream
ListStreams

aws kinesis list-streams
ListStreams (with pagination)

aws kinesis list-streams \
  --limit 10

aws kinesis list-streams \
  --limit 10 \
  --next-token "your-next-token"
UpdateShardCount (Scale Up)

aws kinesis update-shard-count \
  --stream-name test-stream \
  --target-shard-count 2 \
  --scaling-type UNIFORM_SCALING
UpdateShardCount (Scale Down)

aws kinesis update-shard-count \
  --stream-name test-stream \
  --target-shard-count 1 \
  --scaling-type UNIFORM_SCALING
UpdateStreamMode (On-Demand)

aws kinesis update-stream-mode \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --stream-mode-details StreamMode=ON_DEMAND
UpdateStreamMode (Provisioned)

aws kinesis update-stream-mode \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --stream-mode-details StreamMode=PROVISIONED
DeleteStream

aws kinesis delete-stream \
  --stream-name test-stream

# Force delete (with registered consumers)
aws kinesis delete-stream \
  --stream-name test-stream \
  --enforce-consumer-deletion
2. Data Operations (Put/Get Records)

PutRecord (Single)

aws kinesis put-record \
  --stream-name test-stream \
  --partition-key "partition-1" \
  --data "SGVsbG8gV29ybGQ="
PutRecord (with explicit hash key)

aws kinesis put-record \
  --stream-name test-stream \
  --partition-key "partition-1" \
  --explicit-hash-key "0" \
  --data "SGVsbG8gV29ybGQ="
PutRecord (piping data)

echo -n "Hello World" | base64 | xargs -I {} aws kinesis put-record \
  --stream-name test-stream \
  --partition-key "partition-1" \
  --data {}
PutRecords (Batch - Multiple Records)

aws kinesis put-records \
  --stream-name test-stream \
  --records \
    Data="SGVsbG8gMQ==",PartitionKey="partition-1" \
    Data="SGVsbG8gMg==",PartitionKey="partition-2" \
    Data="SGVsbG8gMw==",PartitionKey="partition-3"
PutRecords (Batch - from JSON file)

cat > /tmp/records.json << 'EOF'
{
  "StreamName": "test-stream",
  "Records": [
    {
      "Data": "dGVzdCByZWNvcmQgMQ==",
      "PartitionKey": "pk-1"
    },
    {
      "Data": "dGVzdCByZWNvcmQgMg==",
      "PartitionKey": "pk-2"
    },
    {
      "Data": "dGVzdCByZWNvcmQgMw==",
      "PartitionKey": "pk-3"
    }
  ]
}
EOF

aws kinesis put-records --cli-input-json file:///tmp/records.json
GetShardIterator (TRIM_HORIZON - oldest)

aws kinesis get-shard-iterator \
  --stream-name test-stream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type TRIM_HORIZON
GetShardIterator (LATEST - newest)

aws kinesis get-shard-iterator \
  --stream-name test-stream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type LATEST
GetShardIterator (AT_TIMESTAMP)

aws kinesis get-shard-iterator \
  --stream-name test-stream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type AT_TIMESTAMP \
  --timestamp "2024-01-15T00:00:00Z"
GetShardIterator (AT_SEQUENCE_NUMBER)

aws kinesis get-shard-iterator \
  --stream-name test-stream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type AT_SEQUENCE_NUMBER \
  --starting-sequence-number "49590338271490256608559692538361571095921575989136588898"
GetShardIterator (AFTER_SEQUENCE_NUMBER)

aws kinesis get-shard-iterator \
  --stream-name test-stream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type AFTER_SEQUENCE_NUMBER \
  --starting-sequence-number "49590338271490256608559692538361571095921575989136588898"
GetRecords

# First get the iterator
SHARD_ITERATOR=$(aws kinesis get-shard-iterator \
  --stream-name test-stream \
  --shard-id shardId-000000000000 \
  --shard-iterator-type TRIM_HORIZON \
  --query 'ShardIterator' \
  --output text)

# Then get records
aws kinesis get-records \
  --shard-iterator "$SHARD_ITERATOR"

# With limit
aws kinesis get-records \
  --shard-iterator "$SHARD_ITERATOR" \
  --limit 100
3. Shard Management

ListShards

aws kinesis list-shards \
  --stream-name test-stream
ListShards (with filter)

aws kinesis list-shards \
  --stream-name test-stream \
  --max-results 100

# Filter by timestamp
aws kinesis list-shards \
  --stream-name test-stream \
  --stream-creation-timestamp "2024-01-01T00:00:00Z"
ListShards (using ARN)

aws kinesis list-shards \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream
MergeShards

aws kinesis merge-shards \
  --stream-name test-stream \
  --shard-to-merge shardId-000000000000 \
  --adjacent-shard-to-merge shardId-000000000001
SplitShard

aws kinesis split-shard \
  --stream-name test-stream \
  --shard-to-split shardId-000000000000 \
  --new-starting-hash-key "170141183460469231731687303715884105728"
4. Enhanced Fan-Out (Consumers)

RegisterStreamConsumer

aws kinesis register-stream-consumer \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --consumer-name test-consumer
DescribeStreamConsumer

# By consumer ARN
aws kinesis describe-stream-consumer \
  --consumer-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream/consumer/test-consumer:1234567890

# By stream ARN + consumer name
aws kinesis describe-stream-consumer \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --consumer-name test-consumer
ListStreamConsumers

aws kinesis list-stream-consumers \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream

# With pagination
aws kinesis list-stream-consumers \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --max-results 10
SubscribeToShard (Note: this is a streaming API, limited CLI support)

aws kinesis subscribe-to-shard \
  --consumer-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream/consumer/test-consumer:1234567890 \
  --shard-id shardId-000000000000 \
  --starting-position Type=TRIM_HORIZON
DeregisterStreamConsumer

# By consumer ARN
aws kinesis deregister-stream-consumer \
  --consumer-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream/consumer/test-consumer:1234567890

# By stream ARN + consumer name
aws kinesis deregister-stream-consumer \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --consumer-name test-consumer
5. Encryption

StartStreamEncryption

# AWS managed key
aws kinesis start-stream-encryption \
  --stream-name test-stream \
  --encryption-type KMS \
  --key-id alias/aws/kinesis

# Customer managed key
aws kinesis start-stream-encryption \
  --stream-name test-stream \
  --encryption-type KMS \
  --key-id arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012
StopStreamEncryption

aws kinesis stop-stream-encryption \
  --stream-name test-stream \
  --encryption-type KMS \
  --key-id alias/aws/kinesis
6. Retention Period

IncreaseStreamRetentionPeriod

# Increase to 48 hours
aws kinesis increase-stream-retention-period \
  --stream-name test-stream \
  --retention-period-hours 48

# Increase to 7 days (168 hours) - max is 8760 (365 days)
aws kinesis increase-stream-retention-period \
  --stream-name test-stream \
  --retention-period-hours 168
DecreaseStreamRetentionPeriod

# Decrease back to default 24 hours
aws kinesis decrease-stream-retention-period \
  --stream-name test-stream \
  --retention-period-hours 24
7. Tagging

AddTagsToStream

aws kinesis add-tags-to-stream \
  --stream-name test-stream \
  --tags Environment=test,Project=logging,Owner=security-team

# Using ARN
aws kinesis add-tags-to-stream \
  --stream-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --tags Environment=test
ListTagsForStream

aws kinesis list-tags-for-stream \
  --stream-name test-stream

# With pagination
aws kinesis list-tags-for-stream \
  --stream-name test-stream \
  --limit 10
RemoveTagsFromStream

aws kinesis remove-tags-from-stream \
  --stream-name test-stream \
  --tag-keys Environment Project Owner
8. Resource Policies

PutResourcePolicy

cat > /tmp/resource-policy.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCrossAccountPut",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::987654321098:root"
      },
      "Action": [
        "kinesis:PutRecord",
        "kinesis:PutRecords"
      ],
      "Resource": "arn:aws:kinesis:us-east-1:123456789012:stream/test-stream"
    }
  ]
}
EOF

aws kinesis put-resource-policy \
  --resource-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream \
  --policy file:///tmp/resource-policy.json
GetResourcePolicy

aws kinesis get-resource-policy \
  --resource-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream
DeleteResourcePolicy

aws kinesis delete-resource-policy \
  --resource-arn arn:aws:kinesis:us-east-1:123456789012:stream/test-stream
9. Limits

DescribeLimits

aws kinesis describe-limits
Full End-to-End Test Script

#!/bin/bash
set -e

STREAM_NAME="scp-test-stream-$(date +%s)"
REGION="us-east-1"
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
STREAM_ARN="arn:aws:kinesis:${REGION}:${ACCOUNT_ID}:stream/${STREAM_NAME}"

echo "============================================"
echo "Kinesis CLI Complete Test Suite"
echo "Stream: ${STREAM_NAME}"
echo "Account: ${ACCOUNT_ID}"
echo "Region: ${REGION}"
echo "============================================"

# --- 1. STREAM MANAGEMENT ---
echo -e "\n[1/30] CreateStream"
aws kinesis create-stream --stream-name ${STREAM_NAME} --shard-count 2 && echo "✅ PASS" || echo "❌ FAIL"

echo "Waiting for stream to become ACTIVE..."
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

echo -e "\n[2/30] DescribeStream"
aws kinesis describe-stream --stream-name ${STREAM_NAME} && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[3/30] DescribeStreamSummary"
aws kinesis describe-stream-summary --stream-name ${STREAM_NAME} && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[4/30] ListStreams"
aws kinesis list-streams && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[5/30] DescribeLimits"
aws kinesis describe-limits && echo "✅ PASS" || echo "❌ FAIL"

# --- 2. TAGGING ---
echo -e "\n[6/30] AddTagsToStream"
aws kinesis add-tags-to-stream --stream-name ${STREAM_NAME} --tags Environment=test,TestRun=true && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[7/30] ListTagsForStream"
aws kinesis list-tags-for-stream --stream-name ${STREAM_NAME} && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[8/30] RemoveTagsFromStream"
aws kinesis remove-tags-from-stream --stream-name ${STREAM_NAME} --tag-keys TestRun && echo "✅ PASS" || echo "❌ FAIL"

# --- 3. ENCRYPTION ---
echo -e "\n[9/30] StartStreamEncryption"
aws kinesis start-stream-encryption --stream-name ${STREAM_NAME} --encryption-type KMS --key-id alias/aws/kinesis && echo "✅ PASS" || echo "❌ FAIL"

echo "Waiting for stream to become ACTIVE after encryption..."
sleep 15
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

echo -e "\n[10/30] StopStreamEncryption"
aws kinesis stop-stream-encryption --stream-name ${STREAM_NAME} --encryption-type KMS --key-id alias/aws/kinesis && echo "✅ PASS" || echo "❌ FAIL"

echo "Waiting for stream to become ACTIVE after stopping encryption..."
sleep 15
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

# --- 4. RETENTION ---
echo -e "\n[11/30] IncreaseStreamRetentionPeriod"
aws kinesis increase-stream-retention-period --stream-name ${STREAM_NAME} --retention-period-hours 48 && echo "✅ PASS" || echo "❌ FAIL"

sleep 10

echo -e "\n[12/30] DecreaseStreamRetentionPeriod"
aws kinesis decrease-stream-retention-period --stream-name ${STREAM_NAME} --retention-period-hours 24 && echo "✅ PASS" || echo "❌ FAIL"

sleep 10

# --- 5. DATA OPERATIONS ---
echo -e "\n[13/30] PutRecord"
aws kinesis put-record \
  --stream-name ${STREAM_NAME} \
  --partition-key "test-pk-1" \
  --data "dGVzdCByZWNvcmQgMQ==" && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[14/30] PutRecords (batch)"
aws kinesis put-records \
  --stream-name ${STREAM_NAME} \
  --records \
    Data="cmVjb3JkIDE=",PartitionKey="pk-1" \
    Data="cmVjb3JkIDI=",PartitionKey="pk-2" \
    Data="cmVjb3JkIDM=",PartitionKey="pk-3" && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[15/30] ListShards"
SHARDS=$(aws kinesis list-shards --stream-name ${STREAM_NAME})
echo "$SHARDS" && echo "✅ PASS" || echo "❌ FAIL"

SHARD_ID=$(echo "$SHARDS" | jq -r '.Shards[0].ShardId')
echo "Using Shard: ${SHARD_ID}"

echo -e "\n[16/30] GetShardIterator (TRIM_HORIZON)"
SHARD_ITERATOR=$(aws kinesis get-shard-iterator \
  --stream-name ${STREAM_NAME} \
  --shard-id ${SHARD_ID} \
  --shard-iterator-type TRIM_HORIZON \
  --query 'ShardIterator' \
  --output text)
echo "✅ PASS - Got iterator"

echo -e "\n[17/30] GetRecords"
aws kinesis get-records --shard-iterator "${SHARD_ITERATOR}" --limit 10 && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[18/30] GetShardIterator (LATEST)"
aws kinesis get-shard-iterator \
  --stream-name ${STREAM_NAME} \
  --shard-id ${SHARD_ID} \
  --shard-iterator-type LATEST && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[19/30] GetShardIterator (AT_TIMESTAMP)"
aws kinesis get-shard-iterator \
  --stream-name ${STREAM_NAME} \
  --shard-id ${SHARD_ID} \
  --shard-iterator-type AT_TIMESTAMP \
  --timestamp "$(date -u +%Y-%m-%dT%H:%M:%SZ)" && echo "✅ PASS" || echo "❌ FAIL"

# --- 6. STREAM MODE ---
echo -e "\n[20/30] UpdateStreamMode (ON_DEMAND)"
aws kinesis update-stream-mode \
  --stream-arn ${STREAM_ARN} \
  --stream-mode-details StreamMode=ON_DEMAND && echo "✅ PASS" || echo "❌ FAIL"

echo "Waiting for stream mode update..."
sleep 20
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

echo -e "\n[21/30] UpdateStreamMode (PROVISIONED)"
aws kinesis update-stream-mode \
  --stream-arn ${STREAM_ARN} \
  --stream-mode-details StreamMode=PROVISIONED && echo "✅ PASS" || echo "❌ FAIL"

echo "Waiting for stream mode update..."
sleep 20
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

# --- 7. SHARD OPERATIONS ---
echo -e "\n[22/30] SplitShard"
aws kinesis split-shard \
  --stream-name ${STREAM_NAME} \
  --shard-to-split ${SHARD_ID} \
  --new-starting-hash-key "170141183460469231731687303715884105728" && echo "✅ PASS" || echo "❌ FAIL"

echo "Waiting for split to complete..."
sleep 20
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

echo -e "\n[23/30] ListShards (after split)"
NEW_SHARDS=$(aws kinesis list-shards --stream-name ${STREAM_NAME} \
  --shard-filter Type=FROM_TRIM_HORIZON)
echo "$NEW_SHARDS"

# Get two adjacent open shards for merge
OPEN_SHARDS=$(echo "$NEW_SHARDS" | jq -r '[.Shards[] | select(.SequenceNumberRange.EndingSequenceNumber == null)] | .[0:2] | .[].ShardId')
MERGE_SHARD_1=$(echo "$OPEN_SHARDS" | head -1)
MERGE_SHARD_2=$(echo "$OPEN_SHARDS" | tail -1)

echo -e "\n[24/30] MergeShards"
if [ -n "$MERGE_SHARD_1" ] && [ -n "$MERGE_SHARD_2" ]; then
  aws kinesis merge-shards \
    --stream-name ${STREAM_NAME} \
    --shard-to-merge ${MERGE_SHARD_1} \
    --adjacent-shard-to-merge ${MERGE_SHARD_2} && echo "✅ PASS" || echo "❌ FAIL"
  sleep 20
  aws kinesis wait stream-exists --stream-name ${STREAM_NAME}
else
  echo "⏭️  SKIP - Could not identify adjacent shards"
fi

# --- 8. ENHANCED FAN-OUT ---
CONSUMER_NAME="test-consumer-$(date +%s)"

echo -e "\n[25/30] RegisterStreamConsumer"
CONSUMER_RESULT=$(aws kinesis register-stream-consumer \
  --stream-arn ${STREAM_ARN} \
  --consumer-name ${CONSUMER_NAME})
echo "$CONSUMER_RESULT" && echo "✅ PASS" || echo "❌ FAIL"

CONSUMER_ARN=$(echo "$CONSUMER_RESULT" | jq -r '.Consumer.ConsumerARN')
echo "Consumer ARN: ${CONSUMER_ARN}"

echo "Waiting for consumer to become ACTIVE..."
sleep 15

echo -e "\n[26/30] DescribeStreamConsumer"
aws kinesis describe-stream-consumer \
  --consumer-arn ${CONSUMER_ARN} && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[27/30] ListStreamConsumers"
aws kinesis list-stream-consumers \
  --stream-arn ${STREAM_ARN} && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[28/30] DeregisterStreamConsumer"
aws kinesis deregister-stream-consumer \
  --consumer-arn ${CONSUMER_ARN} && echo "✅ PASS" || echo "❌ FAIL"

# --- 9. RESOURCE POLICY ---
echo -e "\n[29/30] PutResourcePolicy"
POLICY="{"Version":"2012-10-17","Statement":[{"Sid":"TestPolicy","Effect":"Allow","Principal":{"AWS":"arn:aws:iam::${ACCOUNT_ID}:root"},"Action":["kinesis:DescribeStream"],"Resource":"${STREAM_ARN}"}]}"

aws kinesis put-resource-policy \
  --resource-arn ${STREAM_ARN} \
  --policy "${POLICY}" && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[29b/30] GetResourcePolicy"
aws kinesis get-resource-policy \
  --resource-arn ${STREAM_ARN} && echo "✅ PASS" || echo "❌ FAIL"

echo -e "\n[29c/30] DeleteResourcePolicy"
aws kinesis delete-resource-policy \
  --resource-arn ${STREAM_ARN} && echo "✅ PASS" || echo "❌ FAIL"

# --- 10. UPDATE SHARD COUNT ---
echo -e "\n[29d/30] UpdateShardCount"
aws kinesis update-shard-count \
  --stream-name ${STREAM_NAME} \
  --target-shard-count 2 \
  --scaling-type UNIFORM_SCALING && echo "✅ PASS" || echo "❌ FAIL"

sleep 20
aws kinesis wait stream-exists --stream-name ${STREAM_NAME}

# --- 11. CLEANUP ---
echo -e "\n[30/30] DeleteStream"
aws kinesis delete-stream \
  --stream-name ${STREAM_NAME} \
  --enforce-consumer-deletion && echo "✅ PASS" || echo "❌ FAIL"

echo ""
echo "============================================"
echo "Test Suite Complete!"
echo "Stream ${STREAM_NAME} has been deleted."
echo "============================================"
