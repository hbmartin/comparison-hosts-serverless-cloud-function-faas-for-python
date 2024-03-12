# Comparing hosts / providers for serverless cloud functions (FaaS) for Python

TODO: Slack/ Discord / Twitter / Mastodon badges

No Python support: Cloudflare Workers, Netlify Edge Functions, StackPath EdgeEngine. IBM Cloud Functions are deprecated.

This document provides an up-to-date comparison between hosted, serverless (no cost or management to spin down to zero) providers of cloud function hosts with Python runtimes.
Note the distinction between edge providers (execution at PoP) and non-edge (typically predetermined DS region).

[TOC]

## DevEx

|                                       | Python Version | Status    | API Framework                   | requirements.txt | Local Testing | Docs | Basic Example                                                |
| ------------------------------------- | -------------- | --------- | ------------------------------- | ---------------- | ------------- | ---- | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**    | 3.10           | GA        | Plain obj.                      | Vendor in zip    | ❔             | 🎉    | https://www.alibabacloud.com/help/en/functioncompute/latest/event-handlers |
| **AWS Lambda and Lambda@Edge**        | 3.12           | GA        | Plain obj.                      | Vendor in zip    | ✅             | 👍    | https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/blank-python |
| **Azure Functions**                   | 3.11           | GA        | azure-functions                 | ✅                | ✅             | 🎉    | https://learn.microsoft.com/en-us/samples/browse/?products=azure-functions&languages=python |
| **Fermyon** (WASM compiled)           | 3.11           | No-native | Spin                            | ❔                |               | 👍    |                                                              |
| **Fly.io** (microVM)                  | *              | GA        | *                               | ✅                |               | 👍    | https://fly.io/docs/languages-and-frameworks/python/         |
| **Google / Firebase Cloud Functions** | 3.12           | GA        | Flask                           | ✅                | ✅             | 🎉    |                                                              |
| **IBM Code Engine**                   | 3.11           | GA        | Plain obj.                      | ✅                | ✅             | 👍    | https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/function-codebundle-python |
| **Oracle (OCI) Functions**            | 3.11           | GA        | FDK                             | ✅                | ❔             | Min. | https://github.com/oracle-samples/oracle-functions-samples/tree/master/samples/helloworld |
| **Tencent Cloud Functions**           | 3.6            | GA        | SCF                             | Vendor in zip    | ❔             | Min. | https://www.tencentcloud.com/document/product/583/40327      |
| **Vercel Functions**                  | 3.9            | Beta      | HTTP handler or WSGI / ASGI app | ✅                | ❔             | Min. | https://vercel.com/templates/python/python-hello-world       |



## Pricing

|                                    | **Free Plan** | Bill Limits | **First Paid Tier** | Second Paid Tier |
| ---------------------------------- | ------------- | ----------- | ------------------- | ---------------- |
| **Alibaba Cloud Function Compute** | ?             |             | ?                   |                  |
| **AWS Lambda**                     | 128           |             | 3008                |                  |
| **AWS Lambda@Edge**                | 128           |             | 128                 |                  |
| **Azure Functions**                | NA            |             | 1536                |                  |
| **Fermyon**                        |               |             |                     |                  |
| **Fly.io**                         | NA            |             | NA                  |                  |
| **Google Cloud Functions**         | 128           |             | 2048                |                  |
| **IBM Code Engine**                | 256           |             | 2048                |                  |
| **Oracle (OCI) Functions**         | 128           |             | 1024                |                  |
| **Tencent Cloud Functions**        |               |             |                     |                  |
| **Vercel Functions**               | 1024          |             | 3008                |                  |



## Runtime Limits

