Hi Bharat,

Thank you for sharing the optimization recommendations.

We have reviewed the highlighted AVD environments and validated them against the current user count, host pool design, and workload requirements.

Our standard AVD sizing model is based on user density and workload utilization:

• Shared AVD environments are provisioned with Standard E4as v5 (4 vCPU, 32 GiB RAM), supporting approximately 7–8 users per session host. Additional session hosts are provisioned based on user count and workload demand.

• Dedicated AVD environments are typically provisioned with Standard D2as v5 (2 vCPU, 8 GiB RAM), unless application or business requirements necessitate a larger compute size.

The attached analysis includes the current compute configuration, host pool type, VM count, user count, and the resources identified during our review.

Additionally, AVD shutdown schedules have already been implemented across these environments as part of our ongoing cost optimization initiatives.

Based on the current utilization patterns, user density, and operational requirements, the listed VM sizes have been provisioned according to the standard design guidelines. We request the cloud team to review the attached analysis and advise if any specific workload-based optimization opportunities require further discussion.

Please let us know if any additional information is required from our side.

Regards,

Rakesh Kumar
AVD Administration Team



Hi Bharat,

We have reviewed the highlighted AVD resources.

The current VM sizing is based on our standard AVD deployment model, considering user density, host pool design, and workload requirements. Shared environments are provisioned and scaled according to active user count, while dedicated environments are sized based on specific workload needs.

Additionally, shutdown schedules are already in place across these environments as part of our cost optimization strategy.

Please refer to the attached analysis for the current sizing, user count, and host pool details. Based on our review, the existing configurations align with operational requirements.

Please let us know if any further clarification is required.

Regards,
Rakesh


