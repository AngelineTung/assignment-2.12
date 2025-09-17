# assignment-2.12

[README.md](https://github.com/user-attachments/files/22385633/README.md)
# S3 → Lambda permissions: execution role vs. resource-based policy

## 1) What is the purpose of the **execution role** on the Lambda function?
The execution role is an **IAM role that the Lambda function assumes while it runs**. It defines what the function is allowed to do *outbound* against AWS services and resources. Typical examples include:
- Writing logs to **CloudWatch Logs**.
- Reading/writing objects in **Amazon S3**.
- Accessing other services (DynamoDB, SQS, SNS, KMS, etc.) as needed.

> In short: the execution role controls the function’s **actions** on other resources.

---

## 2) What is the purpose of the **resource‑based policy** on the Lambda function?
A Lambda resource‑based policy is **attached to the function itself** and controls **who/what is allowed to invoke the function**. For an S3 trigger, the policy includes a statement allowing the **`s3.amazonaws.com`** service (scoped to your bucket via `SourceArn`/`SourceAccount`) to **invoke** the function.

> In short: the resource policy controls **who can call the function**.

---

## 3) If the function must **upload a file into an S3 bucket**, what needs to change?

### 3a) Needed update on the **execution role** (no actual JSON policy required)
Grant the Lambda execution role **least‑privilege S3 write access** to the destination bucket/prefix, for example:
- `s3:PutObject` on `arn:aws:s3:::<target-bucket>/<desired-prefix>/*`  
  (Optionally `s3:AbortMultipartUpload`, and `s3:PutObjectTagging` if used.)
- If the bucket uses **SSE‑KMS**, also allow the KMS key for `kms:Encrypt`, `kms:GenerateDataKey*`.

> You might also include `s3:GetObject` if the function needs to read back what it wrote, and `s3:ListBucket` **scoped by prefix** if it must list keys.

### 3b) New **resource‑based policy** needed on the Lambda?
**No new resource policy is required** just to upload to S3. Uploading is an **outbound** action governed by the **execution role**.  
Keep (or add) the existing **S3 invoke** permission only if S3 will continue to **trigger** the function. Add additional invoke permissions only if a **new invoker** (e.g., API Gateway, EventBridge, another account) must call the function.

---

## TL;DR
- **Execution role** = what the function can *do to other services* (e.g., `s3:PutObject` to write to a bucket).
- **Resource policy** = who can *invoke the function* (e.g., S3 trigger permission).  
- To enable uploads to S3: **update the execution role with S3 write permissions** to the specific bucket/prefix. **No new resource policy** is needed unless a new invoker will call the function.
