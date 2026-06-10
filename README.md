Hi Bharat,

We have reviewed the optimization recommendations and do not believe the proposed VM downsizing is suitable for our AVD environment.

The recommendation suggests reducing the compute size by approximately 50%. Could you please confirm whether the utilization analysis was performed based on actual user concurrency and session host utilization?

Implementing the recommended sizing may negatively impact user experience and could result in application slowness and session stability issues. Based on our operational experience, a configuration of 2 vCPU and 16 GiB RAM is not sufficient to support 7–8 concurrent users per session host.

For reference, the current VM sizing aligns with our standard AVD deployment model:

• Shared AVD environments are provisioned with Standard E4as v5 (4 vCPU, 32 GiB RAM), supporting approximately 7–8 users per session host.

• Dedicated AVD environments are provisioned with Standard D2as v5 (2 vCPU, 8 GiB RAM), unless application or business requirements necessitate a larger compute size.

Additionally, AVD shutdown schedules have already been implemented across these environments as part of our ongoing cost optimization initiatives.

Based on our review, the current VM sizing aligns with operational requirements and existing user workloads. Please refer to the attached analysis and let us know if any specific workload-based optimization opportunities require further discussion.

Regards,

Rakesh Kumar  
AVD Administration Team