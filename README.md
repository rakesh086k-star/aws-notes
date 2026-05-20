Hi Team,

We reviewed the recent Microsoft announcement regarding the discontinuation of Azure Reserved Instance (RI) purchases/renewals for select VM series effective July 1, 2026.

Impacted VM families mentioned by Microsoft include:
- D / Ds
- Dv2 / Dsv2
- Dv3 / Dsv3
- Ev3 / Esv3
- F / Fs / Fsv2 and other legacy series

Currently, our environment is utilizing the below compute families with RI:
- Standard_D8s_v5
- Standard_D16s_v5
- Standard_D8s_v4

Based on the initial assessment, our active V4/V5 compute series are not part of the impacted VM families listed in the announcement. No immediate impact has been identified for existing workloads.

We will additionally review current Azure Reservations to validate whether any legacy RI dependencies exist in the environment.

Regards,  
Rakesh Kumar