Connect OCI Devops and Minikube using deployment pipeline.

-------------


Objectives
----

- Deploy a single node minikube on an OCI Compute instance .

- Create an acces token for minikube and store and retrive via oci vault service.

- Create a OCI Deployment pipeline and invoke a deployment to an OCI VM.

- Use the OCI VM as a jump host to access minikube cluster.

Audience : Intermediate / Experienced OCI users.

Create an OCI Compute instance
---

- Create an OCI Compute instance - https://docs.oracle.com/en-us/iaas/Content/Compute/home.htm 

- OCI Console > Compute > Instances > Create an instance.

![](images/oci_compute1.png)

- Use `Oracle Linux 8` as os version and any of the available compute shape .

![](images/oci_compute2.png)

- Select the networking configuration as accordingly . Select Public subnet if you are planning to add a public IP in the below step.

- Ensure to assign a public IP for the VM to access it via cloud shell.You may use with a Private IP too ,then use an OCI Bastion service to access the VM.

![](images/oci_compute3.png)


- Use `Show advanced option` > `Management` and enable sudo access to user `ocarun` which will be used by the compute instance agent.

- Select ` Paste cloud-init script` and copy below snippet and `Submit`.

```
#cloud-config
users:
  - default
  - name: ocarun
    sudo: ALL=(ALL) NOPASSWD:ALL

```

![](images/oci_compute4.png)


- Once the vm is in `RUNNING` stage , check the compute agent staus via `Oracle Cloud Agent` tab. Ensure that the agent is `Enabled` and `Running state`

![](images/oci_compute5.png)


Setup the policies and groups
--------

- OCI Console > `Identity` > Create a new group and associate the OCI Users to it to enable to manage compute agent execution .

![](images/oci_group.png)


- OCI Console > `Identity` > Create a new dynamic group for devops deployment pipeline and compute agent.

- Use below `Rules` for the dynamic groups 

```
ALL {resource.type = 'devopsdeploypipeline', resource.compartment.name = '<COMPARTMENT NAME>'}	
All {instance.id = '<OCID OF THE COMPUTE INSTANCE>'}
```

![](images/oci_dg_1.png)

- OCI Console > `Identity` > Policies > Create a policy and add below statements.


```
Allow group <NAME OF THE USER GROUP > to manage instance-agent-command-family in compartment <NAME OF THE COMPARTMENT>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to read secret-family in compartment <NAME OF THE COMPARTMENT>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to use ons-topics in compartment <NAME OF THE COMPARTMENT>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to use instance-agent-command-execution-family in compartment mr-prod-child-2 where request.instance.id=<OCID of the COMPUTE INSTANCE>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to read generic-artifacts in compartment <NAME OF THE COMPARTMENT>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to read all-artifacts in compartment <NAME OF THE COMPARTMENT>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to read objects in compartment <NAME OF THE COMPARTMENT>
Allow dynamic-group <NAME OF THE DYNAMIC GROUP> to manage objects in compartment mr-prod-child-2

```

Install Minikube
---




