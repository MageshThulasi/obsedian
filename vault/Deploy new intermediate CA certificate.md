## JIRA Tickets
1. [MTL-CUST](https://jira2.workday.com/browse/IPCE-127) - WD10-CUST
2. [ORE-CUST](https://jira2.workday.com/browse/IPCE-126) - WD12-NPRD
## Task List
Vault's intermediate certificate for this region is due to expire. We need to get that switched over before we hit the 30 day mark.

You need to:
1. Enable a secondary PKI backend in Vault in ORE-CUST and generate a TBS file
2. Attach the TBS file to the provided PLATSEC ticket so they can generate the new intermediate certificate.
3. Attach the new intermediate CA certificate to the secondary PKI mount and swap it with the current PKI mount.

### Questions
1. Should we add Sprint field to my tickets so that we can view them on the Scrum board?
2. Where is the change window posted?
3. What is the CAB schedule?