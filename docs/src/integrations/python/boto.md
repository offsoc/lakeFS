---
title: Boto3 (S3 Gateway)
description: Use Boto3 to interact with your objects on lakeFS through the S3 Gateway
---

# Using Boto3

!!! info
    To use Boto3 with lakeFS alongside S3, check out [Boto S3 Router](https://github.com/treeverse/boto-s3-router){:target="_blank"}. It will route
    requests to either S3 or lakeFS according to the provided bucket name.

lakeFS exposes an S3-compatible API, so you can use Boto3 to interact with your objects on lakeFS.

## Initializing

Create a Boto3 S3 client with your lakeFS endpoint and key-pair:

```python
import boto3
s3 = boto3.client('s3',
    endpoint_url='https://lakefs.example.com',
    aws_access_key_id='AKIAIOSFODNN7EXAMPLE',
    aws_secret_access_key='wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY')
```

The client is now configured to operate on your lakeFS installation.

### Configuring Boto3 S3 Client with Checksum Settings

In newer versions of Boto3, when connecting to **lakeFS** using **HTTPS**,
you might encounter an **AccessDenied** error on upload,
while the lakeFS logs display an error `encoding/hex: invalid byte: U+0053 'S'`.

To avoid this issue, explicitly configure the Boto3 client with the following checksum settings:

- `request_checksum_calculation: 'when_required'`
- `response_checksum_validation: 'when_required'`

Example of how to configure it:

```python
import boto3
from botocore.config import Config

# Configure checksum settings
config = Config(
    request_checksum_calculation='when_required',
    response_checksum_validation='when_required'
)

s3_client = boto3.client(
    's3',
    endpoint_url='https://lakefs.example.com',
    aws_access_key_id='AKIAIOSFODNN7EXAMPLE',
    aws_secret_access_key='wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY',
    config=config,
)
```

## Usage Examples

### Put an object into lakeFS

Use a branch name and a path to put an object in lakeFS:

```python
with open('/local/path/to/file_0', 'rb') as f:
    s3.put_object(Body=f, Bucket='example-repo', Key='main/example-file.parquet')
```

You can now commit this change using the lakeFS UI or CLI.

### List objects

List the branch objects starting with a prefix:

```python
list_resp = s3.list_objects_v2(Bucket='example-repo', Prefix='main/example-prefix')
for obj in list_resp['Contents']:
    print(obj['Key'])
```

Or, use a lakeFS commit ID to list objects for a specific commit:

```python
list_resp = s3.list_objects_v2(Bucket='example-repo', Prefix='c7a632d74f/example-prefix')
for obj in list_resp['Contents']:
    print(obj['Key'])
```

### Get object metadata

Get object metadata using branch and path:

```python
s3.head_object(Bucket='example-repo', Key='main/example-file.parquet')
# output:
# {'ResponseMetadata': {'RequestId': '72A9EBD1210E90FA',
#  'HostId': '',
#  'HTTPStatusCode': 200,
#  'HTTPHeaders': {'accept-ranges': 'bytes',
#   'content-length': '1024',
#   'etag': '"2398bc5880e535c61f7624ad6f138d62"',
#   'last-modified': 'Sun, 24 May 2020 10:42:24 GMT',
#   'x-amz-request-id': '72A9EBD1210E90FA',
#   'date': 'Sun, 24 May 2020 10:45:42 GMT'},
#  'RetryAttempts': 0},
# 'AcceptRanges': 'bytes',
# 'LastModified': datetime.datetime(2020, 5, 24, 10, 42, 24, tzinfo=tzutc()),
# 'ContentLength': 1024,
# 'ETag': '"2398bc5880e535c61f7624ad6f138d62"',
# 'Metadata': {}}
```