|                             | Memory (MB) |         | Execution Time  |                 | **Payloads (MB)** |              | Keep Alive | Scale Limits |
| --------------------------- | ----------- | ------- | --------------- | --------------- | ----------------- | ------------ | ---------- | ------------ |
|                             | **Default** | **Max** | **Default**     | **Max**         | **Request**       | **Response** | (Mins)     |              |
| **Alibaba Functions**       | ?           | ?       | ?               | 600s            | 61                | 61           |            |              |
| **AWS Lambda**              | 128         | 3008    | 3s              | 15m             | 61                | 61           |            |              |
| **AWS Lambda@Edge**         | 128         | 128     | 30s             | 5s              | 50Mb              | 40 KB        |            |              |
| **Azure Functions**         | NA          | 1536    | 5m              | 10m             | No limit          | No limit     |            |              |
| **Fermyon**                 |             |         |                 |                 |                   |              |            |              |
| **Fly.io**                  | NA          | NA      | No limit        | No limit        | No limit          | No limit     |            |              |
| **Google cloud Functions**  | 128         | 2048    | NA              | 540s            | 10                | 10           |            |              |
| **IBM Code Engine**         | 256         | 2048    | 1m              | 10m             | 5                 | 5            |            |              |
| **Oracle (OCI) Functions**  | 128         | 1024    | 30s             | 120s            | 6                 | 6            |            |              |
| **Tencent Cloud Functions** |             |         |                 |                 |                   |              |            |              |
| **Vercel Functions**        | 1024        | 3008    | 10s (Free plan) | 300s (Pro plan) | 5                 | 5            |            |              |



## Other Platform Products

|                                       | SQL DB | No SQL DB | Blob Store | File Hosting | GPU  | Auth |
| ------------------------------------- | ------ | --------- | ---------- | ------------ | ---- | ---- |
| **Alibaba Cloud Function Compute**    |        |           |            |              |      |      |
| **AWS Lambda and Lambda@Edge**        |        |           |            |              |      |      |
| **Azure Functions**                   |        |           |            |              |      |      |
| **Fermyon**                           |        |           |            |              |      |      |
| **Fly.io**                            |        |           |            |              |      |      |
| **Google / Firebase Cloud Functions** |        |           |            |              |      |      |
| **IBM Code Engine**                   |        |           |            |              |      |      |
| **Oracle (OCI) Functions**            |        |           |            |              |      |      |
| **Vercel Functions**                  |        |           |            |              |      |      |



## Performance (median times)

TODO: replicating for Python: https://github.com/serverless-benchmark/backend

|                                    | PoPs (if edge) or regions | Uptime | Cold Response (ms) | Warm Response (ms) | Overhead (ms) |
| ---------------------------------- | ------------------------- | ------ | ------------------ | ------------------ | ------------- |
| **Alibaba Cloud Function Compute** |                           |        |                    |                    |               |
| **AWS Lambda**                     |                           |        |                    |                    |               |
| **AWS Lambda@Edge**                |                           |        |                    |                    |               |
| **Azure Functions**                |                           |        |                    |                    |               |
| **Fermyon**                        |                           |        |                    |                    |               |
| **Fly.io**                         |                           |        |                    |                    |               |
| **Google Cloud Functions**         |                           |        |                    |                    |               |
| **IBM Code Engine**                |                           |        |                    |                    |               |
| **Oracle (OCI) Functions**         |                           |        |                    |                    |               |
| **Tencent Cloud Functions**        |                           |        |                    |                    |               |
| **Vercel Functions**               |                           |        |                    |                    |               |



## Security Considerations

TODO: e.g. compliance certifications, data encryption, and network security options



## Discussions, Community, and Support

|                                       | Our Wiki | Forum | Reddit |
| ------------------------------------- | -------- | ----- | ------ |
| **Alibaba Cloud Function Compute**    |          |       |        |
| **AWS Lambda and Lambda@Edge**        |          |       |        |
| **Azure Functions**                   |          |       |        |
| **Fermyon**                           |          |       |        |
| **Fly.io**                            |          |       |        |
| **Google / Firebase Cloud Functions** |          |       |        |
| **IBM Code Engine**                   |          |       |        |
| **Oracle (OCI) Functions**            |          |       |        |
| **Vercel Functions**                  |          |       |        |



## References and Useful Links

- https://fauna.com/blog/comparison-faas-providers#pricing
