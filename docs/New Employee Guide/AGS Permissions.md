
## Rancher


Role Name:
```
CaaS Users
```

**Notes:** Rancher role is required in order to work with the Rancher (Kubernetes) Clusters we own.

**Verifying:** Make sure you have permissions to a Rancher cluster individually or from an Active Directory group, then try and login to the Rancher instance. 


![](Rancher%20Instances.md)

## Samba (NFS)

Role Name:
```
EC GER ibi_idc_5961 R
```

**Notes:** Grants access to the following Samba path: [\\ger.corp.intel.com\ec\proj\ha\computing\ibi](file://ger.corp.intel.com/ec/proj/ha/computing/ibi)

**Verifying:** Open the link to make sure you have access.


## Yammer

Role Name:
```
M365 - Yammer License Exception for Contingent Workers
```

**Notes:** Yammer portal is not accessible for contingent workers. Information displayed in Yammer is classified as Intel Confidential that is why its use is restricted

**Verifying:** Go to this link and check if it works [PDA Yammer](https://web.yammer.com/main/groups/eyJfdHlwZSI6Ikdyb3VwIiwiaWQiOiI1NzA3Nzg3NDY4OCJ9/all)


## Azure Portal

Role Name:
```
Design-PDA-Azure-Developers
```

**Notes:** The Azure cloud deployments are there (Kubernetes, etc)

**Verifying:** Login to [https://portal.azure.com/](https://portal.azure.com/) and check if you see the resources.


## Saba

Role Name:

First Role:
```
EC RESOURCES LOCAL TEAMS IDC DESIGNML
```

(EC Resources Local Teams IDC DesignML Admins)

Second Role:
```
EC Resources EC-UNIX ACCESS EC GLOBAL TEAMS iBI global iBI admins AD -sudo group
```


**Notes:**

You must complete the courses below to have access to these roles:

1. GLOBAL EXPORT COMPLIANCE
2. 2021 Information Security and Privacy Awareness
3. ISA FOR SYSTEM ADMIN

**Verifying:** Ask for the roles in AGS.

## Azure Cloud Subscriptions

Old Dev (not in PDA area)
```
Approver-Design and Quality Data Hub_ID_e54d0018-47e8-45df-bb63-ff841f24a811
```

QA
```
Business Group Admin Access to Azure Subscription f72715eb-0350-4d86-b1a5-933b3a12acaf_AZADCORP
```

Dev
```
Business Group Admin Access to Azure Subscription 1fd3090a-949b-4ab5-b266-8fbb6701f677_AZADCORP
```

Prod
```
Business Group Admin Access to Azure Subscription 9c422518-49e0-49ba-a2cd-24287e25878f_AZADCORP
```