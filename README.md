# Comparing hosts / providers for serverless cloud functions (FaaS) for Python

[![discord](https://img.shields.io/discord/267624335836053506?logo=discord&label=&color=323338)](https://discord.gg/python)
[![twitter](https://img.shields.io/badge/@hmartin-00aced.svg?logo=twitter&logoColor=black)](https://twitter.com/hmartin)

No Python support: [Netlify Functions](https://docs.netlify.com/build/functions/api/), [Supabase Edge Functions](https://supabase.com/docs/guides/functions), and [Deno Deploy](https://docs.deno.com/runtime/deploy/) (all JavaScript/TypeScript only). [IBM Cloud Functions (OpenWhisk) was shut down in October 2024](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-dep-overview) — IBM Code Engine is its replacement. AWS App Runner is excluded because [idle services still incur provisioned-memory charges](https://aws.amazon.com/apprunner/pricing/) (no scale to zero cost). StackPath EdgeEngine no longer exists.

This document provides a comparison between hosted, serverless (no cost or management to spin down to zero) providers of cloud function hosts with Python runtimes.
Note the distinction between edge providers (execution at PoP) and non-edge (typically predetermined DS region), and between true FaaS (per-request billing) and scale-to-zero container/VM platforms (per-second instance billing: Fly.io, Koyeb, Render).

**Data last verified July 2, 2026** against the sources listed in [References](#references-and-useful-links). Pricing is USD (except Scaleway, EUR) and subject to change — always confirm against the linked official pages. ❔ = could not be verified against official sources.

Please join our [discussions](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions) or fix/update information by [editing this doc](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/edit/main/README.md)!

See also the [Serverless SQL DB Comparison](https://github.com/hbmartin/comparison-serverless-cloud-sql-databases)

- [Recent Platform Changes](#recent-platform-changes-20242026)
- [DevEx](#devex)
- [Pricing](#pricing)
- [Runtime Limits](#runtime-limits)
- [Other Platform Products](#other-platform-products)
- [Discussions, Community, and Support](#discussions-community-and-support)
- [References and Useful Links](#references-and-useful-links)

## Recent Platform Changes (2024–2026)

- **AWS Lambda**: [Python 3.14 runtime GA (Nov 2025)](https://aws.amazon.com/blogs/compute/python-3-14-runtime-now-available-in-aws-lambda/); [Python 3.9 deprecated Dec 2025](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html); [cold-start (INIT phase) duration is billed since Aug 1, 2025](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/); [new AWS accounts get a credits-based free plan since Jul 2025](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/), though the Lambda always-free allowance persists.
- **Azure Functions**: [Flex Consumption plan GA (Nov 2024)](https://techcommunity.microsoft.com/blog/appsonazureblog/azure-functions-flex-consumption-is-now-generally-available/4298778) — scale to 1,000 instances, no enforced max timeout; [the legacy Linux Consumption plan retires Sept 30, 2028](https://learn.microsoft.com/en-us/azure/azure-functions/consumption-plan) and is capped at Python 3.12.
- **Google Cloud**: Cloud Functions was rebranded [Cloud Run functions](https://cloud.google.com/functions/pricing-overview) and 2nd-gen functions are billed as Cloud Run; [Python 3.14 GA, with uv as the default installer from 3.14 onward](https://docs.cloud.google.com/functions/docs/release-notes).
- **Cloudflare Workers**: ["Python Workers redux" (Dec 2025)](https://blog.cloudflare.com/python-workers-advancements/) — ~10x faster cold starts via deploy-time memory snapshots, [Python 3.13 (Pyodide 0.28) default bundle](https://github.com/cloudflare/workerd/blob/main/build/python_metadata.bzl), new uv-based `pywrangler` CLI. Still beta.
- **Vercel**: [Fluid Compute](https://vercel.com/docs/fluid-compute) with [Active CPU pricing](https://vercel.com/blog/introducing-active-cpu-pricing-for-fluid-compute) (2025) — pay CPU only while executing, not during I/O wait; [Python 3.13/3.14 added Feb 2026](https://vercel.com/changelog/python-3-13-and-3-14-are-now-available); [Python bundle limit raised to 500 MB](https://vercel.com/changelog/python-vercel-functions-bundle-size-limit-increased-to-500mb).
- **Alibaba Cloud**: [switched to tiered Compute-Unit (CU) billing on Aug 27, 2024](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/billing-overview-of-fc); the ongoing free tier was replaced by a [3-month trial quota](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/trial-quota-1).
- **Tencent Cloud**: ongoing free tier eliminated ([2022](https://cloud.tencent.com/document/product/583/73739)–[2024](https://cloud.tencent.com.cn/document/product/583/104909)); [API Gateway triggers decommissioned Jun 30, 2025](https://cloud.tencent.com.cn/document/product/583/107631); Python still capped at 3.10 — the product appears to be in maintenance mode (no official EOL notice found).
- **Fly.io**: [free allowances removed for new orgs Oct 7, 2024](https://fly.io/docs/about/pricing/#discontinued-plans) (replaced by a [one-time 2-VM-hour/7-day trial](https://fly.io/docs/about/free-trial/)); [GPU Machines deprecated, unavailable after Aug 1, 2026](https://fly.io/docs/about/pricing/#gpu-enabled-fly-machines-deprecated).
- **Fermyon**: [Spin joined the CNCF Sandbox (Jan 2025)](https://www.cncf.io/projects/spin/); [Akamai acquired Fermyon (Dec 1, 2025)](https://www.akamai.com/newsroom/press-release/akamai-announces-acquisition-of-function-as-a-service-company-fermyon) — Fermyon Cloud continues as a free open beta, while the GA product is Fermyon Wasm Functions on Akamai.
- **Oracle (OCI)**: [long-running (detached) functions up to 1 hour (Oct 2025)](https://docs.oracle.com/en-us/iaas/releasenotes/functions/functions-long-running-functions.htm); [max memory raised to 3 GB](https://docs.public.content.oci.oraclecloud.com/en-us/iaas/releasenotes/functions/functions-3gb-functions-support.htm).
- **IBM**: [Code Engine functions support Python 3.13 since Jun 2025](https://cloud.ibm.com/docs/codeengine?topic=codeengine-release-notes); [Python 3.11 is deprecated (EOL Oct 2026)](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime).
- **Render**: [new flat-fee workspace plans (2026)](https://render.com/docs/new-workspace-plans) — Hobby $0 / Pro $25 / Scale $499, with reduced included bandwidth (5/25 GB) and $0.15/GB overage; new Python services default to 3.14 with uv.
- **New entrants added below**: [Modal](https://modal.com/docs/guide) (Python-native serverless), [DigitalOcean Functions](https://docs.digitalocean.com/products/functions/), [Scaleway Serverless Functions](https://www.scaleway.com/en/docs/serverless-functions/), and [Koyeb](https://www.koyeb.com/docs). GPU-first Python serverless platforms ([RunPod](https://docs.runpod.io/serverless/pricing), [Beam](https://www.beam.cloud/), [Cerebrium](https://cerebrium.ai/)) are out of scope but worth knowing about.

## DevEx

|                                       | Python Version | Status    | API Framework                   | Requirements | Local Testing | Docs | Hello  World                                   |
| ------------------------------------- | -------------- | --------- | ------------------------------- | ---------------- | ------------- | ---- | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**    | [3.12](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/python/) | GA        | Plain object / WSGI            | Vend in zip or [Serverless Devs](https://www.alibabacloud.com/help/en/fc/developer-reference/what-is-serverless-devs) | ✅ | 🎉    | [Link](https://www.alibabacloud.com/help/en/functioncompute/fc/user-guide/event-handlers-1-1) |
| **AWS Lambda & Lambda@Edge**        | [3.14](https://aws.amazon.com/blogs/compute/python-3-14-runtime-now-available-in-aws-lambda/) | GA        | Plain object                | Vend in zip ([SAM build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-build.html) automates) | ✅ ([SAM local](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-invoke.html)) | [🎉](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html) | [Link](https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/blank-python) |
| **Azure Functions**                   | [3.13](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages) (3.14 preview) | GA        | [azure-functions v2 decorators](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python), ASGI/WSGI | ✅                | ✅ (Core Tools) | [🎉](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python?tabs=get-started%2Casgi%2Capplication-level&pivots=python-mode-decorators) | [Link](https://learn.microsoft.com/en-us/samples/browse/?products=azure-functions&languages=python) |
| **Cloudflare Workers** (WASM) | [3.13 (Pyodide)](https://developers.cloudflare.com/workers/languages/python/how-python-workers-work/) | [Beta](https://developers.cloudflare.com/workers/languages/python/) | fastapi [and others](https://developers.cloudflare.com/workers/languages/python/packages/) | [pyproject.toml + pywrangler](https://developers.cloudflare.com/workers/languages/python/packages/) | ✅ | 🎉 | [Link](https://github.com/cloudflare/python-workers-examples) |
| **DigitalOcean Functions**            | [3.13](https://docs.digitalocean.com/products/functions/reference/runtimes/python/) | GA | Plain object | requirements.txt + build.sh | ✅ ([doctl serverless](https://docs.digitalocean.com/products/functions/quickstart/)) | 👍 | [Link](https://docs.digitalocean.com/products/functions/quickstart/) |
| **Fermyon** (WASM)           | [3.10+](https://github.com/spinframework/spin-python-sdk) | [Experimental SDK](https://spinframework.dev/v3/python-components), Cloud in open beta | Spin ([componentize-py](https://github.com/bytecodealliance/componentize-py)) | ✅               | ✅ (`spin build --up`) | 🎉  | [Link](https://spinframework.dev/v3/python-components) |
| **Fly.io** (microVM)                  | Any              | GA        | Any                               | ✅                | ✅ | 👍    | [Link](https://fly.io/docs/python/) |
| **Google Cloud Run** | [3.14](https://docs.cloud.google.com/functions/docs/release-notes) | GA        | [functions-framework](https://github.com/GoogleCloudPlatform/functions-framework-python), Flask, FastAPI, or any container | [✅](https://cloud.google.com/run/docs/runtimes/python-dependencies) (requirements.txt or pyproject.toml) | ✅             | [🎉](https://docs.cloud.google.com/run/docs/runtimes/python) | [Link](https://docs.cloud.google.com/run/docs/write-functions) |
| **IBM Code Engine**                   | [3.13](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime) (3.11 deprecated) | GA        | Plain object                | ✅                | [Manual](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-test-local) (no emulator) | 👍    | [Link](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/function-codebundle-python) |
| **Koyeb** (containers)                | Any            | GA        | Any                             | ✅                | ✅             | 👍    | [Link](https://www.koyeb.com/docs) |
| **Modal**                             | [3.10–3.14 (per-image)](https://modal.com/docs/guide/images) | GA | [`@app.function()` decorators](https://modal.com/docs/guide), ASGI/WSGI | [In-code image definition](https://modal.com/docs/reference/modal.Image) (incl. `pip_install_from_requirements`) | ☁️ (`modal serve` runs in cloud with live-reload) | 🎉 | [Link](https://modal.com/docs/examples/hello_world) |
| **Oracle (OCI) Functions**            | [3.12](https://github.com/fnproject/fdk-python) | GA        | FDK                             | ✅ (built into container)  | ✅ ([Fn CLI](https://docs.oracle.com/en-us/iaas/developer-tutorials/tutorials/functions/func-setup-cli/01-summary.htm)) | Min. | [Link](https://github.com/oracle-samples/oracle-functions-samples/tree/master/samples/helloworld) |
| **Render** | [3.14](https://render.com/docs/python-version) | GA | Any WSGI/ASGI | ✅ | ✅ | 👍 | [Link](https://render.com/docs/deploy-flask) |
| **Scaleway Serverless Functions**     | [3.13](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-runtimes/) | GA | Handler function | [Vend in zip](https://www.scaleway.com/en/docs/serverless-functions/how-to/package-function-dependencies-in-zip/) | ✅ ([local framework](https://github.com/scaleway/serverless-functions-python)) | 👍 | [Link](https://www.scaleway.com/en/docs/serverless-functions/quickstart/) |
| **Tencent Cloud Functions**           | [3.10](https://cloud.tencent.com/document/product/583/55592) | GA (maintenance mode, see note) | Plain object / [Web functions](https://www.tencentcloud.com/document/product/583/40678) | Vend in zip    | ❔             | Min. | [Link](https://www.tencentcloud.com/document/product/583/40327) |
| **Vercel Functions**                  | [3.14](https://vercel.com/changelog/python-3-13-and-3-14-are-now-available) (3.12 default fallback) | [Beta](https://vercel.com/docs/functions/runtimes/python) | HTTP handler or WSGI / ASGI ([FastAPI/Flask/Django presets](https://vercel.com/docs/frameworks/backend/fastapi)) | ✅                | ✅ (`vercel dev`) | 👍 | [Link](https://vercel.com/templates/python/python-hello-world) |

Tencent note: no official discontinuation exists, but the Python runtime is stale (3.10, CentOS 7), the free tier is trial-only, [API Gateway triggers were decommissioned in June 2025](https://cloud.tencent.com.cn/document/product/583/107631), and Tencent's serverless investment has shifted to [CloudBase](https://cloud.tencent.com/product/tcb).

## Pricing

Note that the "Free Plan" is intended to represent ongoing free resources i.e. not trials or sign-up credits.

|                                                              | **Free Plan**                                                | Bill Limits                                                  | **First Paid Tier**                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**                           | 🚫 since Aug 2024; [3-month trial of 150k CUs / mo](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/trial-quota-1) + [220 GB / mo egress via CDT](https://www.alibabacloud.com/help/en/cdt/product-overview/cdt-public-network-traffic-free-quota-increased-from-20gb-month-to-200gb-month) | ?                                                            | [Tiered Compute-Unit billing: $0.000020 → $0.000014 per CU](https://www.alibabacloud.com/en/product/function-compute/pricing) + [egress via CDT](https://www.alibabacloud.com/help/en/cdt/internet-data-transfers/) |
| [**AWS Lambda**](https://aws.amazon.com/lambda/pricing/)     | 1m reqs / mo + 400,000 GB-s / mo (always free) + [100 GB / mo egress](https://aws.amazon.com/blogs/apn/aws-data-transfer-charges-for-server-and-serverless-architectures/) | 🚫                                                            | $0.20 per 1m reqs + $0.0000166667 per GB-s x86 ($0.0000133334 Arm) + ~$0.09 per GB egress; [INIT phase billed since Aug 2025](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/) |
| **AWS Lambda@Edge**                                          | None                                                         | 🚫                                                            | [$0.60 per 1m reqs + $0.00005001 per GB-s (50 ms granularity)](https://aws.amazon.com/lambda/pricing/) + egress (~$0.09 per GB) |
| [**Azure Functions**](https://azure.microsoft.com/en-us/pricing/details/functions/) | Consumption: 1m execs + 400k GB-s / mo; Flex: 250k execs + 100k GB-s / mo; + [100 GB / mo egress](https://azure.microsoft.com/en-us/pricing/details/bandwidth/) | [Yes](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/spending-limit) | Consumption: $0.20 per 1m + $0.000016 per GB-s; Flex: $0.40 per 1m + $0.000026 per GB-s; + $0.087 per GB egress (over 100 GB) |
| [**Cloudflare Workers**](https://developers.cloudflare.com/workers/platform/pricing/) | 100k reqs / day, 10ms CPU / req; egress always free          | ?                                                            | $5 / mo incl. 10m reqs + 30m CPU-ms; then $0.30 per 1m reqs + $0.02 per 1m CPU-ms (wall time & egress not billed) |
| [**DigitalOcean Functions**](https://docs.digitalocean.com/products/functions/details/pricing/) | 90,000 GiB-s / mo                                            | ?                                                            | $0.0000185 per GiB-s (no per-request fee)                    |
| **Fermyon**                                                  | [100k reqs + 5GB egress](https://developer.fermyon.com/cloud/faq) | NA                                                           | [Growth ~$19 / mo: 1m reqs + 50GB egress](https://www.fermyon.com/pricing) |
| **Fly.io**                                                   | 🚫 for new orgs since Oct 2024 ([one-time trial: 2 VM-hrs / 7 days](https://fly.io/docs/about/free-trial/)) | NA                                                           | [Per-second VM pricing (shared-cpu-1x 256 MB ≈ $2 / mo)](https://fly.io/docs/about/pricing/) + $0.02 per GB egress (NA/EU) + $0.15 per GB / mo stopped VM rootfs |
| **[Google Cloud Run](https://cloud.google.com/run/pricing)** | 2m reqs + 180k vCPU-s + 360k GiB-s RAM + 1 GiB NA egress / mo | [Yes](https://cloud.google.com/billing/docs/how-to/budgets)  | $0.40 per 1m reqs + $0.000024 per vCPU-s + $0.0000025 per GiB-s + egress at [network rates](https://cloud.google.com/vpc/network-pricing) |
| [**IBM Code Engine**](https://cloud.ibm.com/docs/codeengine?topic=codeengine-pricing) | 100k vCPU-s + 200k GB-s + 100k reqs / mo                     | ?                                                            | $0.00003431 per vCPU-s + $0.00000356 per GB-s + $0.538 per 1m reqs |
| [**Koyeb**](https://www.koyeb.com/pricing)                   | One free instance (0.1 vCPU / 512 MB), [scales to zero after idle](https://www.koyeb.com/docs/run-and-scale/scale-to-zero) | NA                                                           | Per-second instance billing from ~$0.000006 / s              |
| [**Modal**](https://modal.com/pricing)                       | $30 / mo usage credit (Starter plan)                         | Yes (credits)                                                | $0.0000131 per CPU core-s + $0.00000222 per GiB-s RAM (no request or egress fees) |
| [**Oracle (OCI) Functions**](https://www.oracle.com/cloud/price-list/) | 2m reqs + 400k GB-s / mo + 10 TB / mo egress                 | Yes                                                          | $0.0000002 per req + $0.00001417 per GB-s + $0.0085 per GB egress |
| **[Render](https://render.com/pricing)**                     | [Free instance (0.1 CPU / 512 MB), 750 hrs / mo, sleeps after 15 min idle](https://render.com/docs/free) | NA                                                           | $7 / mo Starter instance (0.5 CPU / 512 MB) + $0.15 per GB [egress over included 5–25 GB](https://render.com/docs/new-workspace-plans) |
| [**Scaleway Serverless Functions**](https://www.scaleway.com/en/pricing/serverless/) | [1m reqs + 400k GB-s / mo; free ingress/egress](https://www.scaleway.com/en/docs/serverless-functions/faq/) | ?                                                            | €0.15 per 1m reqs + €0.0000170 per GB-s                      |
| **Tencent Cloud Functions**                                  | 🚫 ([3-month trial only](https://www.tencentcloud.com/document/product/583/12282); [free tier eliminated 2022–2024](https://cloud.tencent.com.cn/document/product/583/104909)) | ?                                                            | Pay-as-you-go: GB-s + per-10k invocations + egress ([price table](https://www.tencentcloud.com/document/product/583/12281)) |
| [**Vercel Functions**](https://vercel.com/docs/functions/usage-and-pricing) | Hobby: [1m invocations + 4 active-CPU-hrs + 360 GB-hrs provisioned mem + 100 GB transfer / mo](https://vercel.com/docs/limits) | Yes (hard caps on Hobby)                                     | Pro $20 / seat / mo + $0.60 per 1m invocations + $0.128 per active-CPU-hr + $0.0106 per GB-hr provisioned mem |

reqs = requests, m = million, mo = month, s = seconds, mem = memory, k = thousand, ms = milliseconds, CU = Alibaba Compute Unit

## Runtime Limits

|                                    | Memory      |         | Execution Time (s) |         | **Payloads (MB)** |              | Code Size (MB) | Scale Limits              |
| ---------------------------------- | ----------- | ------- | ------------------ | ------- | ----------------- | ------------ | -------------- | ------------------------- |
|                                    | **Default** | **Max** | **Default**        | **Max** | **Request**       | **Response** |                |                           |
| **Alibaba Cloud Function Compute** | ❔          | [32 GB](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/limits-of-usage) | ❔                | 86,400 (24 h) | [32 (sync) / 0.128 (async)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/synchronous-invocations) | ❔            | 500 (zip, major regions) | [100 on-demand inst. / region default](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/overview-of-configuring-the-maximum-number-of-on-demand-instances) |
| **AWS Lambda**                     | 128 MB      | [10 GB](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) | 3s                 | 15min   | 6                 | 6 (20 streaming) | 50 (zip) / 250 (unzip) / 10 GB (image) | 1k concurrent / region (soft) |
| **AWS Lambda@Edge**                | 128 MB      | [3 GB (origin)](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html) | -                  | 5s viewer / 30s origin | ?    | 0.04 viewer / 1 origin | 50 (zip) | per CloudFront quotas     |
| **Azure Functions**                | [1.5 GB (Consumption, fixed)](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale) | [4 GB (Flex)](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan) | 5min               | 10min (Consumption); no max on Flex (HTTP: 230s) | [100](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale) | ?            | ?              | 100 inst. (Consumption) / 1,000 (Flex) |
| **Cloudflare Workers**             | [128 MB (fixed)](https://developers.cloudflare.com/workers/platform/limits/) | 128 MB  | 10ms CPU (free) / 30s CPU (paid default) | 300s CPU (paid); no wall-clock limit | 100         | none enforced | 3 (free) / 10 (paid), compressed | 100 / 500 Workers per account |
| **DigitalOcean Functions**         | [256 MB](https://docs.digitalocean.com/products/functions/details/limits/) | 1 GB    | 3s                 | 15min   | 1                 | 1            | 48             | 120 concurrent / namespace |
| **Fermyon**                        | ? (Wasm)    | ?       | [30s](https://developer.fermyon.com/cloud/faq) | 30s     | 10                | 10           | 100            | 1k RPS                    |
| **Fly.io**                         | 256 MB      | [128 GB (16 perf. vCPU)](https://fly.io/docs/machines/guides-examples/machine-sizing/) | NA      | NA      | NA                | NA           | NA             | NA                        |
| **Google Cloud Run**               | 512 MiB     | [32 GiB](https://cloud.google.com/run/docs/configuring/services/memory-limits) | 5min               | [60min](https://cloud.google.com/run/docs/configuring/request-timeout) | [32](https://cloud.google.com/run/quotas) | 32 (unbounded if streamed) | None           | [80 concurrent reqs / inst. default (max 1k)](https://cloud.google.com/run/docs/about-concurrency) |
| **IBM Code Engine**                | [4 GB (1 vCPU)](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime#fun-supported-combo) | 4 GB    | -                  | [120s](https://cloud.ibm.com/docs/codeengine?topic=codeengine-limits#limits_functions) | 5                 | 5            | 0.1 (inline) / 200 (source) | 20 fns / project; 250 inst. |
| **Koyeb**                          | [per instance type](https://www.koyeb.com/docs/reference/instances) | -       | NA                 | NA      | NA                | NA           | NA             | instance-based            |
| **Modal**                          | configurable | [~336 GiB](https://modal.com/docs/guide/resources) ❔ | [300s](https://modal.com/docs/guide/timeouts) | 86,400 (24 h) | [4 GiB](https://modal.com/docs/guide/webhooks) | unlimited    | NA (image-based) | [100 containers / 10 GPUs (Starter)](https://modal.com/pricing) |
| **Oracle (OCI) Functions**         | [128 MB](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscustomizing.htm) | [3 GB](https://docs.public.content.oci.oraclecloud.com/en-us/iaas/releasenotes/functions/functions-3gb-functions-support.htm) | 30s                | 300s ([3,600s detached](https://docs.oracle.com/en-us/iaas/releasenotes/functions/functions-long-running-functions.htm)) | [6](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionstroubleshooting_topic-Issues-invoking-functions.htm) | 6            | ?              | [60 GB total mem / AD](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsmonitoringcapacityusage_topic-Monitoring-concurrent-function-execution.htm) |
| **Render**                         | 512 MB (free) | 8 GB+ (instance type) | NA        | NA      | NA                | NA           | NA             | [100 inst. / service (autoscaling on Pro+)](https://render.com/docs/scaling) |
| **Scaleway Serverless Functions**  | [128 MB](https://www.scaleway.com/en/docs/serverless-functions/reference-content/available-memory-and-cpu-tiers/) | 4 GB    | [300s](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-limitations/) | 3,600 (HTTP) | 6            | ?            | 100 (zip) / 500 (unzip) | 50 inst. / fn; 5k reqs / s |
| **Tencent Cloud Functions**        | [128 MB](https://www.tencentcloud.com/document/product/583/9694) | 4 GB ❔  | 3s                 | 900s (24 h async) ❔ | [6](https://www.tencentcloud.com/document/product/583/11637) | 6            | 500 (unzip) ❔  | region-shared concurrency |
| **Vercel Functions**               | [2 GB (Standard, 1 vCPU)](https://vercel.com/docs/functions/configuring-functions/memory) | 4 GB (Performance, 2 vCPU) | [300s](https://vercel.com/docs/functions/configuring-functions/duration) | 800s (1,800s beta) | [4.5](https://vercel.com/docs/functions/limitations) | 4.5          | 250 ([500 for Python](https://vercel.com/changelog/python-vercel-functions-bundle-size-limit-increased-to-500mb)) | [~30k concurrent (Pro)](https://vercel.com/docs/functions/concurrency-scaling) |

AWS allocates 1 vCPU per 1,769 MB of memory configured. Cloudflare bills and limits CPU time only — wall-clock time (e.g. waiting on I/O) is unlimited and unbilled. Modal, Fly.io, Koyeb, and Render size CPU/memory per instance rather than per invocation.

## Other Platform Products

|                                    | SQL DB | No SQL DB | Blob Store | File Hosting | GPU  |
| ---------------------------------- | ------ | --------- | ---------- | ------------ | ---- |
| **Alibaba Cloud Function Compute** | ✅      | ✅         | ✅          | ✅            | ✅    |
| **AWS Lambda and Lambda@Edge**     | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Azure Functions**                | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Cloudflare Workers**             | SQLite (D1) | ✅    | ✅          | ✅            | ✅ (Workers AI) |
| **DigitalOcean Functions**         | ✅      | ✅         | ✅ (Spaces) | ✅            | ✅    |
| **Fermyon**                        | SQLite | 🚫         | 🚫          | 🚫            | ✅    |
| **Fly.io**                         | ✅      | ✅         | ✅ (Tigris) | 🚫            | [🚫 after Aug 2026](https://fly.io/docs/about/pricing/#gpu-enabled-fly-machines-deprecated) |
| **Google Cloud Run**               | ✅      | ✅         | ✅          | ✅            | ✅    |
| **IBM Code Engine**                | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Koyeb**                          | ✅ (Postgres) | 🚫    | 🚫          | 🚫            | ✅    |
| **Modal**                          | 🚫      | ✅ (Dicts/Queues) | ✅ (Volumes) | 🚫       | ✅    |
| **Oracle (OCI) Functions**         | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Render**                         | ✅ (Postgres) | ✅ (Key Value) | 🚫    | ✅ (static)   | 🚫    |
| **Scaleway Serverless Functions**  | ✅      | ✅         | ✅          | ❔            | ✅    |
| **Tencent Cloud Functions**        | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Vercel Functions**               | ✅      | ✅         | ✅ (Blob)   | ✅            | 🚫    |

## Performance (median times)

TODO: Need a benchmark suite for Python, see [this JS suite](https://github.com/serverless-benchmark/backend)

|                                    | PoPs (if edge) or regions | Uptime | Cold Response (ms) | Warm Response (ms) | Overhead (ms) |
| ---------------------------------- | ------------------------- | ------ | ------------------ | ------------------ | ------------- |
| **Alibaba Cloud Function Compute** |                           |        |                    |                    |               |
| **AWS Lambda**                     |                           |        |                    |                    |               |
| **AWS Lambda@Edge**                |                           |        |                    |                    |               |
| **Azure Functions**                |                           |        |                    |                    |               |
| **Cloudflare Workers**             |                           |        | [~1,000 (with FastAPI, post-Dec-2025 snapshots)](https://blog.cloudflare.com/python-workers-advancements/) |                    |               |
| **DigitalOcean Functions**         |                           |        |                    |                    |               |
| **Fermyon**                        |                           |        |                    |                    |               |
| **Fly.io**                         |                           |        |                    |                    |               |
| **Google Cloud Run**               |                           |        |                    |                    |               |
| **IBM Code Engine**                |                           |        |                    |                    |               |
| **Koyeb**                          |                           |        | [~200 (Light Sleep) / 1,000–5,000 (Deep Sleep)](https://www.koyeb.com/blog/avoid-cold-starts-with-scale-to-zero-light-sleep) |                    |               |
| **Modal**                          |                           |        |                    |                    |               |
| **Oracle (OCI) Functions**         |                           |        |                    |                    |               |
| **Render**                         |                           |        | [up to ~60,000 (free tier spin-up)](https://render.com/docs/free) |                    |               |
| **Scaleway Serverless Functions**  |                           |        |                    |                    |               |
| **Tencent Cloud Functions**        |                           |        |                    |                    |               |
| **Vercel Functions**               |                           |        |                    |                    |               |

## Security Considerations

TODO: e.g. compliance certifications, data encryption, and network security options

See also [awesome serverless security](https://github.com/puresec/awesome-serverless-security)

## Discussions, Community, and Support

|                                    | Ours                                                         | Forum                                                        | GitHub                                                       | SO                                                           | Reddit                                                       |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute** | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/1) | [Forum](https://www.alibabacloud.com/forum)                  |                                                              | [SO](https://stackoverflow.com/questions/tagged/alibaba-cloud) | [r/AlibabCloud](https://www.reddit.com/r/AlibabaCloud/)      |
| **AWS Lambda and Lambda@Edge**     | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/2) | [re:Post](https://repost.aws/tags/questions/TA5uNafDy2TpGNjidWLMSxDw?view=all) | [GitHub](https://github.com/aws/aws-lambda-builders)         | [SO](https://stackoverflow.com/collectives/aws)              | [r/aws](https://www.reddit.com/r/aws/)                       |
| **Azure Functions**                | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/3) | [Forum](https://learn.microsoft.com/en-us/answers/tags/87/azure-functions) | [GitHub](https://github.com/Azure/azure-sdk-for-python)      | [SO](https://stackoverflow.com/collectives/azure)            | [r/AZURE](https://www.reddit.com/r/AZURE/)                   |
| **Cloudflare Workers**             | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/10) | [Forum](https://community.cloudflare.com/c/developers/workers/40) | [GitHub](https://github.com/cloudflare/workers-py)           | [SO](https://stackoverflow.com/questions/tagged/cloudflare-workers) | [r/CloudFlare](https://www.reddit.com/r/CloudFlare/)         |
| **DigitalOcean Functions**         |                                                              | [Community](https://www.digitalocean.com/community)          | [GitHub](https://github.com/digitalocean/doctl)              | [SO](https://stackoverflow.com/questions/tagged/digital-ocean) |                                                              |
| **Fermyon**                        | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/4) | [Discord](https://discord.com/invite/P4Cx7xUbJu)             | [Spin (CNCF)](https://github.com/spinframework/spin)         | [SO](https://stackoverflow.com/questions/tagged/fermyon-spin) |                                                              |
| **Fly.io**                         | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/5) | [Forum](https://community.fly.io/)                           |                                                              | [SO](https://stackoverflow.com/questions/tagged/fly?tab=Active) |                                                              |
| **Google Cloud Run**               | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/6) | [Group](https://groups.google.com/g/firebase-talk/)          | [GitHub](https://github.com/firebase/firebase-functions-python) | [SO](https://stackoverflow.com/collectives/google-cloud)     | [r/Firebase](https://www.reddit.com/r/Firebase/) and [r/googlecloud](https://www.reddit.com/r/googlecloud/) |
| **IBM Code Engine**                | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/7) | [Slack](https://cloud.ibm.com/kubernetes/slack)              | [GitHub](https://github.com/IBM/CodeEngine)                  | [SO](https://stackoverflow.com/questions/tagged/ibm-cloud-code-engine) |                                                              |
| **Koyeb**                          |                                                              | [Community](https://community.koyeb.com)                     |                                                              |                                                              |                                                              |
| **Modal**                          |                                                              | [Slack](https://modal.com/slack)                             | [GitHub](https://github.com/modal-labs/modal-client)         |                                                              |                                                              |
| **Oracle (OCI) Functions**         | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/8) | [Forum](https://forums.oracle.com/ords/apexds/domain/dev-community/category/containers-cloud-native) | [GitHub](https://github.com/oracle-samples/oracle-functions-samples) | [SO](https://stackoverflow.com/questions/tagged/oracle-cloud-functions) | [r/oraclecloud](https://www.reddit.com/r/oraclecloud/)       |
| **Render**                         |                                                              | [Community](https://community.render.com)                    | [Feedback](https://feedback.render.com)                      |                                                              |                                                              |
| **Scaleway Serverless Functions**  |                                                              | [Slack](https://slack.scaleway.com)                          | [GitHub](https://github.com/scaleway/serverless-functions-python) | [SO](https://stackoverflow.com/questions/tagged/scaleway)    |                                                              |
| **Vercel Functions**               | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/9) | [Help](https://vercel.com/help)                              | [GitHub](https://github.com/orgs/vercel/discussions)         | [SO](https://stackoverflow.com/questions/tagged/vercel)      | [r/Vercel](https://www.reddit.com/r/vercel/)                 |

## References and Useful Links

All sources below were used to verify the data above and were accessed on July 2, 2026.

### Alibaba Cloud Function Compute

- [Python runtimes (FC 3.0)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/python/)
- [Billing overview (CU model)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/billing-overview-of-fc) · [Pricing page](https://www.alibabacloud.com/en/product/function-compute/pricing) · [Trial quota](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/trial-quota-1)
- [Limits of usage](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/limits-of-usage) · [Synchronous invocations](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/synchronous-invocations)
- [CDT egress free quota](https://www.alibabacloud.com/help/en/cdt/product-overview/cdt-public-network-traffic-free-quota-increased-from-20gb-month-to-200gb-month) · [Serverless Devs](https://www.alibabacloud.com/help/en/fc/developer-reference/what-is-serverless-devs)

### AWS Lambda / Lambda@Edge

- [Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [Lambda quotas/limits](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) · [Runtimes & deprecation schedule](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [Python 3.14 runtime announcement (Nov 2025)](https://aws.amazon.com/blogs/compute/python-3-14-runtime-now-available-in-aws-lambda/) · [Python in Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html) · [Packaging Python dependencies](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html)
- [INIT-phase billing change (Aug 2025)](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/) · [Free Tier restructure (Jul 2025)](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/) · [Data transfer charges](https://aws.amazon.com/blogs/apn/aws-data-transfer-charges-for-server-and-serverless-architectures/)
- [Lambda@Edge restrictions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-edge-function-restrictions.html) · [CloudFront quotas](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html)

### Azure Functions

- [Supported languages/versions](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages) · [Python developer reference](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python)
- [Functions pricing](https://azure.microsoft.com/en-us/pricing/details/functions/) · [Consumption plan costs](https://learn.microsoft.com/en-us/azure/azure-functions/functions-consumption-costs) · [Bandwidth pricing](https://azure.microsoft.com/en-us/pricing/details/bandwidth/)
- [Flex Consumption plan](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan) · [Flex GA announcement (Nov 2024)](https://techcommunity.microsoft.com/blog/appsonazureblog/azure-functions-flex-consumption-is-now-generally-available/4298778) · [Scale & hosting limits](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale)
- [Linux Consumption retirement (Sept 2028)](https://learn.microsoft.com/en-us/azure/azure-functions/consumption-plan)

### Cloudflare Workers (Python)

- [Python Workers docs (beta)](https://developers.cloudflare.com/workers/languages/python/) · [How Python Workers work](https://developers.cloudflare.com/workers/languages/python/how-python-workers-work/) · [Packages](https://developers.cloudflare.com/workers/languages/python/packages/)
- [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/) · [Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- ["Python Workers redux" (Dec 2025)](https://blog.cloudflare.com/python-workers-advancements/) · [workerd Python metadata (Pyodide/CPython versions)](https://github.com/cloudflare/workerd/blob/main/build/python_metadata.bzl)

### DigitalOcean Functions

- [Python runtime reference](https://docs.digitalocean.com/products/functions/reference/runtimes/python/) · [Pricing](https://docs.digitalocean.com/products/functions/details/pricing/) · [Limits](https://docs.digitalocean.com/products/functions/details/limits/) · [Quickstart](https://docs.digitalocean.com/products/functions/quickstart/)

### Fermyon / Spin

- [Spin Python components](https://spinframework.dev/v3/python-components) · [spin-python-sdk](https://github.com/spinframework/spin-python-sdk) · [Spin in CNCF Sandbox (Jan 2025)](https://www.cncf.io/projects/spin/)
- [Fermyon Cloud pricing & billing](https://developer.fermyon.com/cloud/pricing-and-billing) · [Cloud FAQ (quotas/limits)](https://developer.fermyon.com/cloud/faq)
- [Akamai acquires Fermyon (Dec 2025)](https://www.akamai.com/newsroom/press-release/akamai-announces-acquisition-of-function-as-a-service-company-fermyon) · [Fermyon Wasm Functions](https://www.fermyon.com/wasm-functions)

### Fly.io

- [Pricing](https://fly.io/docs/about/pricing/) · [Free trial](https://fly.io/docs/about/free-trial/) · [Billing (machine states)](https://fly.io/docs/about/billing/)
- [Python on Fly](https://fly.io/docs/python/) · [Autostop/autostart (scale to zero)](https://fly.io/docs/launch/autostop-autostart/) · [Machine sizing](https://fly.io/docs/machines/guides-examples/machine-sizing/)

### Google Cloud Run

- [Cloud Run pricing](https://cloud.google.com/run/pricing) · [Quotas & limits](https://cloud.google.com/run/quotas) · [Python runtime](https://docs.cloud.google.com/run/docs/runtimes/python) · [Python dependencies](https://cloud.google.com/run/docs/runtimes/python-dependencies)
- [Cloud Run functions release notes (Python 3.14, uv)](https://docs.cloud.google.com/functions/docs/release-notes) · [Request timeout](https://cloud.google.com/run/docs/configuring/request-timeout) · [Concurrency](https://cloud.google.com/run/docs/about-concurrency) · [Functions Framework for Python](https://github.com/GoogleCloudPlatform/functions-framework-python)

### IBM Code Engine

- [Function runtimes (Python versions & lifecycle)](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime) · [Working with functions](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-work) · [Limits](https://cloud.ibm.com/docs/codeengine?topic=codeengine-limits) · [Pricing](https://cloud.ibm.com/docs/codeengine?topic=codeengine-pricing)
- [IBM Cloud Functions (OpenWhisk) deprecation](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-dep-overview)

### Koyeb

- [Pricing](https://www.koyeb.com/pricing) · [Instances (incl. free)](https://www.koyeb.com/docs/reference/instances) · [Scale to zero](https://www.koyeb.com/docs/run-and-scale/scale-to-zero) · [Light Sleep](https://www.koyeb.com/blog/avoid-cold-starts-with-scale-to-zero-light-sleep)

### Modal

- [Pricing](https://modal.com/pricing) · [Guide](https://modal.com/docs/guide) · [Images (Python versions)](https://modal.com/docs/guide/images) · [Timeouts](https://modal.com/docs/guide/timeouts) · [Resources (CPU/memory)](https://modal.com/docs/guide/resources) · [Web endpoint payloads](https://modal.com/docs/guide/webhooks) · [Scale](https://modal.com/docs/guide/scale)

### Oracle (OCI) Functions

- [OCI price list (Functions section)](https://www.oracle.com/cloud/price-list/) · [Functions product page](https://www.oracle.com/cloud/cloud-native/functions/)
- [Customizing functions (memory/timeout)](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscustomizing.htm) · [3 GB memory support](https://docs.public.content.oci.oraclecloud.com/en-us/iaas/releasenotes/functions/functions-3gb-functions-support.htm) · [Long-running functions (Oct 2025)](https://docs.oracle.com/en-us/iaas/releasenotes/functions/functions-long-running-functions.htm)
- [Concurrency monitoring](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsmonitoringcapacityusage_topic-Monitoring-concurrent-function-execution.htm) · [Python FDK](https://github.com/fnproject/fdk-python)

### Render

- [Free instances](https://render.com/docs/free) · [Pricing](https://render.com/pricing) · [New workspace plans (2026)](https://render.com/docs/new-workspace-plans) · [Python version](https://render.com/docs/python-version) · [Scaling](https://render.com/docs/scaling) · [Outbound bandwidth](https://render.com/docs/outbound-bandwidth)

### Scaleway Serverless Functions

- [Runtimes](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-runtimes/) · [FAQ (free tier & pricing)](https://www.scaleway.com/en/docs/serverless-functions/faq/) · [Serverless pricing](https://www.scaleway.com/en/pricing/serverless/)
- [Limitations](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-limitations/) · [Memory/CPU tiers](https://www.scaleway.com/en/docs/serverless-functions/reference-content/available-memory-and-cpu-tiers/) · [Local testing framework](https://github.com/scaleway/serverless-functions-python)

### Tencent Cloud Functions

- [Python runtime (3.10 max)](https://cloud.tencent.com/document/product/583/55592) · [Quotas & limits](https://www.tencentcloud.com/document/product/583/11637) · [Billing overview](https://www.tencentcloud.com/document/product/583/44254) · [Pricing](https://www.tencentcloud.com/document/product/583/12281) · [Free tier (trial)](https://www.tencentcloud.com/document/product/583/12282)
- [Basic package cancellation (Apr 2024)](https://cloud.tencent.com.cn/document/product/583/104909) · [API Gateway trigger EOL](https://cloud.tencent.com.cn/document/product/583/107631)

### Vercel Functions

- [Python runtime (beta)](https://vercel.com/docs/functions/runtimes/python) · [Python 3.13/3.14 changelog (Feb 2026)](https://vercel.com/changelog/python-3-13-and-3-14-are-now-available)
- [Usage & pricing](https://vercel.com/docs/functions/usage-and-pricing) · [Active CPU pricing announcement](https://vercel.com/blog/introducing-active-cpu-pricing-for-fluid-compute) · [Fluid compute](https://vercel.com/docs/fluid-compute) · [Limits](https://vercel.com/docs/limits)
- [Duration limits](https://vercel.com/docs/functions/configuring-functions/duration) · [Memory/CPU](https://vercel.com/docs/functions/configuring-functions/memory) · [Function limitations (payload)](https://vercel.com/docs/functions/limitations) · [500 MB Python bundle changelog](https://vercel.com/changelog/python-vercel-functions-bundle-size-limit-increased-to-500mb)

### Excluded platforms

- [Netlify Functions (JS/TS only)](https://docs.netlify.com/build/functions/api/) · [Supabase Edge Functions (Deno)](https://supabase.com/docs/guides/functions) · [Deno Deploy (JS/TS)](https://docs.deno.com/runtime/deploy/) · [AWS App Runner pricing (no scale-to-zero cost)](https://aws.amazon.com/apprunner/pricing/) · [IBM Cloud Functions EOL](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-dep-overview)

### Other comparisons

- [Comparison by Brecht De Rooms from Feb 6th, 2020](https://fauna.com/blog/comparison-faas-providers)
