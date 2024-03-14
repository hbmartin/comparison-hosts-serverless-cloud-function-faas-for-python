# Comparing hosts / providers for serverless cloud functions (FaaS) for Python

[![discord](https://img.shields.io/discord/267624335836053506?logo=discord&label=&color=323338)](https://discord.gg/python)
[![twitter](https://img.shields.io/badge/@hmartin-00aced.svg?logo=twitter&logoColor=black)](https://twitter.com/hmartin)

No Python support: Cloudflare Workers, Netlify Edge Functions, StackPath EdgeEngine. IBM Cloud Functions are deprecated.

This document provides an up-to-date comparison between hosted, serverless (no cost or management to spin down to zero) providers of cloud function hosts with Python runtimes.
Note the distinction between edge providers (execution at PoP) and non-edge (typically predetermined DS region).

Please be aware that this information may be inaccurate or outdated. If you discover errors or outdated information please open a PR!

- [DevEx](#devex)
- [Pricing](#pricing)
- [Runtime Limits](#runtime-limits)
- [Other Platform Products](#other-platform-products)
- [Discussions, Community, and Support](#discussions-community-and-support)

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

Note that the "Free Plan" is intended to represent ongoing free resources i.e. not trials or sign-up credits.

|                                       | **Free Plan**                                                | Bill Limits                                                  | **First Paid Tier**                                          |
| ------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**    | 3 month trial with resource limits                           | ?                                                            | [Based on requests and resources](https://www.alibabacloud.com/help/en/fc/product-overview/billing-overview) |
| **AWS Lambda**                        | [1m reqs / mo + 400,000 GB-s / mo](https://docs.aws.amazon.com/whitepapers/latest/how-aws-pricing-works/lambda.html) (based on memory configuration) + [data egress](https://aws.amazon.com/ec2/pricing/on-demand/) (~$0.09 per GB) | 🚫                                                            | $0.20 per 1m reqs + $0.00001667 per GB-s (over free tier) + [data egress](https://aws.amazon.com/ec2/pricing/on-demand/) (~$0.09 per GB) |
| **AWS Lambda@Edge**                   | None                                                         | 🚫                                                            | [$0.60 per 1m reqs + $0.00000625125 per GB-s](https://aws.amazon.com/cloudfront/pricing/) + egress (~$0.09 per GB) |
| **Azure Functions**                   | 1m reqs / mo + 400,000 GB-s / mo + first 100 GB / mo egress  | [Yes](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/spending-limit) | $0.20 per 1m reqs + $0.000016 per GB-s (over free) + $0.08 per GB egress (over 100 GB) |
| **Fermyon**                           | [100k reqs + 5GB egress](https://www.fermyon.com/pricing)    | NA                                                           | 1m reqs + 50GB egress at $20 / mo                            |
| **Fly.io**                            | 3 shared-cpu-1x 256mb VMs + 100GB egress                     | NA                                                           | Depends on VM + $0.02 per GB egress (over free, min $5 / mo) + $0.15 per GB / mo stopped VMs |
| **Google / Firebase Cloud Functions** | 2m / mo reqs + 400k / mo GB-s + 200k / mo CPU-s + 5 GB / mo egress | [Yes](https://cloud.google.com/billing/docs/how-to/budgets)  | $0.40 per 1m reqs + $0.0000025 per GB-s +  $0.0000100 per CPU-s + $0.12 per GB egress |
| **IBM Code Engine**                   | 100k vCPU-s + 200k GB-s + 100k reqs per mo                   | ?                                                            | $0.00003431 / vCPU-s + $0.00000356 / GB-s + $0.538 / 1m reqs |
| **Oracle (OCI) Functions**            | 2m / mo reqs + 400k / mo GB-s                                | Yes                                                          | $0.0000002 per req + $0.00001417 per GB-s                    |
| **Tencent Cloud Functions**           | 3 month trial with resource limits                           | ?                                                            | $0.0000167 per GB-s + $0.002 per 10k reqs + $0.00000847 per idle GB-s + $0.06 per day + $0.0752 per GB egress |
| **Vercel Functions**                  | 100 GB-hrs + 100 GB data transfer                            | Yes                                                          | 1,000 GB-hrs + 1TB data transfer                             |

reqs = requests, m = million, mo = month, s = seconds, mem = memory, k = thousand, ms = milliseconds

## Runtime Limits

|                                       | Memory      |         | Execution Time (s) |         | **Payloads (MB)** |              | Code Size (MB) | Scale Limits              |
| ------------------------------------- | ----------- | ------- | ------------------ | ------- | ----------------- | ------------ | -------------- | ------------------------- |
|                                       | **Default** | **Max** | **Default**        | **Max** | **Request**       | **Response** |                |                           |
| **Alibaba Cloud Function Compute**    | 32 GB       | 32 GB   | 86k                | 86k     | 32                | ?            | 500            | 300                       |
| **AWS Lambda**                        | 128 MB      | 10 GB   | 3s                 | 15min   | 6                 | 6            | 50 (zip)       | 10k per reg. per sec.     |
| **AWS Lambda@Edge**                   | 128 MB      | 3 GB    | 3s                 | 30s     | ?                 | 1            | 50 (zip)       | 10k per reg. per sec.     |
| **Azure Functions**                   | 1.5 GB      | 14 GB   | 5min               | 10min   | 100               | ?            | ?              | 100 inst.                 |
| **Fermyon**                           | ?           | ?       | 30s                | 30s     | ?                 | ?            | 100            | 1k RPS                    |
| **Fly.io**                            | 256 MB      | 128 GB  | NA                 | NA      | NA                | NA           | NA             | NA                        |
| **Google / Firebase Cloud Functions** | -           | 32GB    | -                  | 60min   | 32                | 32           | None           | 1k RPS / inst.            |
| **IBM Code Engine**                   | 2 GB        | 4 GB    | 120s               | -       | ?                 | ?            | 100 KB         | ?                         |
| **Oracle (OCI) Functions**            | 128 MB      | 2 GB    | 30s                | 300s    | ?                 | ?            | ?              | 40 GB total mem           |
| **Tencent Cloud Functions**           | 64 MB       | 3 GB    | 1s                 | 900s    | 6                 | 6            | 500 (unzip)    | 64 GB total mem           |
| **Vercel Functions**                  | 1 GB        | 3 GB    | 10s                | 300s    | ?                 | ?            | 250 (zip)      | 1k RPS per 10 sec per reg |

AWS allocates 1 vCPU per 1,769 MB of memory configured.

## Other Platform Products

|                                       | SQL DB | No SQL DB | Blob Store | File Hosting | GPU  |
| ------------------------------------- | ------ | --------- | ---------- | ------------ | ---- |
| **Alibaba Cloud Function Compute**    | ✅      | ✅         | ✅          | ✅            | ✅    |
| **AWS Lambda and Lambda@Edge**        | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Azure Functions**                   | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Fermyon**                           | SQLite | 🚫         | 🚫          | 🚫            | ✅    |
| **Fly.io**                            | ✅      | ✅         | 🚫          | 🚫            | ✅    |
| **Google / Firebase Cloud Functions** | ✅      | ✅         | ✅          | ✅            | ✅    |
| **IBM Code Engine**                   | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Oracle (OCI) Functions**            | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Tencent Cloud Functions**           | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Vercel Functions**                  | ✅      | ✅         | ✅          | ✅            | 🚫    |

## Performance (median times)

TODO: Need a benchmark suite for Python, see [this JS suite](https://github.com/serverless-benchmark/backend)

|                                       | PoPs (if edge) or regions | Uptime | Cold Response (ms) | Warm Response (ms) | Overhead (ms) |
| ------------------------------------- | ------------------------- | ------ | ------------------ | ------------------ | ------------- |
| **Alibaba Cloud Function Compute**    |                           |        |                    |                    |               |
| **AWS Lambda**                        |                           |        |                    |                    |               |
| **AWS Lambda@Edge**                   |                           |        |                    |                    |               |
| **Azure Functions**                   |                           |        |                    |                    |               |
| **Fermyon**                           |                           |        |                    |                    |               |
| **Fly.io**                            |                           |        |                    |                    |               |
| **Google / Firebase Cloud Functions** |                           |        |                    |                    |               |
| **IBM Code Engine**                   |                           |        |                    |                    |               |
| **Oracle (OCI) Functions**            |                           |        |                    |                    |               |
| **Tencent Cloud Functions**           |                           |        |                    |                    |               |
| **Vercel Functions**                  |                           |        |                    |                    |               |

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
