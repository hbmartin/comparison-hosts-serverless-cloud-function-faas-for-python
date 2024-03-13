# Comparing hosts / providers for serverless cloud functions (FaaS) for Python

[![discord](https://img.shields.io/discord/267624335836053506?logo=discord&label=&color=323338)](https://discord.gg/python)
[![twitter](https://img.shields.io/badge/@hmartin-00aced.svg?logo=twitter&logoColor=black)](https://twitter.com/hmartin)

No Python support: Cloudflare Workers, Netlify Edge Functions, StackPath EdgeEngine. IBM Cloud Functions are deprecated.

This document provides an up-to-date comparison between hosted, serverless (no cost or management to spin down to zero) providers of cloud function hosts with Python runtimes.
Note the distinction between edge providers (execution at PoP) and non-edge (typically predetermined DS region).

[TOC]

## DevEx

|                                       | Python Version | Status    | API Framework                   | requirements.txt | Local Testing | Docs | Hello  World                                   |
| ------------------------------------- | -------------- | --------- | ------------------------------- | ---------------- | ------------- | ---- | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**    | 3.10           | GA        | Plain object                   | Vendor in zip    | ❔             | 🎉    | [Link](https://www.alibabacloud.com/help/en/functioncompute/latest/event-handlers) |
| **AWS Lambda & Lambda@Edge**        | 3.12           | GA        | Plain object                | Vendor in zip    | ✅             | 👍    | [Link](https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/blank-python) |
| **Azure Functions**                   | 3.11           | GA        | azure functions                | ✅                | ✅             | 🎉    | [Link](https://learn.microsoft.com/en-us/samples/browse/?products=azure-functions&languages=python) |
| **Fermyon** (WASM)           | 3.11           | No-native | Spin                            | ❔                | ❔ | 👍    | [Link](https://www.fermyon.com/blog/spin-python-sdk) |
| **Fly.io** (microVM)                  | Any              | GA        | Any                               | ✅                | ❔ | 👍    | [Link](https://fly.io/docs/languages-and-frameworks/python/) |
| **Google / Firebase Cloud Functions** | 3.12           | GA        | Flask                           | ✅                | ✅             | 🎉    |                                                              |
| **IBM Code Engine**                   | 3.11           | GA        | Plain object                | ✅                | ✅             | 👍    | [Link](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/function-codebundle-python) |
| **Oracle (OCI) Functions**            | 3.11           | GA        | FDK                             | ✅                | ❔             | Min. | [Link](https://github.com/oracle-samples/oracle-functions-samples/tree/master/samples/helloworld) |
| **Tencent Cloud Functions**           | 3.6            | GA        | SCF                             | Vendor in zip    | ❔             | Min. | [Link](https://www.tencentcloud.com/document/product/583/40327) |
| **Vercel Functions**                  | 3.9            | Beta      | HTTP handler or WSGI / ASGI | ✅                | ❔             | Min. | [Link](https://vercel.com/templates/python/python-hello-world) |

## Pricing

|                                    | **Free Plan**                                                | Bill Limits | **First Paid Tier**                                          |
| ---------------------------------- | ------------------------------------------------------------ | ----------- | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute** | 3 month trial with resource limits                           | ❔           | [Based on requests and resources](https://www.alibabacloud.com/help/en/fc/product-overview/billing-overview) |
| **AWS Lambda**                     | [1m reqs / mo + 400,000 GB-s / month](https://docs.aws.amazon.com/whitepapers/latest/how-aws-pricing-works/lambda.html) (based on memory configuration) + [data transfer](https://aws.amazon.com/ec2/pricing/on-demand/) (~$0.09 per GB out) | 🚫           | $0.20 per 1m reqs + $0.00001667 per GB-s (over free tier) + [data transfer](https://aws.amazon.com/ec2/pricing/on-demand/) (~$0.09 per GB out) |
| **AWS Lambda@Edge**                | None                                                         | 🚫           | [$0.60 per 1m reqs + $0.00000625125 per GB-s](https://aws.amazon.com/cloudfront/pricing/) + data transfer (~$0.09 per GB out) |
| **Azure Functions**                | NA                                                           |             | 1536                                                         |
| **Fermyon**                        |                                                              |             |                                                              |
| **Fly.io**                         | NA                                                           |             | NA                                                           |
| **Google Cloud Functions**         | 128                                                          |             | 2048                                                         |
| **IBM Code Engine**                | 256                                                          |             | 2048                                                         |
| **Oracle (OCI) Functions**         | 128                                                          |             | 1024                                                         |
| **Tencent Cloud Functions**        |                                                              |             |                                                              |
| **Vercel Functions**               | 1024                                                         |             | 3008                                                         |

Notes: reqs = requests, m = million, mo = month, s = seconds, mem = memory

## Runtime Limits

|                                    | Memory (MB) |         | Execution Time (s) |                 | **Payloads (MB)** |              | Code Size (MB) | Scale Limits  |
| ---------------------------------- | ----------- | ------- | ------------------ | --------------- | ----------------- | ------------ | -------------- | ------------- |
|                                    | **Default** | **Max** | **Default**        | **Max**         | **Request**       | **Response** |                |               |
| **Alibaba Cloud Function Compute** | 32 GB       | 32 GB   | 86,400             | 86,400          | 32                | ?            | 500            | 300           |
| **AWS Lambda**                     | 128         | 10,240  | 3s                 | 15min           | 6                 | 6            | 50 (zip)       | 1k concurrent |
| **AWS Lambda@Edge**                | 128         | 128     | 30s                | 5s              | 50Mb              | 40 KB        |                |               |
| **Azure Functions**                | NA          | 1536    | 5m                 | 10m             | No limit          | No limit     |                |               |
| **Fermyon**                        |             |         |                    |                 |                   |              |                |               |
| **Fly.io**                         | NA          | NA      | No limit           | No limit        | No limit          | No limit     |                |               |
| **Google cloud Functions**         | 128         | 2048    | NA                 | 540s            | 10                | 10           |                |               |
| **IBM Code Engine**                | 256         | 2048    | 1m                 | 10m             | 5                 | 5            |                |               |
| **Oracle (OCI) Functions**         | 128         | 1024    | 30s                | 120s            | 6                 | 6            |                |               |
| **Tencent Cloud Functions**        |             |         |                    |                 |                   |              |                |               |
| **Vercel Functions**               | 1024        | 3008    | 10s (Free plan)    | 300s (Pro plan) | 5                 | 5            |                |               |

AWS allocates 1 vCPU per 1,769 MB of memory configured.

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

TODO: Need a benchmark suite for Python, see [this JS suite](https://github.com/serverless-benchmark/backend)

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

- [Comparison by Brecht De Rooms from Feb 6th, 2020](https://fauna.com/blog/comparison-faas-providers)
