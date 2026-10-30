# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
49
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->267.5<!--/answer--> | <!--answer:vm_warm_median_ms-->212.4<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->4744.2<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->143.2<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1534.9<!--/answer--> | <!--answer:function_warm_median_ms-->133.6<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
[OK ] vm cold= 267.5ms warm_median= 212.4ms

[OK ] cloudrun cold= 4744.2ms warm_median= 143.2ms

[OK ] function cold= 1534.9ms warm_median= 133.6ms
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
The VM does not automatically create anything to absorb additional load. All requests are served by a single e2-micro instance, so capacity is limited by the CPU and memory of that machine. Cloud Run automatically creates additional container instances when traffic increases. In my deployment the maximum was capped at 3 instances using the --max-instances=3 configuration. Cloud Functions Gen 2 also scales automatically by creating additional function instances when needed. Its limit comes from platform configuration and quotas rather than from a VM size selected by the user. With one request the platforms use minimal resources, with ten or one hundred requests Cloud Run and Functions can scale out automatically while the VM cannot.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
The VM deployment required the most manual work. I had to create a VPC, subnet, firewall rules, a Compute Engine VM, install Docker, configure Artifact Registry access, pull the container image, and run the application manually. Cloud Run was simpler because I only needed to provide the container image and deployment settings while Google managed the infrastructure, networking, and scaling. Cloud Functions required the least operational effort because I deployed source code directly from main.py without building or managing a container myself. The platform automatically built and deployed the function and handled scaling and runtime management.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-89248238899.us-central1.run.app
LAST DEPLOYED BY: gabriel.lapointe@eurecom.fr
LAST DEPLOYED AT: 2026-10-29T14:15:58.094016Z

SERVICE: echo-function
REGION: us-central1
URL: https://echo-function-89248238899.us-central1.run.app
LAST DEPLOYED BY: service-89248238899@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-29T14:41:27.365098Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output shows that both the Cloud Run application and the Cloud Function appear as Cloud Run services. This indicates that Cloud Functions Gen 2 is implemented on top of Cloud Run. As a result, both platforms share a similar execution environment and scaling model, which explains why their cold-start measurements are relatively close compared with the VM deployment. In both cases Google creates and manages the underlying container infrastructure automatically. The main difference is the deployment workflow. Cloud Run deploys a container image that I built and stored in Artifact Registry, while Cloud Functions deploys source code directly. Google then builds, packages, and runs the function on my behalf. Operationally they are managed similarly, but the developer experience and packaging model differ.
<!--/answer-->
