
T
TLE - Tenant Lifecycle Engineering
# Workday systems
Scylla
Pharos
Venice - Workday multi-cloud service mesh
Atlantis

# Vault-related #kb/vault/glossary
PKI - Public Key Infrastructure. There are 3 entities form a PKI: Client, Server and CA. Client needs to connect securely or verify an identity, Server needs to prove its identity and CA validates identities & generates certificates.

[Alpaca](https://confluence.workday.com/display/sec/Alpaca) - Workday's internal Certificate Authority. It is the CA (certificate authority) solution authorized to issue, manage, validate, and renew digital internal certificates to Workday.

CSR - Certificate Signing Request. In PKI systems, a CSR is a message sent from an application to a CA (certificate authority) of the PKI in order to apply for a digital identity certificate.

TBS - Tenant Build Sequence for arcane reasons related to the origins of Alpaca. It can be thought of as something like a CSR, but for Alpaca intermediates.
# Technologies used
Kubernetes
Hashicorp Vault

DNS being in DC-PDX is a problem
	Only traffic towards services running in PDX and traffic from client in us-west-2 trying to reach ATL as TGW there plugs into PDX edge
