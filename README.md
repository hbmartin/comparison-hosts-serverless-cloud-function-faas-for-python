# Comparing hosts / providers for serverless cloud functions (FaaS) for Python

TODO: Slack/ Discord / Twitter / Mastodon badges

No Python support (as of 11 March, 2024): Cloudflare Workers, Netlify Edge Functions, StackPath EdgeEngine. IBM Cloud Functions are deprecated.

This document provides an up-to-date comparison between hosted, serverless (no cost or management to spin down to zero) providers of cloud function hosts with Python runtimes. Note the disctinction between edge providers (execution at PoP) and non-edge (typically predetermined DS region).



## DevEx

|                                       | Python Version | Status | API Framework                   | requirements.txt            | Local Testing | Docs | Basic Example                                                |
| ------------------------------------- | -------------- | ------ | ------------------------------- | --------------------------- | ------------- | ---- | ------------------------------------------------------------ |
| **Vercel Functions**                  | 3.9            | Beta   | HTTP handler or WSGI / ASGI app | ✅                           | ❔             | Min. | (Yes)[https://vercel.com/templates/python/python-hello-world] |
| **IBM Code Engine**                   | 3.11           | GA     | Plain obj.                      | ✅                           | ✅             | 👍    | https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/function-codebundle-python |
| **Google / Firebase Cloud Functions** | 3.12           | GA     | Flask                           | ✅                           | ✅             | 👍👍   |                                                              |
| **Oracle (OCI) Functions**            | 3.11           | GA     | FDK                             | ✅                           | ❔             | Min. | https://github.com/oracle-samples/oracle-functions-samples/tree/master/samples/helloworld |
| **Azure Functions**                   | 3.11           | GA     | azure-functions                 | ✅                           | ✅             | 👍👍   | https://learn.microsoft.com/en-us/samples/browse/?products=azure-functions&languages=python |
| **AWS Lambda and Lambda@Edge**        | 3.12           | GA     | Plain obj.                      | Manual vendoring / bundling | ✅             | 👍    | https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/blank-python |
| **Alibaba Cloud Function Compute**    | 3.10           | GA     | Plain obj.                      | Manual vendoring / bundling | ❔             | 👍👍   | https://www.alibabacloud.com/help/en/functioncompute/latest/event-handlers |



## Pricing

|                                    | **Free Plan** | **First Paid Tier** | Second Paid Tier |
| ---------------------------------- | ------------- | ------------------- | ---------------- |
| **Vercel Functions**               | 1024          | 3008                |                  |
| **IBM Code Engine**                | 256           | 2048                |                  |
| **Google Cloud Functions**         | 128           | 2048                |                  |
| **Oracle (OCI) Functions**         | 128           | 1024                |                  |
| **Azure Functions**                | NA            | 1536                |                  |
| **AWS Lambda**                     | 128           | 3008                |                  |
| **AWS Lambda@Edge**                | 128           | 128                 |                  |
| **Alibaba Cloud Function Compute** | ?             | ?                   |                  |
| **Fly.io**                         | NA            | NA                  |                  |

## Technical Limits

|                            | Memory (MB) |         | Execution Time     |                   | **Payloads (MB)** |              |
| -------------------------- | ----------- | ------- | ------------------ | ----------------- | ----------------- | ------------ |
|                            | **Default** | **Max** | **Default**        | **Max**           | **Request**       | **Response** |
| **Vercel Functions**       | 1024        | 3008    | 10s (Free plan)    | 300s (Pro plan)   | 5                 | 5            |
| **IBM Cloud Functions**    | 256         | 2048    | 1m                 | 10m               | 5                 | 5            |
| **Google cloud Functions** | 128         | 2048    | NA                 | 540s              | 10                | 10           |
| **Oracle Functions** 4     | 128         | 1024    | 30s                | 120s              | 6                 | 6            |
| **Azure Functions** 5      | NA          | 1536    | 5m                 | 10m               | No limit          | No limit     |
| **AWS Lambda**             | 128         | 3008    | 3s                 | 15m               | 61                | 61           |
| **Alibaba Functions**      | ?           | ?       | ?                  | 600s              | 61                | 61           |
| **Cloudflare Workers**     | 128         | 128     | No limit 10ms CPU6 | No limit 10ms CPU | ?                 | ?            |
| **AWS Lambda@Edge**        | 128         | 128     | 30s                | 5s                | 50Mb              | 40 KB        |
| **Fly.io** 7               | NA          | NA      | No limit           | No limit          | No limit          | No limit     |



1. **Performance**: Look into the cold start times and the overall performance of the functions. Performance can vary significantly between providers and can impact the user experience, especially for customer-facing applications.
2. **Pricing**: Understand the pricing model of the provider. Serverless pricing can be complex and is typically based on the number of executions, computation time, and resources consumed (like memory). Estimate your usage to understand potential costs.
3. **Scalability**: Evaluate how well the platform can scale to meet demand. Serverless architectures are known for their ability to scale, but different providers may have different limits or scaling capabilities.
4. **Ecosystem and Integrations**: Consider the ecosystem around the provider, including integrations with other services and tools. Some providers offer a more extensive range of integrated services (like databases, authentication, and storage) which can simplify development.
5. **Developer Experience**: Tools, documentation, and the developer community are critical for efficiently developing and troubleshooting. Look for providers that offer good support, an active community, and a developer-friendly environment.
6. **Availability and Reliability**: Check the service level agreements (SLAs) and historical uptime of the providers. High availability and reliability are crucial for most applications, especially in production.
7. **Security**: Review the security features offered by the provider, including compliance certifications, data encryption, and network security options. Security is a top priority, especially for applications dealing with sensitive data.
8. **Geographical Reach**: If your application targets users in specific regions, ensure the provider has data centers or points of presence in those areas to reduce latency.
9. **Customization and Flexibility**: Some projects may require specific runtime environments, custom configurations, or other flexibilities. Check what customization options are available with each provider.
10. **Migration Support and Vendor Lock-in**: Consider how easy it is to migrate to or from the provider's platform. Some serverless platforms might use proprietary technologies that could lead to vendor lock-in, making it difficult to switch providers in the future.

