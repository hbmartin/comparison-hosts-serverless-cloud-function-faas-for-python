# Comparing hosts / providers for serverless cloud functions (FaaS) for Python

[![discord](https://img.shields.io/discord/267624335836053506?logo=discord&label=&color=323338)](https://discord.gg/python)
[![twitter](https://img.shields.io/badge/@hmartin-00aced.svg?logo=twitter&logoColor=black)](https://twitter.com/hmartin)

No Python support: [Netlify Functions](https://docs.netlify.com/build/functions/get-started/) (JavaScript/TypeScript, plus Go and Rust via custom runtimes), [Supabase Edge Functions](https://supabase.com/docs/guides/functions) (Deno/TypeScript), and [Deno Deploy](https://docs.deno.com/runtime/deploy/) (JavaScript/TypeScript). [IBM Cloud Functions (OpenWhisk) was shut down in October 2024](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-dep-overview) — IBM Code Engine is its replacement. AWS App Runner is excluded because [idle services still incur provisioned-memory charges](https://aws.amazon.com/apprunner/pricing/) (no scale to zero cost) and it is now [in maintenance mode — closed to new customers since April 30, 2026](https://docs.aws.amazon.com/apprunner/latest/dg/apprunner-availability-change.html). StackPath EdgeEngine no longer exists.

This document provides a comparison between hosted, serverless (no cost or management to spin down to zero) providers of cloud function hosts with Python runtimes.
Note the distinction between edge providers (execution at PoP) and non-edge (typically predetermined DS region), and between true FaaS (per-request billing), serverless containers (per-use vCPU-s/GiB-s billing: Google Cloud Run, Azure Container Apps), and scale-to-zero container/VM platforms (per-second or per-minute instance billing: Fly.io, Koyeb, Railway, Render).

**Data last verified July 12, 2026.** Sources for table data are cited as footnotes (rendered at the bottom of this page); broader reading is collected in [References](#references-and-useful-links). Pricing is USD (except Scaleway in EUR and Huawei's China list prices in CNY) and subject to change — always confirm against the linked official pages. ❔ = could not be verified against official sources.

Please join our [discussions](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions) or fix/update information by [editing this doc](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/edit/main/README.md)!

See also the ["one-click" container image deployment comparison](one-click-image-deployment.md) (not Python specific) and the [Serverless SQL DB Comparison](https://github.com/hbmartin/comparison-serverless-cloud-sql-databases)

- [Recent Platform Changes](#recent-platform-changes-20242026)
- [DevEx](#devex)
- [Pricing](#pricing)
- [Runtime Limits](#runtime-limits)
- [Other Platform Products](#other-platform-products)
- [Discussions, Community, and Support](#discussions-community-and-support)
- [References and Useful Links](#references-and-useful-links)

## Recent Platform Changes (2024–2026)

- **AWS Lambda**: [Python 3.14 runtime GA (Nov 2025)](https://aws.amazon.com/blogs/compute/python-3-14-runtime-now-available-in-aws-lambda/); [Python 3.9 deprecated Dec 2025](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html); [cold-start (INIT phase) duration is billed since Aug 1, 2025](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/); [new AWS accounts get a credits-based free plan since Jul 2025](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/), though the Lambda always-free allowance persists.
- **Azure Functions**: [Flex Consumption plan GA (Nov 2024)](https://techcommunity.microsoft.com/blog/appsonazureblog/azure-functions-flex-consumption-is-now-generally-available/4298778) — scale to 1,000 instances, no enforced max timeout; [Python 3.14 went GA in early July 2026](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages); [the legacy Linux Consumption plan is retired for new features and shuts down Sept 30, 2028](https://learn.microsoft.com/en-us/azure/azure-functions/consumption-plan), capped at Python 3.12.
- **Google Cloud**: Cloud Functions was rebranded [Cloud Run functions](https://cloud.google.com/functions/pricing-overview) and 2nd-gen functions are billed as Cloud Run; [Python 3.14 GA, with uv as the default installer from 3.14 onward](https://docs.cloud.google.com/functions/docs/release-notes); [pyproject.toml source deploys went GA July 2026](https://docs.cloud.google.com/run/docs/release-notes).
- **Cloudflare**: ["Python Workers redux" (Dec 2025)](https://blog.cloudflare.com/python-workers-advancements/) — [~10x faster cold starts via deploy-time memory snapshots](https://developers.cloudflare.com/changelog/2025-12-08-python-cold-start-improvements/), [Python 3.13 (Pyodide 0.28) default bundle](https://github.com/cloudflare/workerd/blob/main/build/python_metadata.bzl), new uv-based `pywrangler` CLI. Still beta. Separately, [Cloudflare Containers went GA on April 13, 2026](https://developers.cloudflare.com/changelog/post/2026-04-13-containers-sandbox-ga/) — a scale-to-zero, per-second-billed route to full CPython on Cloudflare (see the [container comparison](one-click-image-deployment.md)).
- **Vercel**: [Fluid Compute](https://vercel.com/docs/fluid-compute) with [Active CPU pricing](https://vercel.com/blog/introducing-active-cpu-pricing-for-fluid-compute) (2025) — pay CPU only while executing, not during I/O wait; [Python 3.13/3.14 added Feb 2026](https://vercel.com/changelog/python-3-13-and-3-14-are-now-available); [Python bundle limit raised to 500 MB](https://vercel.com/changelog/python-vercel-functions-bundle-size-limit-increased-to-500mb); [Pro plans include $20/mo of flexible usage credit since Sept 2025](https://vercel.com/changelog/included-pro-usage-is-now-credit-based).
- **Alibaba Cloud**: [switched to tiered Compute-Unit (CU) billing on Aug 27, 2024](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/billing-overview-of-fc); the ongoing free tier was replaced by a [3-month trial quota](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/trial-quota-1).
- **Tencent Cloud**: ongoing free tier eliminated ([2022](https://cloud.tencent.com/document/product/583/73739)–[2024](https://cloud.tencent.com.cn/document/product/583/104909)); [API Gateway triggers decommissioned Jun 30, 2025](https://cloud.tencent.com.cn/document/product/583/107631); Python still capped at 3.10 — the product appears to be in maintenance mode (no official EOL notice found).
- **Fly.io**: [free allowances removed for new orgs Oct 7, 2024](https://fly.io/docs/about/pricing/#discontinued-plans) (replaced by a [one-time trial: 2 machine-hours or 7 days](https://fly.io/docs/about/free-trial/)); [GPU Machines deprecated, unavailable after Aug 1, 2026](https://fly.io/docs/gpus/); [volume snapshots are billed since Jan 1, 2026](https://fly.io/docs/about/pricing/).
- **Fermyon**: [Spin joined the CNCF Sandbox (Jan 2025)](https://www.cncf.io/projects/spin/); [Fermyon Wasm Functions on Akamai went GA Nov 12, 2025](https://www.globenewswire.com/news-release/2025/11/12/3186327/0/en/Fermyon-Wasm-Functions-on-Akamai-Now-Generally-Available-Scales-to-75-Million-RPS.html); [Akamai acquired Fermyon (Dec 1, 2025)](https://www.akamai.com/newsroom/press-release/akamai-announces-acquisition-of-function-as-a-service-company-fermyon) — Fermyon Cloud continues as a free open beta while the GA product is being rebranded [Akamai Functions](https://www.fermyon.com/wasm-functions).
- **Koyeb**: [Koyeb is joining Mistral AI (agreement announced Feb 17, 2026)](https://www.koyeb.com/blog/koyeb-is-joining-mistral-ai-to-build-the-future-of-ai-infrastructure) — the platform continues operating and is becoming a component of Mistral Compute; [Serverless Postgres went GA May 2025](https://www.koyeb.com/blog/serverless-postgres-ga-production-ready-databases-for-large-scale-and-ai-apps); [Light Sleep (~200 ms wake) scale-to-zero](https://www.koyeb.com/blog/avoid-cold-starts-with-scale-to-zero-light-sleep).
- **Oracle (OCI)**: [long-running (detached) functions up to 1 hour (Oct 2025)](https://docs.oracle.com/en-us/iaas/releasenotes/functions/functions-long-running-functions.htm); [max memory raised to 3 GB](https://docs.public.content.oci.oraclecloud.com/en-us/iaas/releasenotes/functions/functions-3gb-functions-support.htm).
- **IBM**: [Code Engine functions support Python 3.13 since Jun 2025](https://cloud.ibm.com/docs/codeengine?topic=codeengine-release-notes); [Python 3.11 is deprecated (EOL Oct 2026), and 3.13 is itself scheduled for deprecation Oct 1, 2026 (EOL Apr 2027)](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime).
- **Render**: [new flat-fee workspace plans (Apr 23, 2026)](https://render.com/changelog/updated-plans-for-render-workspaces) — Hobby $0 / Pro $25 / Scale $499 with 5 / 25 / 1,000 GB included bandwidth and $0.15/GB overage; legacy workspaces force-migrate Aug 1, 2026; [new Python services default to 3.14 with uv since Feb 2026](https://render.com/docs/python-version); [the community forum was retired in favor of Discord (Mar 2026)](https://render.com/docs/community).
- **Scaleway**: [Python 3.14 runtime added in the July 2026 runtime wave](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-runtimes/); [targeted price changes took effect June 1, 2026](https://www.scaleway.com/en/blog/a-transparent-update-on-scaleway-pricing/) — the function rates below were still listed as of July 12, 2026.
- **New entrants added below**: [Modal](https://modal.com/docs/guide) (Python-native serverless), [DigitalOcean Functions](https://docs.digitalocean.com/products/functions/), [Scaleway Serverless Functions](https://www.scaleway.com/en/docs/serverless-functions/), [Koyeb](https://www.koyeb.com/docs) — and, as of July 2026, [Appwrite Functions](https://appwrite.io/docs/products/functions), [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/overview), [Huawei Cloud FunctionGraph](https://www.huaweicloud.com/intl/en-us/product/functiongraph.html), [Railway](https://docs.railway.com/deployments/serverless), and [Yandex Cloud Functions](https://yandex.cloud/en/docs/functions/). GPU-first Python serverless platforms ([RunPod](https://docs.runpod.io/serverless/pricing), [Beam](https://www.beam.cloud/), [Cerebrium](https://cerebrium.ai/), [fal](https://fal.ai/docs/documentation/serverless/pricing)) are out of scope but worth knowing about.

### Watchlist and reviewed-but-excluded

Promising platforms that are not (yet) full rows below:

- [Wasmer Edge](https://wasmer.io/pricing) — CPython 3.12 at the edge via WASIX, with [unmodified FastAPI/Django/Flask support since Sept 2025](https://wasmer.io/posts/python-on-the-edge-powered-by-webassembly); free Hobby tier (100 compute-hrs + 150 GB bandwidth/mo) and $10/mo Pro, but [runtime resource limits are undocumented](https://docs.wasmer.io/edge/architecture) and the company is very small.
- [Leapcell](https://leapcell.io/) — new platform billing per request + per compute-ms with [$0 when idle and Python (FastAPI/Django/Flask) support](https://docs.leapcell.io/); young and lightly documented.
- [Sevalla](https://sevalla.com/) — Kinsta's PaaS: per-second pod billing with [Hibernation scale-to-zero (8–20 s wake)](https://sevalla.com/blog/shipping-hibernation-in-3-days/); no ongoing free tier (one-time credit only).
- [Blaxel](https://blaxel.ai/) — "perpetual sandboxes" for AI agents with a Python SDK, standby after 15 s idle and sub-25 ms resume; agent-infrastructure niche rather than general FaaS.

Reviewed and excluded: [Heroku](https://devcenter.heroku.com/articles/eco-dyno-hours) (Eco dynos sleep but are a flat $5/mo subscription, and the new Fir generation has no Eco tier), [Northflank](https://northflank.com/docs/v1/application/scale/scale-instances) (per-second billing but no wake-on-request scale to zero for web services), [Zeabur](https://zeabur.com/changelogs/phasing-out-shared-cluster) (stopped accepting new usage-billed shared-cluster services Apr 1, 2026 — now fixed-price dedicated servers), [PythonAnywhere](https://www.pythonanywhere.com/pricing/) (always-on hosting, not FaaS), [Genezio](https://genezio.com/blog/2025-recap-2026-roadmap/) (pivoted away from FaaS during 2025), and [Deta Space](https://news.ycombinator.com/item?id=41426388) (shut down Oct 2024).

## DevEx

|                                       | Python Version | Status    | API Framework                   | Requirements | Local Testing | Docs | Hello  World                                   |
| ------------------------------------- | -------------- | --------- | ------------------------------- | ---------------- | ------------- | ---- | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**    | 3.12[^ali-py] | GA        | Plain object / WSGI            | Vend in zip or [Serverless Devs](https://www.alibabacloud.com/help/en/fc/developer-reference/what-is-serverless-devs) | ✅ | 🎉    | [Link](https://www.alibabacloud.com/help/en/functioncompute/fc/user-guide/event-handlers-1-1) |
| **Appwrite Functions**                | 3.14 (Cloud: 3.9 / 3.12 / 3.14)[^appwrite-py] | GA (Cloud GA Aug 2025)[^appwrite-py] | [`def main(context)` handler](https://appwrite.io/docs/products/functions/develop) | ✅ (requirements.txt build command)[^appwrite-dev] | ✅ ([`appwrite run functions`](https://appwrite.io/docs/products/functions/develop-locally), Docker + hot reload) | 👍 | [Link](https://appwrite.io/docs/products/functions/quick-start) |
| **AWS Lambda & Lambda@Edge**        | 3.14[^aws-py] | GA        | Plain object                | Vend in zip ([SAM build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-build.html) automates) | ✅ ([SAM local](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/using-sam-cli-local-invoke.html)) | [🎉](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html) | [Link](https://github.com/awsdocs/aws-lambda-developer-guide/tree/main/sample-apps/blank-python) |
| **Azure Container Apps** (containers) | Any            | GA        | Any                             | ✅                | ✅ (any container runs locally) | 👍 | [Link](https://learn.microsoft.com/en-us/azure/container-apps/get-started-existing-container-image) |
| **Azure Functions**                   | 3.14[^azure-py] | GA        | [azure-functions v2 decorators](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python), ASGI/WSGI[^azure-asgi] | ✅                | ✅ (Core Tools) | [🎉](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python?tabs=get-started%2Casgi%2Capplication-level&pivots=python-mode-decorators) | [Link](https://learn.microsoft.com/en-us/samples/browse/?products=azure-functions&languages=python) |
| **Cloudflare Workers** (WASM) | 3.13 (Pyodide 0.28)[^cf-py] | Beta[^cf-beta] | fastapi [and others](https://developers.cloudflare.com/workers/languages/python/packages/) | [pyproject.toml + pywrangler](https://developers.cloudflare.com/workers/languages/python/packages/) | ✅ | 🎉 | [Link](https://github.com/cloudflare/python-workers-examples) |
| **DigitalOcean Functions**            | 3.13[^do-py] | GA | Plain object | requirements.txt + build.sh[^do-py] | ☁️ ([doctl serverless](https://docs.digitalocean.com/reference/doctl/reference/serverless/) — dev runs in a cloud namespace)[^do-local] | 👍 | [Link](https://docs.digitalocean.com/products/functions/quickstart/) |
| **Fermyon** (WASM)           | 3.10+[^fermyon-py] | [Experimental SDK](https://spinframework.dev/v3/python-components), Cloud in open beta[^fermyon-beta] | Spin ([componentize-py](https://github.com/bytecodealliance/componentize-py)) | ✅               | ✅ (`spin build --up`) | 🎉  | [Link](https://spinframework.dev/v3/python-components) |
| **Fly.io** (microVM)                  | Any              | GA        | Any                               | ✅                | ✅ | 👍    | [Link](https://fly.io/docs/python/) |
| **Google Cloud Run** | 3.14[^gcr-py] | GA        | [functions-framework](https://github.com/GoogleCloudPlatform/functions-framework-python), Flask, FastAPI, or any container | ✅ (requirements.txt or pyproject.toml)[^gcr-deps] | ✅             | [🎉](https://docs.cloud.google.com/run/docs/runtimes/python) | [Link](https://docs.cloud.google.com/run/docs/write-functions) |
| **Huawei Cloud FunctionGraph**        | 3.12[^huawei-py] | GA        | Plain object / [HTTP functions run web servers (WSGI/ASGI)](https://support.huaweicloud.com/intl/en-us/usermanual-functiongraph/functiongraph_01_1442.html) | Vend in zip or [dependency packages](https://support.huaweicloud.com/intl/en-us/devg-functiongraph/functiongraph_02_0616.html) | ✅ ([VS Code / PyCharm plugins](https://support.huaweicloud.com/intl/en-us/devg-functiongraph/functiongraph_01_1831.html)) | 👍 | [Link](https://support.huaweicloud.com/intl/en-us/qs-functiongraph/functiongraph_04_0101.html) |
| **IBM Code Engine**                   | 3.13 (3.11 deprecated)[^ibm-py] | GA        | Plain object                | ✅                | [Manual](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-test-local) (no emulator) | 👍    | [Link](https://github.com/IBM/CodeEngine/tree/main/helloworld-samples/function-codebundle-python) |
| **Koyeb** (containers)                | Any            | GA        | Any                             | ✅                | ✅             | 👍    | [Link](https://www.koyeb.com/docs) |
| **Modal**                             | 3.10–3.14 (per-image)[^modal-py] | GA | [`@app.function()` decorators](https://modal.com/docs/guide), ASGI/WSGI | [In-code image definition](https://modal.com/docs/reference/modal.Image) (incl. `pip_install_from_requirements`) | ☁️ (`modal serve` runs in cloud with live-reload) | 🎉 | [Link](https://modal.com/docs/examples/hello_world) |
| **Oracle (OCI) Functions**            | 3.12[^oci-py] | GA        | FDK                             | ✅ (built into container)  | ✅ ([Fn CLI](https://docs.oracle.com/en-us/iaas/developer-tutorials/tutorials/functions/func-setup-cli/01-summary.htm)) | Min. | [Link](https://github.com/oracle-samples/oracle-functions-samples/tree/master/samples/helloworld) |
| **Railway** (containers)              | Any (Railpack default 3.13)[^railway-py] | GA | Any (Flask / FastAPI / Django auto-detected)[^railway-py] | ✅ (requirements.txt, pyproject, Pipfile)[^railway-py] | ✅ ([`railway run`](https://docs.railway.com/cli/run)) | 👍 | [Link](https://docs.railway.com/guides/fastapi) |
| **Render** | 3.14[^render-py] | GA | Any WSGI/ASGI | ✅ | ✅ | 👍 | [Link](https://render.com/docs/deploy-flask) |
| **Scaleway Serverless Functions**     | 3.14[^scw-py] | GA | Handler function | [Vend in zip](https://www.scaleway.com/en/docs/serverless-functions/how-to/package-function-dependencies-in-zip/) | ✅ ([local framework](https://github.com/scaleway/serverless-functions-python)) | 👍 | [Link](https://www.scaleway.com/en/docs/serverless-functions/quickstart/) |
| **Tencent Cloud Functions**           | 3.10[^tencent-py] | GA (maintenance mode, see note) | Plain object / [Web functions](https://www.tencentcloud.com/document/product/583/40678) | Vend in zip    | ❔             | Min. | [Link](https://www.tencentcloud.com/document/product/583/40327) |
| **Vercel Functions**                  | 3.14 (3.12 default)[^vercel-py] | Beta[^vercel-py] | HTTP handler or WSGI / ASGI ([FastAPI/Flask/Django presets](https://vercel.com/docs/frameworks/backend/fastapi)) | ✅                | ✅ (`vercel dev`) | 👍 | [Link](https://vercel.com/templates/python/python-hello-world) |
| **Yandex Cloud Functions**            | 3.14 / 3.12[^yandex-py] | GA | Plain object (async `def` supported; no WSGI/ASGI)[^yandex-py] | ✅ (requirements.txt auto-installed)[^yandex-deps] | [Manual](https://yandex.cloud/en/docs/functions/operations/function/function-invoke) (console test tab, no emulator) | 👍 | [Link](https://yandex.cloud/en/docs/functions/quickstart/create-function/python-function-quickstart) |

Tencent note: no official discontinuation exists, but the Python runtime is stale (3.10, CentOS 7), the free tier is trial-only, [API Gateway triggers were decommissioned in June 2025](https://cloud.tencent.com.cn/document/product/583/107631), and Tencent's serverless investment has shifted to [CloudBase](https://cloud.tencent.com/product/tcb).

Region notes: Huawei Cloud has [no US regions](https://www.huaweicloud.com/intl/en-us/about/global-infrastructure.html) (China plus Asia-Pacific/EU/Middle East/Africa/LatAm). Yandex Cloud regions are in [Russia and Kazakhstan](https://yandex.cloud/en/docs/overview/concepts/region); after the [July 2024 Yandex/Nebius split](https://nebius.com/newsroom/ynv-announces-successful-completion-of-the-divestment-of-its-russia-based-businesses) international customers contract via non-Russian entities in USD — weigh data-residency and sanctions considerations.

## Pricing

Note that the "Free Plan" is intended to represent ongoing free resources i.e. not trials or sign-up credits.

|                                                              | **Free Plan**                                                | Bill Limits                                                  | **First Paid Tier**                                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute**                           | 🚫 since Aug 2024; 3-month trial of 150k CUs / mo[^ali-trial] + 220 GB / mo egress via CDT[^ali-cdt] | 🚫 (instance caps only)[^ali-cap]                             | Tiered Compute-Unit billing: $0.000020 → $0.000017 → $0.000014 per CU[^ali-price] + [egress via CDT](https://www.alibabacloud.com/help/en/cdt/internet-data-transfers/) |
| **Appwrite Functions** | 750k execs + 100 GB-hrs compute + 5 GB bandwidth / mo (free projects pause after 7 days of inactivity)[^appwrite-free] | ✅ (budget caps, paid plans)[^appwrite-cap] | Pro $25 / mo / project incl. 3.5m execs + 1,000 GB-hrs + 2 TB bandwidth; then $2 per 1m execs + $0.06 per GB-hr + $15 per 100 GB[^appwrite-price] |
| **AWS Lambda**     | 1m reqs / mo + 400,000 GB-s / mo (always free)[^aws-free] + 100 GB / mo egress[^aws-egress] | 🚫 (budget alerts only)[^aws-budget]                          | $0.20 per 1m reqs + $0.0000166667 per GB-s x86 ($0.0000133334 Arm)[^aws-price] + ~$0.09 per GB egress[^aws-egress]; INIT phase billed since Aug 2025[^aws-init] |
| **AWS Lambda@Edge**                                          | None[^edge-price]                                            | 🚫                                                            | $0.60 per 1m reqs + $0.00005001 per GB-s (1 ms granularity)[^edge-price] + egress (~$0.09 per GB) |
| **Azure Container Apps** | 180k vCPU-s + 360k GiB-s + 2m reqs / mo[^aca-price]; + 100 GB / mo egress[^azure-egress] | ✅ only on credit-based subscriptions[^azure-cap] | $0.000024 per vCPU-s ($0.000003 idle) + $0.000003 per GiB-s + $0.40 per 1m reqs over 2m[^aca-price] + $0.087 per GB egress[^azure-egress] |
| **Azure Functions** | Consumption: 1m execs + 400k GB-s / mo; Flex: 250k execs + 100k GB-s / mo[^azure-price]; + 100 GB / mo egress[^azure-egress] | ✅ only on credit-based subscriptions[^azure-cap] | Consumption: $0.20 per 1m + $0.000016 per GB-s; Flex: $0.40 per 1m + $0.000026 per GB-s[^azure-price]; + $0.087 per GB egress (over 100 GB)[^azure-egress] |
| **Cloudflare Workers** | 100k reqs / day, 10ms CPU / req; egress always free[^cf-price]          | 🚫 (per-invocation CPU caps only)[^cf-cap]                    | $5 / mo incl. 10m reqs + 30m CPU-ms; then $0.30 per 1m reqs + $0.02 per 1m CPU-ms (wall time & egress not billed)[^cf-price] |
| **DigitalOcean Functions** | 90,000 GiB-s / mo[^do-price]                                 | 🚫 (billing alerts only)[^do-cap]                             | $0.0000185 per GiB-s (no per-request fee, min. 100 ms billed per invocation)[^do-price] |
| **Fermyon**                                                  | 100k reqs + 5GB egress[^fermyon-quota]                       | NA                                                           | Growth $19.38 / mo: 1m reqs + 50GB egress[^fermyon-price]    |
| **Fly.io**                                                   | 🚫 for new orgs since Oct 2024 (one-time trial: 2 machine-hrs or 7 days)[^fly-trial] | NA                                                           | Per-second machine pricing (shared-cpu-1x 256 MB ≈ $2 / mo, region-dependent)[^fly-price] + $0.02 per GB egress (NA/EU) + $0.15 per GB / mo stopped machine rootfs[^fly-price] |
| **Google Cloud Run** | 2m reqs + 180k vCPU-s + 360k GiB-s RAM + 1 GiB NA egress / mo[^gcr-price] | 🚫 (budget alerts; programmatic cutoff possible)[^gcr-budget] | $0.40 per 1m reqs + $0.000024 per vCPU-s + $0.0000025 per GiB-s[^gcr-price] + egress at [network rates](https://cloud.google.com/vpc/network-pricing) |
| **Huawei Cloud FunctionGraph** | 1m reqs + 400,000 GB-s / mo (permanent)[^huawei-free] | 🚫 (budgets alert-only)[^huawei-cap] | ≈$0.20 per 1m reqs + $0.0000167 per GB-s ❔ (intl rates in console calculator; CN list ¥1.33 per 1m + ¥0.00011108 per GB-s)[^huawei-price]; egress billed via EIP/NAT[^huawei-price] |
| **IBM Code Engine** | 100k vCPU-s + 200k GB-s + 100k reqs / mo (shared across all Code Engine apps/jobs/fns)[^ibm-price]         | 🚫 (spending notifications only)[^ibm-cap]                    | $0.00003431 per vCPU-s + $0.00000356 per GB-s + $0.538 per 1m reqs[^ibm-price] |
| **Koyeb**                   | One free instance (0.1 vCPU / 512 MB), scales to zero after 1 h idle[^koyeb-free] | NA                                                           | Per-second instance billing from $0.0000006 / s (eco-nano, ≈ $1.61 / mo)[^koyeb-price] |
| **Modal**                       | $30 / mo recurring usage credit (Starter plan)[^modal-price] | ✅ (prepaid credits)                                          | $0.0000131 per CPU core-s + $0.00000222 per GiB-s RAM (no per-request fee; egress not billed as a line item)[^modal-price] |
| **Oracle (OCI) Functions** | 2m reqs + 400k GB-s / mo + 10 TB / mo egress[^oci-price]     | 🚫 (budget alerts only)[^oci-budget]                          | $0.0000002 per req + $0.00001417 per GB-s + $0.0085 per GB egress[^oci-price] |
| **Railway** | $1 / mo usage credit (0.5 GB RAM / 1 vCPU cap); "Serverless" sleep ≈ $0 compute while idle[^railway-sleep] | ✅ (hard usage limit stops workloads)[^railway-cap] | Hobby $5 / mo (incl. $5 usage) + $10 per GB RAM / mo + $20 per vCPU / mo + $0.05 per GB egress (per-minute metering)[^railway-price] |
| **Render**                     | Free instance (0.1 CPU / 512 MB), 750 hrs / mo, sleeps after 15 min idle[^render-free] | NA                                                           | $7 / mo Starter instance (0.5 CPU / 512 MB)[^render-price] + $0.15 per GB egress over included 5 GB (Hobby) / 25 GB (Pro) / 1 TB (Scale)[^render-plans] |
| **Scaleway Serverless Functions** | 1m reqs + 400k GB-s / mo; free ingress/egress[^scw-price]    | 🚫 (billing alerts only)[^scw-cap]                            | €0.15 per 1m reqs + €0.0000170 per GB-s[^scw-price]          |
| **Tencent Cloud Functions**                                  | 🚫 ([3-month trial only](https://www.tencentcloud.com/document/product/583/12282); [free tier eliminated 2022–2024](https://cloud.tencent.com.cn/document/product/583/104909)) | ?                                                            | Pay-as-you-go: GB-s + per-10k invocations + egress ([price table](https://www.tencentcloud.com/document/product/583/12281)) |
| **Vercel Functions** | Hobby: 1m invocations + 4 active-CPU-hrs + 360 GB-hrs provisioned mem + 100 GB transfer / mo[^vercel-hobby] | ✅ (hard caps on Hobby)[^vercel-hobby]                        | Pro $20 / seat / mo (incl. $20 / mo usage credit) + $0.60 per 1m invocations + $0.128 per active-CPU-hr + $0.0106 per GB-hr provisioned mem[^vercel-price] |
| **Yandex Cloud Functions** | 1m invocations + 10 GB×hr / mo + 100 GB / mo egress[^yandex-free] | 🚫 (budgets notify; can trigger a cutoff function)[^yandex-cap] | $0.144 per 1m invocations + $0.049 per GB×hr (≈$0.0000137 per GB-s; 100 ms rounding)[^yandex-price] |

reqs = requests, m = million, mo = month, s = seconds, mem = memory, k = thousand, ms = milliseconds, CU = Alibaba Compute Unit

## Runtime Limits

|                                    | Memory      |         | Execution Time (s) |         | **Payloads (MB)** |              | Code Size (MB) | Scale Limits              |
| ---------------------------------- | ----------- | ------- | ------------------ | ------- | ----------------- | ------------ | -------------- | ------------------------- |
|                                    | **Default** | **Max** | **Default**        | **Max** | **Request**       | **Response** |                |                           |
| **Alibaba Cloud Function Compute** | 512 MB (console)[^ali-createfn] | 32 GB[^ali-createfn] | 3s[^ali-createfn]                | 86,400 (24 h)[^ali-limits] | 32 (sync) / 0.128 (async)[^ali-payload] | not documented[^ali-limits]            | 500 (zip, major regions)[^ali-limits] | 100 on-demand inst. / region default[^ali-inst] |
| **Appwrite Functions**             | 512 MB (0.5 CPU; Free plan fixed)[^appwrite-limits] | 4 GB (4 CPU, Pro)[^appwrite-limits] | -                  | 900s (sync / HTTP capped at 30s)[^appwrite-timeout] | not documented    | not documented | ❔ (30 MB self-hosted default) | not published             |
| **AWS Lambda**                     | 128 MB      | 10 GB[^aws-limits] | 3s                 | 15min[^aws-limits] | 6[^aws-limits]                 | 6 (20 streaming)[^aws-limits] | 50 (zip) / 250 (unzip) / 10 GB (image)[^aws-limits] | 1k concurrent / region (soft)[^aws-limits] |
| **AWS Lambda@Edge**                | 128 MB      | 3 GB (origin)[^edge-limits] | -                  | 5s viewer / 30s origin[^edge-limits] | 0.04 viewer / 1 origin (body exposed to fn)[^edge-body]    | 0.04 viewer / 1 origin (generated)[^edge-body] | 1 (viewer) / 50 (origin) zip[^edge-limits] | per CloudFront quotas[^edge-limits]     |
| **Azure Container Apps**           | configurable | 8 GiB (4 vCPU) / replica[^aca-limits] | NA (HTTP ingress 240s default)[^aca-ingress] | 30 min idle (premium ingress)[^aca-ingress] | not documented | not documented | NA (image) | 0–1,000 replicas (default max 10)[^aca-scale] |
| **Azure Functions**                | 1.5 GB (Consumption, fixed)[^azure-limits] | 4 GB (Flex: 0.5 / 2 / 4 GB sizes)[^azure-flex] | 5min               | 10min (Consumption); no max on Flex (HTTP: 230s)[^azure-limits] | 210[^azure-limits] | not documented[^azure-limits] | 1 GB (deployment package)[^azure-pkg] | 100 inst. (Consumption) / 1,000 (Flex)[^azure-flex] |
| **Cloudflare Workers**             | 128 MB (fixed)[^cf-limits] | 128 MB  | 10ms CPU (free) / 30s CPU (paid default) | 300s CPU (paid); no wall-clock limit[^cf-limits] | 100 (zone-plan dependent)[^cf-body]         | none enforced | 3 (free) / 10 (paid), compressed[^cf-limits] | 100 / 500 Workers per account[^cf-limits] |
| **DigitalOcean Functions**         | 256 MB[^do-limits] | 1 GB    | 3s                 | 15min (sync auto-converts to async at 30s)[^do-async]   | 1[^do-limits]                 | 1[^do-limits]            | 48[^do-limits]             | 120 concurrent / namespace[^do-limits] |
| **Fermyon**                        | ? (Cloud); 128 MiB default on Wasm Functions[^fermyon-fwf]    | ?       | 30s[^fermyon-quota] | 30s     | 10[^fermyon-quota]                | 10           | 100[^fermyon-quota]            | 1k RPS[^fermyon-quota]                    |
| **Fly.io**                         | 256 MB      | 128 GB (16 perf. vCPU)[^fly-size] | NA      | NA      | NA                | NA           | NA             | NA                        |
| **Google Cloud Run**               | 512 MiB[^gcr-mem]     | 32 GiB[^gcr-mem] | 5min[^gcr-timeout]               | 60min[^gcr-timeout] | 32 (HTTP/1 only)[^gcr-quota] | 32 (unbounded if streamed)[^gcr-quota] | None           | 80×vCPU (CLI) or 80 (console) concurrent reqs / inst. default, max 1k[^gcr-conc] |
| **Huawei Cloud FunctionGraph**     | ❔ | 10 GB[^huawei-limits] | 3s[^huawei-timeout] | 259,200 (72 h async; 900s sync)[^huawei-timeout] | 6 (sync) / 0.25 (async)[^huawei-limits] | 6[^huawei-limits] | 40 (console zip) / 300 via OBS / 10 GB (image)[^huawei-limits] | 400 fns; 1k concurrent inst. default[^huawei-scale] |
| **IBM Code Engine**                | 4 GB (1 vCPU)[^ibm-limits] | 4 GB    | -                  | 120s[^ibm-limits] | 5[^ibm-limits]                 | 5[^ibm-limits]            | 0.1 (inline) / 200 (source)[^ibm-limits] | 20 fns / project; 250 inst.[^ibm-limits] |
| **Koyeb**                          | [per instance type](https://www.koyeb.com/docs/reference/instances) | -       | NA                 | NA      | NA                | NA           | NA             | instance-based            |
| **Modal**                          | configurable | 336 GiB (64 CPUs)[^modal-limits] | 300s[^modal-limits] | 86,400 (24 h)[^modal-limits] | 4 GiB[^modal-payload] | unlimited (streamed)[^modal-payload]    | NA (image-based) | 100 containers / 10 GPUs (Starter)[^modal-scale] |
| **Oracle (OCI) Functions**         | 128 MB[^oci-limits] | 3 GB[^oci-limits] | 30s                | 300s (3,600s detached)[^oci-detached] | 6[^oci-limits] | 6[^oci-limits]            | ?              | 60 GB total mem / AD[^oci-limits] |
| **Railway**                        | per plan (0.5 GB free cap)[^railway-free] | 8 GB / replica (Hobby); 24 GB (Pro)[^railway-scale] | NA | NA (HTTP: 15 min max, 5 min idle)[^railway-http] | NA | NA | NA (image: 4 GB free / 100 GB Hobby)[^railway-free] | ~10k conns / ~11k RPS per domain[^railway-http] |
| **Render**                         | 512 MB (free) | 32 GB (8 CPU, Pro Ultra; custom above)[^render-scale] | NA        | NA      | NA                | NA           | NA             | 100 inst. / service (autoscaling on paid)[^render-scale] |
| **Scaleway Serverless Functions**  | 128 MB (min tier)[^scw-mem] | 4 GB[^scw-mem]    | 300s[^scw-limits] | 3,600[^scw-limits] | 6[^scw-limits]            | not documented[^scw-limits]            | 100 (zip) / 500 (unzip)[^scw-limits] | 50 inst. / fn; 5k reqs / s[^scw-limits] |
| **Tencent Cloud Functions**        | 128 MB[^tencent-mem] | 3 GB (large-mem specs to 120 GB)[^tencent-mem]  | 3s[^tencent-timeout]                 | 900s (24 h async)[^tencent-timeout] | 6[^tencent-limits] | 6[^tencent-limits]            | 500 (unzip)[^tencent-limits]  | region-shared concurrency |
| **Vercel Functions**               | 2 GB (Standard, 1 vCPU)[^vercel-mem] | 4 GB (Performance, 2 vCPU)[^vercel-mem] | 300s[^vercel-limits] | 800s Pro (300s Hobby; 1,800s beta)[^vercel-limits] | 4.5[^vercel-limits] | 4.5[^vercel-limits]          | 250 (500 for Python; 5 GB "Large Functions" beta)[^vercel-bundle] | ~30k concurrent (Pro)[^vercel-conc] |
| **Yandex Cloud Functions**         | 128 MB[^yandex-limits] | 8 GB (4 vCPU)[^yandex-limits] | ❔ | 600s (3,600s long-lived)[^yandex-timeout] | 3.5[^yandex-limits] | 3.5[^yandex-limits] | 3.5 (console zip) / 128 via S3 / 680 unzipped[^yandex-limits] | 10 fns / cloud; 10 concurrent / AZ default[^yandex-limits] |

AWS allocates 1 vCPU per 1,769 MB of memory configured[^aws-vcpu]. Cloudflare bills and limits CPU time only — wall-clock time (e.g. waiting on I/O) is unlimited and unbilled[^cf-price]. Modal, Fly.io, Koyeb, Railway, Render, and Azure Container Apps size CPU/memory per instance/replica rather than per invocation.

## Other Platform Products

|                                    | SQL DB | No SQL DB | Blob Store | File Hosting | GPU  |
| ---------------------------------- | ------ | --------- | ---------- | ------------ | ---- |
| **Alibaba Cloud Function Compute** | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Appwrite Functions**             | 🚫 (document API over SQL)[^appwrite-db] | ✅ (TablesDB)[^appwrite-db] | ✅ (Storage) | ✅ (Sites)[^appwrite-sites] | 🚫    |
| **AWS Lambda and Lambda@Edge**     | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Azure Container Apps**           | ✅      | ✅         | ✅          | ✅            | ✅ (serverless GPUs)[^aca-gpu]    |
| **Azure Functions**                | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Cloudflare Workers**             | SQLite (D1) | ✅    | ✅          | ✅            | ✅ (Workers AI) |
| **DigitalOcean Functions**         | ✅      | ✅         | ✅ (Spaces) | ✅            | ✅    |
| **Fermyon**                        | SQLite | 🚫         | 🚫          | 🚫            | ✅    |
| **Fly.io**                         | ✅ (Managed Postgres)[^fly-mpg] | ✅  | ✅ (Tigris) | 🚫            | [🚫 after Aug 2026](https://fly.io/docs/gpus/) |
| **Google Cloud Run**               | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Huawei Cloud FunctionGraph**     | ✅      | ✅ (GeminiDB / DDS) | ✅ (OBS)   | ✅            | ✅ (incl. GPU functions)[^huawei-gpu]    |
| **IBM Code Engine**                | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Koyeb**                          | ✅ (Serverless Postgres, GA)[^koyeb-pg] | 🚫    | 🚫          | 🚫            | ✅    |
| **Modal**                          | 🚫      | ✅ (Dicts/Queues) | ✅ (Volumes) | 🚫       | ✅    |
| **Oracle (OCI) Functions**         | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Railway**                        | ✅ (templates, unmanaged)[^railway-db] | ✅ (Redis / Mongo templates)[^railway-db] | ✅ (Buckets)[^railway-buckets] | 🚫            | 🚫[^railway-gpu]    |
| **Render**                         | ✅ (Postgres) | ✅ (Key Value) | 🚫 (object storage in alpha)[^render-blob]    | ✅ (static)   | 🚫    |
| **Scaleway Serverless Functions**  | ✅      | ✅         | ✅          | ❔            | ✅    |
| **Tencent Cloud Functions**        | ✅      | ✅         | ✅          | ✅            | ✅    |
| **Vercel Functions**               | ✅      | ✅         | ✅ (Blob)   | ✅            | 🚫    |
| **Yandex Cloud Functions**         | ✅      | ✅ (YDB serverless) | ✅          | ✅ (static hosting)[^yandex-hosting] | ✅ (GPU VMs)    |

## Performance (median times)

TODO: Need a benchmark suite for Python, see [this JS suite](https://github.com/serverless-benchmark/backend)

|                                    | PoPs (if edge) or regions | Uptime | Cold Response (ms) | Warm Response (ms) | Overhead (ms) |
| ---------------------------------- | ------------------------- | ------ | ------------------ | ------------------ | ------------- |
| **Alibaba Cloud Function Compute** |                           |        |                    |                    |               |
| **Appwrite Functions**             |                           |        |                    |                    |               |
| **AWS Lambda**                     |                           |        | ~2,500 (FastAPI, per Cloudflare's cross-vendor test)[^cf-cold] |                    |               |
| **AWS Lambda@Edge**                |                           |        |                    |                    |               |
| **Azure Container Apps**           |                           |        | no official figure; ~15,000–30,000 commonly reported[^aca-cold] |                    |               |
| **Azure Functions**                |                           |        |                    |                    |               |
| **Cloudflare Workers**             |                           |        | ~1,000 (with FastAPI, post-Dec-2025 snapshots)[^cf-cold] |                    |               |
| **DigitalOcean Functions**         |                           |        |                    |                    |               |
| **Fermyon**                        |                           |        |                    |                    |               |
| **Fly.io**                         |                           |        |                    |                    |               |
| **Google Cloud Run**               |                           |        | ~3,100 (FastAPI, per Cloudflare's cross-vendor test)[^cf-cold] |                    |               |
| **Huawei Cloud FunctionGraph**     |                           |        |                    |                    |               |
| **IBM Code Engine**                |                           |        |                    |                    |               |
| **Koyeb**                          |                           |        | ~200 (Light Sleep) / 1,000–5,000 (Deep Sleep)[^koyeb-sleep] |                    |               |
| **Modal**                          |                           |        |                    |                    |               |
| **Oracle (OCI) Functions**         |                           |        |                    |                    |               |
| **Railway**                        |                           |        |                    |                    |               |
| **Render**                         |                           |        | ~60,000 ("about one minute" free-tier spin-up)[^render-free] |                    |               |
| **Scaleway Serverless Functions**  |                           |        |                    |                    |               |
| **Tencent Cloud Functions**        |                           |        |                    |                    |               |
| **Vercel Functions**               |                           |        |                    |                    |               |
| **Yandex Cloud Functions**         |                           |        |                    |                    |               |

## Security Considerations

TODO: e.g. compliance certifications, data encryption, and network security options

See also [awesome serverless security](https://github.com/puresec/awesome-serverless-security)

## Discussions, Community, and Support

|                                    | Ours                                                         | Forum                                                        | GitHub                                                       | SO                                                           | Reddit                                                       |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Alibaba Cloud Function Compute** | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/1) | [Forum](https://www.alibabacloud.com/forum)                  |                                                              | [SO](https://stackoverflow.com/questions/tagged/alibaba-cloud) | [r/AlibabaCloud](https://www.reddit.com/r/AlibabaCloud/)      |
| **Appwrite Functions**             |                                                              | [Discord](https://appwrite.io/discord)                       | [GitHub](https://github.com/appwrite/appwrite)               | [SO](https://stackoverflow.com/questions/tagged/appwrite)    | [r/appwrite](https://www.reddit.com/r/appwrite/)             |
| **AWS Lambda and Lambda@Edge**     | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/2) | [re:Post](https://repost.aws/tags/questions/TA5uNafDy2TpGNjidWLMSxDw?view=all) | [GitHub](https://github.com/aws/aws-lambda-builders)         | [SO](https://stackoverflow.com/collectives/aws)              | [r/aws](https://www.reddit.com/r/aws/)                       |
| **Azure Container Apps**           |                                                              | [Forum](https://learn.microsoft.com/en-us/answers/tags/553/azure-container-apps) | [GitHub](https://github.com/microsoft/azure-container-apps)  | [SO](https://stackoverflow.com/questions/tagged/azure-container-apps) | [r/AZURE](https://www.reddit.com/r/AZURE/)                   |
| **Azure Functions**                | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/3) | [Forum](https://learn.microsoft.com/en-us/answers/tags/87/azure-functions) | [GitHub](https://github.com/Azure/azure-sdk-for-python)      | [SO](https://stackoverflow.com/collectives/azure)            | [r/AZURE](https://www.reddit.com/r/AZURE/)                   |
| **Cloudflare Workers**             | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/10) | [Forum](https://community.cloudflare.com/c/developers/workers/40) | [GitHub](https://github.com/cloudflare/workers-py)           | [SO](https://stackoverflow.com/questions/tagged/cloudflare-workers) | [r/CloudFlare](https://www.reddit.com/r/CloudFlare/)         |
| **DigitalOcean Functions**         |                                                              | [Community](https://www.digitalocean.com/community)          | [GitHub](https://github.com/digitalocean/doctl)              | [SO](https://stackoverflow.com/questions/tagged/digital-ocean) |                                                              |
| **Fermyon**                        | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/4) | [Discord](https://discord.com/invite/P4Cx7xUbJu)             | [Spin (CNCF)](https://github.com/spinframework/spin)         | [SO](https://stackoverflow.com/questions/tagged/fermyon-spin) |                                                              |
| **Fly.io**                         | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/5) | [Forum](https://community.fly.io/)                           |                                                              | [SO](https://stackoverflow.com/questions/tagged/fly?tab=Active) |                                                              |
| **Google Cloud Run**               | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/6) | [Group](https://groups.google.com/g/firebase-talk/)          | [GitHub](https://github.com/firebase/firebase-functions-python) | [SO](https://stackoverflow.com/collectives/google-cloud)     | [r/Firebase](https://www.reddit.com/r/Firebase/) and [r/googlecloud](https://www.reddit.com/r/googlecloud/) |
| **Huawei Cloud FunctionGraph**     |                                                              | [Forum](https://developer.huaweicloud.com/intl/en-us/forum/) | [GitHub](https://github.com/huaweicloud)                     | [SO](https://stackoverflow.com/questions/tagged/huawei-cloud) |                                                              |
| **IBM Code Engine**                | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/7) | [Slack](https://cloud.ibm.com/kubernetes/slack)              | [GitHub](https://github.com/IBM/CodeEngine)                  | [SO](https://stackoverflow.com/questions/tagged/ibm-cloud-code-engine) |                                                              |
| **Koyeb**                          |                                                              | [Community](https://community.koyeb.com)                     |                                                              |                                                              |                                                              |
| **Modal**                          |                                                              | [Slack](https://modal.com/slack)                             | [GitHub](https://github.com/modal-labs/modal-client)         |                                                              |                                                              |
| **Oracle (OCI) Functions**         | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/8) | [Forum](https://forums.oracle.com/ords/apexds/domain/dev-community/category/containers-cloud-native) | [GitHub](https://github.com/oracle-samples/oracle-functions-samples) | [SO](https://stackoverflow.com/questions/tagged/oracle-cloud-functions) | [r/oraclecloud](https://www.reddit.com/r/oraclecloud/)       |
| **Railway**                        |                                                              | [Central Station](https://station.railway.com)               | [GitHub](https://github.com/railwayapp/cli)                  |                                                              |                                                              |
| **Render**                         |                                                              | [Discord](https://render.com/docs/community) (forum retired Mar 2026) | [Feedback](https://feedback.render.com)                      |                                                              |                                                              |
| **Scaleway Serverless Functions**  |                                                              | [Slack](https://slack.scaleway.com)                          | [GitHub](https://github.com/scaleway/serverless-functions-python) | [SO](https://stackoverflow.com/questions/tagged/scaleway)    |                                                              |
| **Vercel Functions**               | [Link](https://github.com/hbmartin/comparison-hosts-serverless-cloud-function-faas-for-python/discussions/9) | [Help](https://vercel.com/help)                              | [GitHub](https://github.com/orgs/vercel/discussions)         | [SO](https://stackoverflow.com/questions/tagged/vercel)      | [r/Vercel](https://www.reddit.com/r/vercel/)                 |
| **Yandex Cloud Functions**         |                                                              |                                                              | [GitHub](https://github.com/yandex-cloud-examples)           | [SO](https://stackoverflow.com/questions/tagged/yandex-cloud) |                                                              |

## References and Useful Links

All sources below were used to verify the data above and were accessed on July 12, 2026. Sources cited directly by table cells appear as numbered footnotes at the very bottom of this page.

### Alibaba Cloud Function Compute

- [Python runtimes (FC 3.0)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/python/)
- [Billing overview (CU model)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/billing-overview-of-fc) · [Pricing page](https://www.alibabacloud.com/en/product/function-compute/pricing) · [Trial quota](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/trial-quota-1)
- [Limits of usage](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/limits-of-usage) · [Synchronous invocations](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/synchronous-invocations)
- [CDT egress free quota](https://www.alibabacloud.com/help/en/cdt/product-overview/cdt-public-network-traffic-free-quota-increased-from-20gb-month-to-200gb-month) · [Serverless Devs](https://www.alibabacloud.com/help/en/fc/developer-reference/what-is-serverless-devs)

### Appwrite Functions

- [Function runtimes](https://appwrite.io/docs/products/functions/runtimes) · [Developing functions](https://appwrite.io/docs/products/functions/develop) · [Local development](https://appwrite.io/docs/products/functions/develop-locally) · [Execution & timeouts](https://appwrite.io/docs/products/functions/execute) · [Quick start](https://appwrite.io/docs/products/functions/quick-start)
- [Pricing](https://appwrite.io/pricing) · [Compute billing (GB-hours)](https://appwrite.io/docs/advanced/billing/compute) · [Budget cap](https://appwrite.io/docs/advanced/billing/payments#budget-cap) · [Cloud GA (Aug 2025)](https://appwrite.io/changelog/entry/2025-08-06) · [Sept 2025 repricing](https://appwrite.io/changelog/entry/2025-08-08) · [Free-project pausing (Feb 2026)](https://appwrite.io/changelog/entry/2026-02-20-1) · [Paused-project deletion (Jun 2026)](https://appwrite.io/changelog/entry/2026-06-29) · [Sites](https://appwrite.io/docs/products/sites)

### AWS Lambda / Lambda@Edge

- [Lambda pricing](https://aws.amazon.com/lambda/pricing/) · [Lambda quotas/limits](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) · [Runtimes & deprecation schedule](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [Python 3.14 runtime announcement (Nov 2025)](https://aws.amazon.com/blogs/compute/python-3-14-runtime-now-available-in-aws-lambda/) · [Python in Lambda](https://docs.aws.amazon.com/lambda/latest/dg/lambda-python.html) · [Packaging Python dependencies](https://docs.aws.amazon.com/lambda/latest/dg/python-package.html)
- [INIT-phase billing change (Aug 2025)](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/) · [Free Tier restructure (Jul 2025)](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/) · [Data transfer charges](https://aws.amazon.com/blogs/apn/aws-data-transfer-charges-for-server-and-serverless-architectures/)
- [Lambda memory & vCPU allocation](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html) · [Lambda@Edge restrictions](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-edge-function-restrictions.html) · [CloudFront quotas](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html) · [Lambda@Edge generated responses](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-generating-http-responses.html)

### Azure Container Apps

- [Overview](https://learn.microsoft.com/en-us/azure/container-apps/overview) · [Pricing page](https://azure.microsoft.com/en-us/pricing/details/container-apps/) · [Billing](https://learn.microsoft.com/en-us/azure/container-apps/billing) · [Containers (CPU/memory sizes)](https://learn.microsoft.com/en-us/azure/container-apps/containers)
- [Scaling (0–1,000 replicas)](https://learn.microsoft.com/en-us/azure/container-apps/scale-app) · [Ingress timeouts](https://learn.microsoft.com/en-us/azure/container-apps/ingress-overview) · [Cold start](https://learn.microsoft.com/en-us/azure/container-apps/cold-start) · [Serverless GPUs GA (Mar 2025)](https://techcommunity.microsoft.com/blog/appsonazureblog/announcing-ga-for-azure-container-apps-serverless-gpus/4394302) · [Azure Functions on Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/functions-overview) · [Quickstart (deploy an image)](https://learn.microsoft.com/en-us/azure/container-apps/get-started-existing-container-image)

### Azure Functions

- [Supported languages/versions](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages) · [Python developer reference](https://learn.microsoft.com/en-us/azure/azure-functions/functions-reference-python)
- [Functions pricing](https://azure.microsoft.com/en-us/pricing/details/functions/) · [Consumption plan costs](https://learn.microsoft.com/en-us/azure/azure-functions/functions-consumption-costs) · [Bandwidth pricing](https://azure.microsoft.com/en-us/pricing/details/bandwidth/)
- [Flex Consumption plan](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan) · [Flex GA announcement (Nov 2024)](https://techcommunity.microsoft.com/blog/appsonazureblog/azure-functions-flex-consumption-is-now-generally-available/4298778) · [Scale & hosting limits](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale)
- [Linux Consumption retirement (Sept 2028)](https://learn.microsoft.com/en-us/azure/azure-functions/consumption-plan) · [Run from deployment package (1 GB)](https://learn.microsoft.com/en-us/azure/azure-functions/run-functions-from-deployment-package) · [Azure spending limit](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/spending-limit)

### Cloudflare Workers (Python)

- [Python Workers docs (beta)](https://developers.cloudflare.com/workers/languages/python/) · [How Python Workers work](https://developers.cloudflare.com/workers/languages/python/how-python-workers-work/) · [Packages](https://developers.cloudflare.com/workers/languages/python/packages/)
- [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/) · [Workers limits](https://developers.cloudflare.com/workers/platform/limits/)
- ["Python Workers redux" (Dec 2025)](https://blog.cloudflare.com/python-workers-advancements/) · [Cold-start changelog with cross-vendor numbers (Dec 2025)](https://developers.cloudflare.com/changelog/2025-12-08-python-cold-start-improvements/) · [workerd Python metadata (Pyodide/CPython versions)](https://github.com/cloudflare/workerd/blob/main/build/python_metadata.bzl)
- [Cloudflare Containers GA (Apr 2026)](https://developers.cloudflare.com/changelog/post/2026-04-13-containers-sandbox-ga/) · [Containers pricing](https://developers.cloudflare.com/containers/pricing/)

### DigitalOcean Functions

- [Python runtime reference](https://docs.digitalocean.com/products/functions/reference/runtimes/python/) · [Pricing](https://docs.digitalocean.com/products/functions/details/pricing/) · [Limits](https://docs.digitalocean.com/products/functions/details/limits/) · [Quickstart](https://docs.digitalocean.com/products/functions/quickstart/)
- [Async functions (30 s sync conversion)](https://docs.digitalocean.com/products/functions/how-to/async-functions/) · [doctl serverless reference](https://docs.digitalocean.com/reference/doctl/reference/serverless/) · [Billing alerts](https://docs.digitalocean.com/platform/billing/billing-alerts/)

### Fermyon / Spin

- [Spin Python components](https://spinframework.dev/v3/python-components) · [spin-python-sdk](https://github.com/spinframework/spin-python-sdk) · [Spin in CNCF Sandbox (Jan 2025)](https://www.cncf.io/projects/spin/)
- [Fermyon Cloud pricing & billing](https://developer.fermyon.com/cloud/pricing-and-billing) · [Cloud FAQ (quotas/limits)](https://developer.fermyon.com/cloud/faq) · [Fermyon Wasm Functions FAQ (quotas)](https://developer.fermyon.com/wasm-functions/faq)
- [Akamai acquires Fermyon (Dec 2025)](https://www.akamai.com/newsroom/press-release/akamai-announces-acquisition-of-function-as-a-service-company-fermyon) · [Fermyon Wasm Functions GA (Nov 2025)](https://www.globenewswire.com/news-release/2025/11/12/3186327/0/en/Fermyon-Wasm-Functions-on-Akamai-Now-Generally-Available-Scales-to-75-Million-RPS.html) · [Fermyon Wasm Functions](https://www.fermyon.com/wasm-functions)

### Fly.io

- [Pricing](https://fly.io/docs/about/pricing/) · [Free trial](https://fly.io/docs/about/free-trial/) · [Billing (machine states)](https://fly.io/docs/about/billing/)
- [Python on Fly](https://fly.io/docs/python/) · [Autostop/autostart (scale to zero)](https://fly.io/docs/launch/autostop-autostart/) · [Machine sizing](https://fly.io/docs/machines/guides-examples/machine-sizing/) · [GPU deprecation](https://fly.io/docs/gpus/) · [Managed Postgres](https://fly.io/docs/mpg/)

### Google Cloud Run

- [Cloud Run pricing](https://cloud.google.com/run/pricing) · [Quotas & limits](https://cloud.google.com/run/quotas) · [Python runtime](https://docs.cloud.google.com/run/docs/runtimes/python) · [Python dependencies](https://cloud.google.com/run/docs/runtimes/python-dependencies)
- [Cloud Run functions release notes (Python 3.14, uv)](https://docs.cloud.google.com/functions/docs/release-notes) · [Runtime support](https://docs.cloud.google.com/functions/docs/runtime-support) · [Request timeout](https://cloud.google.com/run/docs/configuring/request-timeout) · [Concurrency](https://cloud.google.com/run/docs/about-concurrency) · [Functions Framework for Python](https://github.com/GoogleCloudPlatform/functions-framework-python)

### Huawei Cloud FunctionGraph

- [Supported runtimes (FAQ)](https://support.huaweicloud.com/intl/en-us/functiongraph_faq/functiongraph_03_0260.html) · [Free tier](https://support.huaweicloud.com/intl/en-us/price-functiongraph/functiongraph_00_0012.html) · [Billing overview](https://support.huaweicloud.com/intl/en-us/price-functiongraph/functiongraph_00_0001.html) · [China list prices](https://www.huaweicloud.com/zhishi/price-FunctionGraph.html)
- [Notes & constraints (limits)](https://support.huaweicloud.com/intl/en-us/productdesc-functiongraph/functiongraph_01_0150.html) · [Memory configuration](https://support.huaweicloud.com/intl/en-us/usermanual-functiongraph/functiongraph_01_1828.html) · [Quotas](https://support.huaweicloud.com/intl/en-us/usermanual-functiongraph/functiongraph_01_0303.html) · [Timeout FAQ](https://support.huaweicloud.com/intl/en-us/functiongraph_faq/functiongraph_03_0250.html)
- [HTTP functions](https://support.huaweicloud.com/intl/en-us/usermanual-functiongraph/functiongraph_01_1442.html) · [Dependency packages](https://support.huaweicloud.com/intl/en-us/devg-functiongraph/functiongraph_02_0616.html) · [VS Code plugin](https://support.huaweicloud.com/intl/en-us/devg-functiongraph/functiongraph_01_1831.html) · [GPU functions](https://support.huaweicloud.com/intl/en-us/productdesc-functiongraph/functiongraph_02_1003.html) · [Python quickstart](https://support.huaweicloud.com/intl/en-us/qs-functiongraph/functiongraph_04_0101.html) · [Global regions](https://www.huaweicloud.com/intl/en-us/about/global-infrastructure.html)

### IBM Code Engine

- [Function runtimes (Python versions & lifecycle)](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime) · [Working with functions](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-work) · [Limits](https://cloud.ibm.com/docs/codeengine?topic=codeengine-limits) · [Pricing](https://cloud.ibm.com/docs/codeengine?topic=codeengine-pricing)
- [IBM Cloud Functions (OpenWhisk) deprecation](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-dep-overview)

### Koyeb

- [Pricing](https://www.koyeb.com/pricing) · [Instances (incl. free)](https://www.koyeb.com/docs/reference/instances) · [Scale to zero](https://www.koyeb.com/docs/run-and-scale/scale-to-zero) · [Light Sleep](https://www.koyeb.com/blog/avoid-cold-starts-with-scale-to-zero-light-sleep)
- [Koyeb joins Mistral AI (Feb 2026)](https://www.koyeb.com/blog/koyeb-is-joining-mistral-ai-to-build-the-future-of-ai-infrastructure) · [Serverless Postgres GA (May 2025)](https://www.koyeb.com/blog/serverless-postgres-ga-production-ready-databases-for-large-scale-and-ai-apps)

### Modal

- [Pricing](https://modal.com/pricing) · [Guide](https://modal.com/docs/guide) · [Images (Python versions)](https://modal.com/docs/guide/images) · [Timeouts](https://modal.com/docs/guide/timeouts) · [Resources (CPU/memory)](https://modal.com/docs/guide/resources) · [Web endpoint payloads](https://modal.com/docs/guide/webhooks) · [Scale](https://modal.com/docs/guide/scale)

### Oracle (OCI) Functions

- [OCI price list (Functions section)](https://www.oracle.com/cloud/price-list/) · [Functions product page](https://www.oracle.com/cloud/cloud-native/functions/)
- [Customizing functions (memory/timeout)](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscustomizing.htm) · [3 GB memory support](https://docs.public.content.oci.oraclecloud.com/en-us/iaas/releasenotes/functions/functions-3gb-functions-support.htm) · [Long-running functions (Oct 2025)](https://docs.oracle.com/en-us/iaas/releasenotes/functions/functions-long-running-functions.htm)
- [Concurrency monitoring](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsmonitoringcapacityusage_topic-Monitoring-concurrent-function-execution.htm) · [Python FDK](https://github.com/fnproject/fdk-python)

### Railway

- [Plans & pricing](https://docs.railway.com/pricing/plans) · [Free plan announcement (Aug 2025)](https://blog.railway.com/p/free-plan) · [Cost control (hard limits)](https://docs.railway.com/pricing/cost-control) · [Serverless / app sleeping](https://docs.railway.com/deployments/serverless)
- [Railpack Python](https://railpack.com/languages/python) · [Networking limits](https://docs.railway.com/networking/public-networking/specs-and-limits) · [Scaling](https://docs.railway.com/deployments/scaling) · [Storage Buckets](https://docs.railway.com/storage-buckets) · [Databases (unmanaged templates)](https://docs.railway.com/databases) · [Railway Metal](https://docs.railway.com/platform/railway-metal) · [FastAPI guide](https://docs.railway.com/guides/fastapi)

### Render

- [Free instances](https://render.com/docs/free) · [Pricing](https://render.com/pricing) · [New workspace plans (2026)](https://render.com/docs/new-workspace-plans) · [Workspace plans changelog (Apr 2026)](https://render.com/changelog/updated-plans-for-render-workspaces) · [Python version](https://render.com/docs/python-version) · [Scaling](https://render.com/docs/scaling) · [Outbound bandwidth](https://render.com/docs/outbound-bandwidth) · [Community (Discord)](https://render.com/docs/community)

### Scaleway Serverless Functions

- [Runtimes](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-runtimes/) · [FAQ (free tier & pricing)](https://www.scaleway.com/en/docs/serverless-functions/faq/) · [Serverless pricing](https://www.scaleway.com/en/pricing/serverless/) · [June 2026 pricing update](https://www.scaleway.com/en/blog/a-transparent-update-on-scaleway-pricing/)
- [Limitations](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-limitations/) · [Memory/CPU tiers](https://www.scaleway.com/en/docs/serverless-functions/reference-content/available-memory-and-cpu-tiers/) · [Local testing framework](https://github.com/scaleway/serverless-functions-python) · [Billing alerts](https://www.scaleway.com/en/docs/billing/how-to/use-billing-alerts/)

### Tencent Cloud Functions

- [Python runtime (3.10 max)](https://cloud.tencent.com/document/product/583/55592) · [Quotas & limits](https://www.tencentcloud.com/document/product/583/11637) · [Billing overview](https://www.tencentcloud.com/document/product/583/44254) · [Pricing](https://www.tencentcloud.com/document/product/583/12281) · [Free tier (trial)](https://www.tencentcloud.com/document/product/583/12282)
- [Basic package cancellation (Apr 2024)](https://cloud.tencent.com.cn/document/product/583/104909) · [API Gateway trigger EOL](https://cloud.tencent.com.cn/document/product/583/107631)

### Vercel Functions

- [Python runtime (beta)](https://vercel.com/docs/functions/runtimes/python) · [Python 3.13/3.14 changelog (Feb 2026)](https://vercel.com/changelog/python-3-13-and-3-14-are-now-available)
- [Usage & pricing](https://vercel.com/docs/functions/usage-and-pricing) · [Active CPU pricing announcement](https://vercel.com/blog/introducing-active-cpu-pricing-for-fluid-compute) · [Fluid compute](https://vercel.com/docs/fluid-compute) · [Limits](https://vercel.com/docs/limits) · [Hobby plan](https://vercel.com/docs/plans/hobby) · [Pro plan](https://vercel.com/docs/plans/pro-plan) · [Per-unit invocation billing (May 2026)](https://vercel.com/changelog/function-invocations-now-billed-per-unit) · [Pro usage credit (Sept 2025)](https://vercel.com/changelog/included-pro-usage-is-now-credit-based)
- [Duration limits](https://vercel.com/docs/functions/configuring-functions/duration) · [Memory/CPU](https://vercel.com/docs/functions/configuring-functions/memory) · [Function limitations (payload)](https://vercel.com/docs/functions/limitations) · [Concurrency scaling](https://vercel.com/docs/functions/concurrency-scaling) · [500 MB Python bundle changelog](https://vercel.com/changelog/python-vercel-functions-bundle-size-limit-increased-to-500mb)

### Yandex Cloud Functions

- [Runtimes](https://yandex.cloud/en/docs/functions/concepts/runtime/) · [Python handler](https://yandex.cloud/en/docs/functions/lang/python/handler) · [Dependencies (auto pip install)](https://yandex.cloud/en/docs/functions/lang/python/dependencies) · [Python quickstart](https://yandex.cloud/en/docs/functions/quickstart/create-function/python-function-quickstart)
- [Pricing](https://yandex.cloud/en/docs/functions/pricing) · [Serverless free tier](https://yandex.cloud/en/docs/billing/concepts/serverless-free-tier) · [Limits & quotas](https://yandex.cloud/en/docs/functions/concepts/limits) · [Long-lived functions (1 h)](https://yandex.cloud/en/docs/functions/concepts/long-lived-functions) · [Budgets](https://yandex.cloud/en/docs/billing/concepts/budget) · [Regions](https://yandex.cloud/en/docs/overview/concepts/region) · [Nebius divestment (Jul 2024)](https://nebius.com/newsroom/ynv-announces-successful-completion-of-the-divestment-of-its-russia-based-businesses)

### Watchlist platforms

- [Wasmer Edge pricing](https://wasmer.io/pricing) · [Wasmer "Python on the Edge" (Sept 2025)](https://wasmer.io/posts/python-on-the-edge-powered-by-webassembly) · [Wasmer Edge architecture](https://docs.wasmer.io/edge/architecture) · [Flask on Wasmer Edge guide](https://docs.wasmer.io/edge/guides/python-flask-server)
- [Leapcell docs](https://docs.leapcell.io/) · [Leapcell pricing](https://leapcell.io/pricing) · [Sevalla application pricing](https://docs.sevalla.com/billing/application-pricing) · [Sevalla Hibernation](https://sevalla.com/blog/shipping-hibernation-in-3-days/) · [Blaxel](https://blaxel.ai/)

### Excluded platforms

- [Netlify Functions (JS/TS, no Python)](https://docs.netlify.com/build/functions/get-started/) · [Supabase Edge Functions (Deno)](https://supabase.com/docs/guides/functions) · [Deno Deploy (JS/TS)](https://docs.deno.com/runtime/deploy/) · [AWS App Runner pricing (no scale-to-zero cost)](https://aws.amazon.com/apprunner/pricing/) · [AWS App Runner closed to new customers (Apr 2026)](https://docs.aws.amazon.com/apprunner/latest/dg/apprunner-availability-change.html) · [IBM Cloud Functions EOL](https://cloud.ibm.com/docs/openwhisk?topic=openwhisk-dep-overview)
- [Heroku Eco dyno hours (flat subscription)](https://devcenter.heroku.com/articles/eco-dyno-hours) · [Heroku generations (Fir has no Eco)](https://devcenter.heroku.com/articles/generations) · [Northflank scaling (no wake-on-request)](https://northflank.com/docs/v1/application/scale/scale-instances) · [Zeabur shared-cluster phase-out (Apr 2026)](https://zeabur.com/changelogs/phasing-out-shared-cluster) · [PythonAnywhere pricing (always-on)](https://www.pythonanywhere.com/pricing/) · [Genezio 2025 recap (pivot away from FaaS)](https://genezio.com/blog/2025-recap-2026-roadmap/) · [Deta Space shutdown discussion (Oct 2024)](https://news.ycombinator.com/item?id=41426388)

### Other comparisons

- [Comparison by Brecht De Rooms from Feb 6th, 2020](https://fauna.com/blog/comparison-faas-providers)

[^ali-py]: [Alibaba FC 3.0 Python runtimes](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/python/)
[^ali-trial]: [Alibaba FC trial quota — 150,000 CUs/mo for 3 months](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/trial-quota-1)
[^ali-cdt]: [CDT public-network free quota raised to 200 GB/mo (plus 20 GB CN)](https://www.alibabacloud.com/help/en/cdt/product-overview/cdt-public-network-traffic-free-quota-increased-from-20gb-month-to-200gb-month)
[^ali-price]: [Alibaba Function Compute pricing — tiered CU billing](https://www.alibabacloud.com/en/product/function-compute/pricing)
[^ali-limits]: [Alibaba FC 3.0 limits of usage](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/product-overview/limits-of-usage)
[^ali-payload]: [Alibaba FC synchronous invocations (32 MB) and async (128 KB)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/synchronous-invocations)
[^ali-inst]: [Alibaba FC max on-demand instances (default 100/region)](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/overview-of-configuring-the-maximum-number-of-on-demand-instances)
[^ali-cap]: No hard spending cap exists; Alibaba's cost-containment mechanism is [capping maximum on-demand instances](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/user-guide/overview-of-configuring-the-maximum-number-of-on-demand-instances).
[^ali-createfn]: [FC 3.0 CreateFunction API — memory 128–32,768 MB (no documented default; the console pre-fills 512 MB), timeout default 3 s / max 86,400 s](https://www.alibabacloud.com/help/en/functioncompute/fc-3-0/developer-reference/api-fc-2023-03-30-createfunction)
[^appwrite-py]: [Appwrite function runtimes — Appwrite Cloud offers python-3.9/3.12/3.14 and python-ml-3.11](https://appwrite.io/docs/products/functions/runtimes); [Appwrite Cloud GA (Aug 6, 2025)](https://appwrite.io/changelog/entry/2025-08-06)
[^appwrite-dev]: [Developing Appwrite functions — `def main(context)`, dependencies via a requirements.txt build command](https://appwrite.io/docs/products/functions/develop)
[^appwrite-free]: [Appwrite pricing — Free plan includes 750k executions, 100 GB-hours, 5 GB bandwidth/mo](https://appwrite.io/pricing); [free projects pause after 1 week of inactivity (Feb 2026)](https://appwrite.io/changelog/entry/2026-02-20-1) and [paused free projects are deleted after 90 days (Jun 2026)](https://appwrite.io/changelog/entry/2026-06-29)
[^appwrite-cap]: [Appwrite budget caps limit add-on spend on paid plans, with alerts from 75%](https://appwrite.io/docs/advanced/billing/payments#budget-cap)
[^appwrite-price]: [Appwrite pricing — Pro $25/mo/project; overages $2 per 1M executions, $0.06 per GB-hour, $15 per 100 GB bandwidth](https://appwrite.io/pricing); [Sept 2025 repricing details](https://appwrite.io/changelog/entry/2025-08-08)
[^appwrite-limits]: [Appwrite compute tiers — 0.5 CPU / 512 MB on Free (fixed), configurable up to 4 CPU / 4 GB on paid plans](https://appwrite.io/docs/advanced/billing/compute)
[^appwrite-timeout]: [Appwrite function timeout — system-wide maximum 900 s](https://appwrite.io/docs/products/functions/functions); [synchronous (HTTP) executions are capped at 30 s](https://appwrite.io/docs/products/functions/execute)
[^appwrite-db]: [Appwrite TablesDB — document-style API backed by SQL (MariaDB)](https://appwrite.io/docs/products/databases/tables)
[^appwrite-sites]: [Appwrite Sites — static & SSR hosting (launched May 2025)](https://appwrite.io/docs/products/sites)
[^aws-py]: [Python 3.14 runtime now available in AWS Lambda (Nov 18, 2025)](https://aws.amazon.com/blogs/compute/python-3-14-runtime-now-available-in-aws-lambda/); [runtime list & deprecation schedule](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
[^aws-free]: [AWS Lambda pricing — always-free tier of 1M requests + 400,000 GB-s/mo](https://aws.amazon.com/lambda/pricing/), retained under the [July 2025 credits-based free plan](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/)
[^aws-egress]: [100 GB/mo aggregate free data transfer out, then ~$0.09/GB (first 10 TB, us-east-1)](https://aws.amazon.com/blogs/aws/aws-free-tier-data-transfer-expansion-100-gb-from-regions-and-1-tb-from-amazon-cloudfront-per-month/); see also [data transfer charges for serverless](https://aws.amazon.com/blogs/apn/aws-data-transfer-charges-for-server-and-serverless-architectures/)
[^aws-budget]: [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) sends alerts (and can trigger actions) but AWS has no account-wide hard spending cap.
[^aws-price]: [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/) — GB-s rates are tiered, dropping above 6B GB-s/mo (x86); rates re-verified against the AWS Price List API publication of 2026-07-09.
[^aws-init]: [AWS Lambda standardizes billing for the INIT phase, effective Aug 1, 2025](https://aws.amazon.com/blogs/compute/aws-lambda-standardizes-billing-for-init-phase/)
[^aws-limits]: [AWS Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html); [container images up to 10 GB](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html); [timeout configuration](https://docs.aws.amazon.com/lambda/latest/dg/configuration-timeout.html)
[^aws-vcpu]: ["At 1,769 MB, a function has the equivalent of one vCPU"](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html)
[^edge-price]: [Lambda@Edge pricing — $0.60/1M requests, $0.00005001/GB-s, metered at 1 ms granularity, excluded from the Lambda free tier](https://aws.amazon.com/lambda/pricing/)
[^edge-limits]: [CloudFront quotas on Lambda@Edge — 128 MB viewer / 3,008 MB origin memory, 5 s viewer / 30 s origin timeout, 1 MB viewer / 50 MB origin package size](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-limits.html)
[^edge-body]: [Request bodies exposed to the function are truncated at 40 KB (viewer) / 1 MB (origin); generated responses are capped at the same sizes](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-at-edge-function-restrictions.html); see also [generated-response size limits](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-generating-http-responses.html)
[^azure-py]: [Azure Functions supported languages — Python 3.14 GA (as of early July 2026; EOL Oct 2030)](https://learn.microsoft.com/en-us/azure/azure-functions/supported-languages)
[^azure-asgi]: [AsgiFunctionApp / WsgiFunctionApp in the azure-functions Python library](https://learn.microsoft.com/en-us/python/api/azure-functions/azure.functions.asgifunctionapp)
[^azure-price]: [Azure Functions pricing — Consumption and Flex Consumption rates and monthly free grants](https://azure.microsoft.com/en-us/pricing/details/functions/)
[^azure-egress]: [Azure bandwidth pricing — first 100 GB/mo free, then $0.087/GB (Zone 1)](https://azure.microsoft.com/en-us/pricing/details/bandwidth/)
[^azure-cap]: [Azure spending limit — on by default for credit-based subscriptions only; not available on pay-as-you-go](https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/spending-limit)
[^azure-limits]: [Azure Functions service limits — 1.5 GB Consumption memory, 210 MB max request size, timeouts per plan](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale#service-limits)
[^azure-flex]: [Flex Consumption plan — 512 MB / 2,048 MB / 4,096 MB instance sizes, scale to 1,000 instances](https://learn.microsoft.com/en-us/azure/azure-functions/flex-consumption-plan)
[^azure-pkg]: [Maximum deployment package size is 1 GB (run-from-package)](https://learn.microsoft.com/en-us/azure/azure-functions/run-functions-from-deployment-package)
[^aca-price]: [Azure Container Apps pricing — Consumption: first 180k vCPU-s + 360k GiB-s + 2M requests/mo free, then $0.000024/vCPU-s active ($0.000003 idle), $0.000003/GiB-s, $0.40 per additional 1M requests](https://azure.microsoft.com/en-us/pricing/details/container-apps/); [billing docs (idle rates, scale-to-zero = no charges)](https://learn.microsoft.com/en-us/azure/container-apps/billing)
[^aca-limits]: [Container Apps CPU/memory — up to 4 vCPU / 8 GiB per replica on Consumption (workload-profiles environments; 2 vCPU / 4 GiB in legacy consumption-only environments)](https://learn.microsoft.com/en-us/azure/container-apps/containers)
[^aca-ingress]: [Container Apps ingress — 240 s default request timeout](https://learn.microsoft.com/en-us/azure/container-apps/ingress-overview); [premium ingress allows a 4–30 min idle request timeout](https://learn.microsoft.com/en-us/azure/container-apps/ingress-environment-configuration)
[^aca-scale]: [Container Apps scale rules — 0–1,000 replicas, default max 10; scaled-to-zero apps incur no usage charges](https://learn.microsoft.com/en-us/azure/container-apps/scale-app)
[^aca-cold]: [Microsoft's cold-start doc describes the pull/provision/start delay without an official figure](https://learn.microsoft.com/en-us/azure/container-apps/cold-start); [Microsoft community guidance reports ~15–30 s for heavier images](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/performance-tuning-cold-starts-scaling-delays-and-startup-latency-in-azure-conta/4518523)
[^aca-gpu]: [Serverless GPUs (A100/T4) GA on Azure Container Apps with scale to zero (Mar 2025)](https://techcommunity.microsoft.com/blog/appsonazureblog/announcing-ga-for-azure-container-apps-serverless-gpus/4394302)
[^cf-py]: [How Python Workers work — Pyodide-based](https://developers.cloudflare.com/workers/languages/python/how-python-workers-work/); released bundle is Pyodide 0.28 / CPython 3.13 per [workerd python_metadata.bzl](https://github.com/cloudflare/workerd/blob/main/build/python_metadata.bzl)
[^cf-beta]: [Python Workers are in open beta](https://developers.cloudflare.com/workers/languages/python/)
[^cf-price]: [Workers pricing — free 100k reqs/day + 10 ms CPU; paid $5/mo incl. 10M reqs + 30M CPU-ms; duration and egress unbilled](https://developers.cloudflare.com/workers/platform/pricing/)
[^cf-cap]: [No spend cap exists; Cloudflare's cost-control mechanism is per-invocation CPU limits](https://developers.cloudflare.com/workers/wrangler/configuration/#limits)
[^cf-limits]: [Workers limits — 128 MB memory, 5 min max CPU (30 s default), 3/10 MB gzip'd script size, 100/500 Workers](https://developers.cloudflare.com/workers/platform/limits/)
[^cf-body]: [Request-body limit follows the zone's Cloudflare plan: 100 MB Free/Pro, 200 MB Business, 500 MB Enterprise default](https://developers.cloudflare.com/workers/platform/limits/)
[^cf-cold]: [Cloudflare's Dec 2025 cold-start changelog — mean cold starts with httpx+FastAPI+Pydantic: Cloudflare 1,027 ms vs AWS Lambda 2,502 ms vs Google Cloud Run 3,069 ms (vendor-run benchmark)](https://developers.cloudflare.com/changelog/2025-12-08-python-cold-start-improvements/)
[^do-py]: [DigitalOcean Functions Python runtime reference — python 3.9/3.11/3.12/3.13, `__main__.py`, requirements.txt + build.sh](https://docs.digitalocean.com/products/functions/reference/runtimes/python/)
[^do-local]: [doctl serverless workflow uses a connected cloud namespace; there is no offline emulator](https://docs.digitalocean.com/reference/doctl/reference/serverless/)
[^do-price]: [DigitalOcean Functions pricing — 90,000 GiB-s/mo free, $0.0000185/GiB-s, no per-invocation fee, 100 ms minimum](https://docs.digitalocean.com/products/functions/details/pricing/)
[^do-cap]: [DigitalOcean billing alerts notify but cannot cap usage](https://docs.digitalocean.com/platform/billing/billing-alerts/)
[^do-limits]: [DigitalOcean Functions limits](https://docs.digitalocean.com/products/functions/details/limits/)
[^do-async]: [Synchronous invocations are converted to asynchronous after 30 seconds](https://docs.digitalocean.com/products/functions/how-to/async-functions/)
[^fermyon-py]: [spin-python-sdk requires Python 3.10+ (componentize-py)](https://github.com/spinframework/spin-python-sdk)
[^fermyon-beta]: [Fermyon Cloud is an open beta; the Starter plan is free without expiration](https://developer.fermyon.com/cloud/pricing-and-billing). The GA sibling is [Fermyon Wasm Functions on Akamai](https://www.fermyon.com/wasm-functions).
[^fermyon-quota]: [Fermyon Cloud FAQ quotas — 100k requests + 5 GB egress/mo free, 30 s handler duration, 10 MB HTTP body, 100 MB app package, 1,000 req/s](https://developer.fermyon.com/cloud/faq)
[^fermyon-price]: [Fermyon Cloud plans — Growth $19.38/mo with 1M requests + 50 GB egress](https://developer.fermyon.com/cloud/pricing-and-billing)
[^fermyon-fwf]: [Fermyon Wasm Functions FAQ — 128 MiB default memory per instance, increasable on request](https://developer.fermyon.com/wasm-functions/faq)
[^fly-trial]: [Fly.io free trial — 2 hours of machine runtime or 7 days, whichever comes first; no ongoing free allowances since Oct 7, 2024](https://fly.io/docs/about/free-trial/)
[^fly-price]: [Fly.io pricing — per-second machine billing (region-dependent; shared-cpu-1x 256 MB from $1.94/mo), $0.02/GB NA/EU egress, $0.15/GB per 30 days stopped-machine rootfs](https://fly.io/docs/about/pricing/)
[^fly-size]: [Fly machine sizing — largest preset performance-16x; RAM up to 8 GB per performance vCPU (128 GB)](https://fly.io/docs/machines/guides-examples/machine-sizing/)
[^fly-mpg]: [Fly.io Managed Postgres](https://fly.io/docs/mpg/)
[^gcr-py]: [Cloud Run functions runtime support — Python 3.14 GA](https://docs.cloud.google.com/functions/docs/runtime-support); [uv is the default installer from 3.14 onward](https://docs.cloud.google.com/run/docs/runtimes/python)
[^gcr-deps]: [Python dependencies on Cloud Run — requirements.txt or pyproject.toml (GA July 2026)](https://cloud.google.com/run/docs/runtimes/python-dependencies)
[^gcr-price]: [Cloud Run pricing — request-based billing rates and always-free tier (us-central1)](https://cloud.google.com/run/pricing)
[^gcr-budget]: [Google Cloud budgets alert but do not cap spending; billing can be disabled programmatically via budget notifications](https://cloud.google.com/billing/docs/how-to/budgets)
[^gcr-mem]: [Cloud Run memory limits — 512 MiB default, 32 GiB max](https://cloud.google.com/run/docs/configuring/services/memory-limits)
[^gcr-timeout]: [Cloud Run request timeout — 5 min default, 60 min max](https://cloud.google.com/run/docs/configuring/request-timeout)
[^gcr-quota]: [Cloud Run quotas — 32 MiB request cap applies to HTTP/1 only; responses over 32 MiB must be streamed](https://cloud.google.com/run/quotas)
[^gcr-conc]: [Cloud Run concurrency — default 80×vCPUs for gcloud/Terraform deploys (80 for console), maximum 1,000](https://cloud.google.com/run/docs/about-concurrency)
[^huawei-py]: [FunctionGraph supported runtimes — Python up to 3.12](https://support.huaweicloud.com/intl/en-us/functiongraph_faq/functiongraph_03_0260.html)
[^huawei-free]: [FunctionGraph free tier — 1M requests + 400,000 GB-s per month, resets monthly (permanent, pay-per-use accounts)](https://support.huaweicloud.com/intl/en-us/price-functiongraph/functiongraph_00_0012.html)
[^huawei-cap]: [Huawei Cloud budgets are alert-only — "budgets cannot stop consumption"](https://support.huaweicloud.com/intl/en-us/cost_faq/cost_faq_00000022.html)
[^huawei-price]: [FunctionGraph billing overview — billed per request + GB-s duration; egress is billed by the attached network service (EIP/NAT/APIG), not FunctionGraph](https://support.huaweicloud.com/intl/en-us/price-functiongraph/functiongraph_00_0001.html); international USD rates are only exposed in the console price calculator — [China list prices](https://www.huaweicloud.com/zhishi/price-FunctionGraph.html) shown for reference
[^huawei-limits]: [FunctionGraph notes & constraints — memory 128 MB–10,240 MB, 6 MB sync request/response (256 KB async), code 40 MB console zip / 300 MB via OBS / 10 GB images](https://support.huaweicloud.com/intl/en-us/productdesc-functiongraph/functiongraph_01_0150.html); [memory options](https://support.huaweicloud.com/intl/en-us/usermanual-functiongraph/functiongraph_01_1828.html)
[^huawei-timeout]: [FunctionGraph timeout — default 3 s, configurable up to 259,200 s (72 h); synchronous invocations ≤900 s](https://support.huaweicloud.com/intl/en-us/functiongraph_faq/functiongraph_03_0250.html)
[^huawei-scale]: [FunctionGraph quotas — 400 functions and 1,000 concurrent on-demand instances per region by default](https://support.huaweicloud.com/intl/en-us/usermanual-functiongraph/functiongraph_01_0303.html)
[^huawei-gpu]: [GPU-accelerated FunctionGraph functions](https://support.huaweicloud.com/intl/en-us/productdesc-functiongraph/functiongraph_02_1003.html)
[^ibm-py]: [IBM Code Engine function runtimes — Python 3.13; 3.11 deprecated with EOL Oct 2026](https://cloud.ibm.com/docs/codeengine?topic=codeengine-fun-runtime)
[^ibm-price]: [IBM Code Engine pricing](https://cloud.ibm.com/docs/codeengine?topic=codeengine-pricing); [exact per-unit rates and free-tier scope in IBM's official cost scenario](https://cloud.ibm.com/docs/account?topic=account-sample)
[^ibm-cap]: [IBM Cloud spending notifications alert at thresholds but "don't stop charges from incurring"](https://cloud.ibm.com/docs/account?topic=account-billusagefaqs)
[^ibm-limits]: [IBM Code Engine limits — function memory/CPU combinations, 120 s timeout, payload and code size](https://cloud.ibm.com/docs/codeengine?topic=codeengine-limits)
[^koyeb-free]: [Koyeb pricing — free instance 0.1 vCPU / 512 MB; free instances scale to zero after 1 hour without traffic](https://www.koyeb.com/pricing)
[^koyeb-price]: [Koyeb instances — eco-nano $0.0000006/s (≈$1.61/mo), billed per second](https://www.koyeb.com/docs/reference/instances)
[^koyeb-sleep]: [Koyeb Light Sleep (~200 ms wake via snapshotting) vs Deep Sleep (1–5 s cold start)](https://www.koyeb.com/docs/run-and-scale/scale-to-zero)
[^koyeb-pg]: [Koyeb Serverless Postgres GA (May 2025)](https://www.koyeb.com/blog/serverless-postgres-ga-production-ready-databases-for-large-scale-and-ai-apps)
[^modal-py]: [Modal images — Python 3.10–3.14 (incl. 3.14t) selectable per image](https://modal.com/docs/guide/images)
[^modal-price]: [Modal pricing — $30/mo recurring credit on Starter; $0.0000131 per physical CPU core-s, $0.00000222 per GiB-s; no per-request or data-transfer line items](https://modal.com/pricing)
[^modal-limits]: [Modal resources — up to 64 CPUs / 336 GB per container](https://modal.com/docs/guide/resources); [timeouts — default 300 s, max 24 h](https://modal.com/docs/guide/timeouts)
[^modal-payload]: [Modal web endpoints — request bodies up to 4 GiB, response bodies unlimited (streamed)](https://modal.com/docs/guide/webhooks)
[^modal-scale]: [Modal plans — Starter: 100 containers, 10 concurrent GPUs](https://modal.com/pricing)
[^oci-py]: [Languages supported by OCI Functions](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/languagessupportedbyfunctions.htm); [Python FDK supports 3.11/3.12](https://github.com/fnproject/fdk-python)
[^oci-budget]: [OCI budgets are "soft limits" that alert but do not stop spending](https://docs.oracle.com/en-us/iaas/Content/Billing/Concepts/budgetsoverview.htm)
[^oci-price]: [OCI price list — Functions: $0.0000002/invocation, $0.00001417/GB-s, 2M requests + 400k GB-s always free; 10 TB/mo free egress then $0.0085/GB](https://www.oracle.com/cloud/price-list/)
[^oci-limits]: [Customizing functions — memory 128 MB–3 GB](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionscustomizing.htm); [concurrency capped at 60 GB total function memory per AD](https://docs.oracle.com/en-us/iaas/Content/Functions/Tasks/functionsmonitoringcapacityusage_topic-Monitoring-concurrent-function-execution.htm)
[^oci-detached]: [Detached (long-running) function invocations up to 3,600 s (Oct 2025)](https://docs.oracle.com/en-us/iaas/releasenotes/functions/functions-long-running-functions.htm)
[^railway-py]: [Railpack Python — 3.10+, default 3.13; auto-detects Flask/FastAPI/Django; requirements.txt, pyproject (poetry/pdm/uv), or Pipfile](https://railpack.com/languages/python)
[^railway-free]: [Railway plans — Free plan: $1/mo credit with 0.5 GB RAM / 1 vCPU / 4 GB image caps (100 GB image on Hobby)](https://docs.railway.com/pricing/plans); [free plan announcement (Aug 2025)](https://blog.railway.com/p/free-plan); [Serverless sleep means no compute charges while asleep](https://docs.railway.com/deployments/serverless)
[^railway-cap]: [Railway usage limits — a configurable hard limit stops all workloads (min $10), plus soft alert thresholds](https://docs.railway.com/pricing/cost-control)
[^railway-price]: [Railway pricing — Hobby $5/mo incl. $5 usage; $10 per GB RAM/mo, $20 per vCPU/mo, $0.05/GB egress, metered by the minute](https://docs.railway.com/pricing/plans)
[^railway-sleep]: [Railway Serverless (app sleeping) — sleeps after ~10 min without outbound traffic (open DB connections or telemetry keep it awake); wakes on inbound request, which may return a 502 on first hit](https://docs.railway.com/deployments/serverless)
[^railway-http]: [Railway public-networking limits — requests up to 15 min if data flows (5 min idle close), 32 KB headers, ~10k concurrent connections and ~11k RPS per domain](https://docs.railway.com/networking/public-networking/specs-and-limits)
[^railway-scale]: [Railway scaling — per-replica up to 8 GB / 8 vCPU on Hobby and 24 GB / 24 vCPU on Pro](https://docs.railway.com/deployments/scaling)
[^railway-db]: [Railway database templates (Postgres/MySQL/Redis/Mongo) are explicitly "unmanaged services"](https://docs.railway.com/databases)
[^railway-buckets]: [Railway Storage Buckets — S3-compatible object storage, $0.015/GB-mo (GA Sept 2025)](https://docs.railway.com/storage-buckets)
[^railway-gpu]: [Railway lists machine-learning compute as an unsupported use case (no GPUs)](https://docs.railway.com/platform/use-cases)
[^render-py]: [Render Python version docs — new services default to Python 3.14.3 with uv (since Feb 2026)](https://render.com/docs/python-version)
[^render-free]: [Render free instances — 0.1 CPU / 512 MB, 750 hrs/mo per workspace, spin down after 15 min idle, spin-up takes about one minute](https://render.com/docs/free)
[^render-price]: [Render pricing — Starter instance $7/mo (0.5 CPU / 512 MB)](https://render.com/pricing)
[^render-plans]: [Render workspace plans (Apr 2026) — Hobby $0 / Pro $25 / Scale $499 with 5 / 25 / 1,000 GB included bandwidth, $0.15/GB overage](https://render.com/docs/new-workspace-plans)
[^render-scale]: [Render scaling — autoscaling on paid instance types, max 100 instances/service](https://render.com/docs/scaling); [largest standard instance Pro Ultra 8 CPU / 32 GB, custom types above](https://render.com/pricing)
[^render-blob]: [Render object storage is in early access/alpha](https://feedback.render.com/features/p/cloud-object-storage)
[^scw-py]: [Scaleway function runtimes — Python 3.14 available (July 2026)](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-runtimes/)
[^scw-price]: [Scaleway serverless FAQ — free tier 1M requests + 400k GB-s/mo, €0.00000015/request + €0.0000170/GB-s beyond, free ingress/egress](https://www.scaleway.com/en/docs/serverless-functions/faq/)
[^scw-cap]: [Scaleway billing alerts notify only; no hard cap](https://www.scaleway.com/en/docs/billing/how-to/use-billing-alerts/)
[^scw-mem]: [Scaleway memory/CPU tiers — 128 MB to 4,096 MB](https://www.scaleway.com/en/docs/serverless-functions/reference-content/available-memory-and-cpu-tiers/)
[^scw-limits]: [Scaleway function limitations — timeout 10 s–60 min (default 300 s), 6 MiB request payload, 100 MiB zip / 500 MiB code, 50 instances/function, 5,000 req/s](https://www.scaleway.com/en/docs/serverless-functions/reference-content/functions-limitations/)
[^tencent-py]: [Tencent SCF Python runtime — capped at Python 3.10](https://cloud.tencent.com/document/product/583/55592)
[^tencent-limits]: [Tencent SCF quota limits](https://www.tencentcloud.com/document/product/583/11637)
[^tencent-mem]: [Tencent SCF memory specs — 64–3,072 MB standard plus fixed large-memory specs of 6/14/30/60/120 GB](https://cloud.tencent.com/document/buy-guide/583/68734)
[^tencent-timeout]: [Tencent SCF execution timeout — 1–900 s (default 3 s); asynchronous executions up to 24 h](https://cloud.tencent.com/document/product/583/51519)
[^vercel-py]: [Vercel Python runtime (Beta) — 3.12 default](https://vercel.com/docs/functions/runtimes/python); [Python 3.13/3.14 available since Feb 2026](https://vercel.com/changelog/python-3-13-and-3-14-are-now-available)
[^vercel-hobby]: [Vercel Hobby plan included usage — hard-paused when exceeded](https://vercel.com/docs/plans/hobby)
[^vercel-price]: [Vercel Pro plan — $20/seat/mo with $20/mo flexible usage credit](https://vercel.com/docs/plans/pro-plan); [rates per usage & pricing docs](https://vercel.com/docs/functions/usage-and-pricing); [invocations billed per unit ($0.0000006) since May 2026](https://vercel.com/changelog/function-invocations-now-billed-per-unit)
[^vercel-mem]: [Vercel function memory — Standard 2 GB / 1 vCPU (Hobby fixed), Performance 4 GB / 2 vCPU](https://vercel.com/docs/functions/configuring-functions/memory)
[^vercel-limits]: [Vercel function duration (default 300 s; max 300 s Hobby / 800 s Pro; 1,800 s beta for Node.js & Python)](https://vercel.com/docs/functions/configuring-functions/duration); [4.5 MB request & response body limit](https://vercel.com/docs/functions/limitations)
[^vercel-bundle]: [250 MB bundle standard; 500 MB for Python](https://vercel.com/changelog/python-vercel-functions-bundle-size-limit-increased-to-500mb); ["Large Functions" up to 5 GB in public beta](https://vercel.com/docs/functions/limitations)
[^vercel-conc]: [Vercel concurrency — 30,000 on Pro (burst 1,000/10 s per region), 100,000 on Enterprise](https://vercel.com/docs/functions/concurrency-scaling)
[^yandex-py]: [Yandex Cloud Functions runtimes — python312 and python314 supported (3.7–3.11 desupported)](https://yandex.cloud/en/docs/functions/concepts/runtime/); [Python handler model — async supported; per-instance concurrency is not available for Python](https://yandex.cloud/en/docs/functions/lang/python/handler)
[^yandex-deps]: [requirements.txt dependencies are pip-installed automatically when a function version is created](https://yandex.cloud/en/docs/functions/lang/python/dependencies)
[^yandex-free]: [Yandex serverless free tier — 1M invocations + 10 GB×hr per month, resets monthly](https://yandex.cloud/en/docs/billing/concepts/serverless-free-tier); [first 100 GB/mo of egress free](https://yandex.cloud/en/docs/functions/pricing)
[^yandex-cap]: [Yandex budgets are notification-only](https://yandex.cloud/en/docs/billing/concepts/budget), but [budget triggers can invoke a function that stops resources](https://yandex.cloud/en/docs/functions/tutorials/serverless-trigger-budget-vm)
[^yandex-price]: [Yandex Cloud Functions pricing — $0.144 per 1M invocations + $0.049230 per GB×hour (USD for international entities; RUB for Russian customers), rounded up to 100 ms](https://yandex.cloud/en/docs/functions/pricing)
[^yandex-limits]: [Yandex Cloud Functions limits — memory 128 MB–8 GB (4 vCPU at 8 GB), 3.5 MB request/response, code 3.5 MB console zip / 128 MB via bucket / 680 MB unzipped; default quotas: 10 functions per cloud, 10 concurrent calls and 10 instances per AZ (raisable)](https://yandex.cloud/en/docs/functions/concepts/limits)
[^yandex-timeout]: [Maximum execution 10 min, or up to 1 h for long-lived functions](https://yandex.cloud/en/docs/functions/concepts/long-lived-functions)
[^yandex-hosting]: [Yandex Object Storage static website hosting](https://yandex.cloud/en/docs/storage/concepts/hosting)
