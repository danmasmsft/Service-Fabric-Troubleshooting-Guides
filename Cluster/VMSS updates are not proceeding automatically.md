# VMSS updates are stuck

## Symptoms

- You have enabled durabilityLevel = Silver/Gold in the VMSS extension configuration for Service Fabric
- Updates to the VMSS itself or to other resources (like load balancers) linked to the VMSS are slow, timing out, or end up in the Failed state

## Background

Setting durabilityLevel = Silver/Gold in the Service Fabric extension configuration of a VM Scale Set causes many actions affecting the VMs to be delayed until they are approved by the cluster, to reduce the likelihood of availability loss and data loss.  Service Fabric coordinates with VM Scale Sets to approve these actions after the Service Fabric cluster determines that it is safe for them to proceed.  The SF system service that handles this coordination is `fabric:/System/InfrastructureService/<NodeType>`, and there should be one instance per VMSS with durabilityLevel = Silver/Gold.

Misconfigurations, bugs, concurrent update activity, and other issues in your Service Fabric cluster can result in VMSS operations being delayed.

## Common issues

- Misconfiguration: InfrastructureService does not exist for the VMSS
  - VMSS has durabilityLevel = Silver/Gold, but the corresponding SF node type does not
  - Multiple VM Scale Sets have durabilityLevel = Silver/Gold and map to the *same* node type
- Safety issue: InfrastructureService is functioning properly and has determined it is unsafe to proceed
  - The system or an application has a problem which is preventing node deactivation from completing.
  - There is too much other activity occurring simultaneously in the cluster (i.e. job throttling)
- Other issue: InfrastructureService is failing and is therefore unable to respond

