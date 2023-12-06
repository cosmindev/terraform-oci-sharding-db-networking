# OCI DB Sharding Service - Required IAM & Network Configurations for the customer tenancy

## **Table of Contents**

[1. Summary](#1-summary)</br>
[2. Setup IAM Configuration](#2-setup-iam-configuration)</br>
[3. Setup Network Configuration](#3-setup-network-configuration)</br>
[4. Run with ORM](#4-run-with-orm) </br>
[5. Run with TF CLI](#5-run-with-terraform-cli)


&nbsp; 


## **1. Summary**

| |  |
|---|---| 
| **OP. NAME** | Provisioning the required OCI IAM & Netwoking configuration for sharding | 
| **OBJECTIVE** | Provisioning the required OCI IAM & Netwoking configuration for sharding. |
| **TARGET RESOURCES** | - **Security**: Compartments, Groups, Policies</br>- **Network**: DRG, VCN, Subnets, SL, RT, DRG, RPCs|
| **IAM CONFIG**| [open_lz_shared_identity.auto.tfvars.json](sharding_identity.auto.tfvars.json)|
| **NETWORK CONFIG** |[sharding_network_region_1.auto.tfvars.json](sharding_network_region_1.auto.tfvars.json), [sharding_network_region_2.auto.tfvars.json](sharding_network_region_2.auto.tfvars.json) |
| **TERRAFORM MODULES**| [CIS IAM](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam), [CIS Network](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-networking)  |
| **DETAILS** |  For more details refer to the ... |
| **PRE-ACTIVITIES** | Tenancy created. Review configuration files. |
| **POST-ACTIVITIES** | Review provisioned configuration, terraform output and terraform state file. |
| **RUN OPERATION** | Use [ORM](#4-run-with-orm) or use [Terraform CLI](#5-run-with-terraform-cli). |


&nbsp; 



&nbsp; 

![Diagram](diagrams/ShardingCustomerNetwork.png)

## **2. Setup IAM Configuration**

For configuring and running the Sharding IAM layer use the following JSON file: [open_lz_shared_identity.auto.tfvars.json](sharding_identity.auto.tfvars.json). You can customize this  configuration to fit your exact OCI IAM topology.

This configuration file will cover the following four categories of resources described in the next sections.

&nbsp; 

###  **2.1. Compartments**

The diagram below identifies the compartments in the scope of this operation.

&nbsp; 



&nbsp; 

The corresponding JSON configuration for the compartments topology is: 

```
...
    "compartments_configuration": {
        "enable_delete": "true",
        "default_parent_ocid": "ocid1.....",
        "compartments": {
            "CMP-NETWORK-KEY": {
                "name": "cmp-network",
                "description": "OCI Sharding Network Compartment",
                "defined_tags": null,
                "freeform_tags": {
                    "oci-open-lz": "oci-sharding",
                    "oci-open-lz-customer": "oci-sharding-customer",
                    "oci-open-lz-cmp": "network"
                }
            }
        }
    }
...
```

For extended documentation please refer to the [Identity & Access Management CIS Terraform module compartments example](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam/blob/main/compartments/examples/vision/input.auto.tfvars.template).

&nbsp; 

### **2.2 Groups**

The corresponding JSON configuration for the groups topology is: 

```
...
    "groups_configuration": {
        "default_defined_tags": null,
        "default_freeform_tags": null,
        "groups": {
            "GRP-NETWORK-ADMINS": { 
                "name": "grp-network-admins",  
                "description": "Sharding Network Admins"
            }
        }
    }
...
```

This automation provides fully supports any kind of OCI IAM Groups topology to be specified in the JSON format.

For an example of such a configuration and for extended documentation please refer to the [Identity & Access Management CIS Terraform module groups example](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam/blob/main/groups/examples/vision/input.auto.tfvars.template).

&nbsp; 

### **2.3 Dynamic Groups**

The corresponding JSON configuration for the dynamic groups topology is: 

```
...
   "dynamic_groups_configuration": null,
...
```

This automation fully supports any kind of OCI IAM Dynamic Groups to be specified in the JSON format.

For an example of such configuration and for extended documentation please refer to the [Identity & Access Management CIS Terraform module dynamic groups example](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam/blob/main/dynamic-groups/examples/vision/input.auto.tfvars.template).

&nbsp; 

### **2.4 Policies**

The corresponding JSON configuration for the policies is: 

For this example, replace the compartment_ocid to your tenancy OCID.

```
...
   "policies_configuration": {
        "enable_cis_benchmark_checks": "false",
        "supplied_policies": {
            "PCY-NETWORK-ADMINISTRATION": {
                "name": "pcy-network-administration",
                "description": "Sharding netwoking IAM policies",
                "compartment_ocid": "ocid1....",
                "statements": [
                    "allow group grp-network-admins to use cloud-shell in tenancy",
                    "allow group grp-network-admins to read usage-budgets in tenancy",
                    "allow group grp-network-admins to read usage-reports in tenancy",
                    "allow group grp-network-admins to read objectstorage-namespaces in tenancy",
                    "allow group grp-network-admins to read all-resources in compartment cmp-network",
                    "allow group grp-network-admins to manage virtual-network-family in compartment cmp-network",
                    "allow group grp-network-admins to manage dns in compartment paalonso-openlz-tfcli:cmp-network",
                    "allow group grp-network-admins to manage load-balancers in compartment cmp-network",
                    "allow group grp-network-admins to manage alarms in compartment cmp-network",
                    "allow group grp-network-admins to manage metrics in compartment cmp-network",
                    "allow group grp-network-admins to manage ons-family in compartment cmp-network",
                    "allow group grp-network-admins to manage orm-stacks in compartment cmp-network",
                    "allow group grp-network-admins to manage orm-jobs in compartment cmp-network",
                    "allow group grp-network-admins to manage orm-config-source-providers in compartment cmp-network"
                ]
            }
        }
    }
...
```

This automation fully supports any type of OCI IAM Policy  to be specified in the JSON format. 

For an example of such configuration and for extended documentation please refer to the [Identity & Access Management CIS Terraform module policies examples](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-iam/tree/main/policies/examples).


&nbsp; 

## **3. Setup Network Configuration**

For configuring the OCI Sharding required Network configuration open and edit the following JSON configuration file: [sharding_network_region_1.auto.tfvars.json](sharding_network_region_1.auto.tfvars.json). 


You can customize this JSON configuration to fit your exact OCI Networking topology. This Terraform automation is extremely versatible and can support any type of network topology. 

For complete documentation and a larger set of examples on configuring an OCI networking topology using this JSON terraform automation approach please refer to the [OCI CIS Terraform Networking Module](https://github.com/oracle-quickstart/terraform-oci-cis-landing-zone-networking) documentation and examples.


&nbsp; 

## **4. Run with ORM**

| STEP |  ACTION |
|---|---| 
| **1** | [![Deploy_To_OCI](./diagrams/DeployToOCI.svg)](https://cloud.oracle.com/resourcemanager/stacks/create?zipUrl=https://github.com/oracle-quickstart/terraform-oci-open-lz/archive/refs/heads/master.zip&zipUrlVariables={"input_config_files_urls":"https://raw.githubusercontent.com/cosmindev/terraform-oci-sharding-db-networking/main/sharding_identity.auto.tfvars.json,https://raw.githubusercontent.com/cosmindev/terraform-oci-sharding-db-networking/main/sharding_network_region_1.auto.tfvars.json"}) |
| **2** | Accept terms,  wait for the configuration to load. |
| **3** | Set the working directory to “orm-facade”. | 
| **4** | Set the stack name you prefer. | 
| **5** |  Set the terraform version to 1.2.x. Click Next. | 
| **6** | Accept the defaul configurations. Click Next. Optionally,replace with your json/yaml config files. |
| **7** | Un-check run apply. Click Create. 

&nbsp; 

## **5. Run with Terraform CLI**
&nbsp; 

### **5.1. Setup Terraform Authentication**

For authenticating against the OCI tenancy terraform execute the following [instructions](/examples/oci-open-lz/common_terraform_authentication.md).

&nbsp; 

### **5.2 Clone this Git repo to your Machine**

```
git clone git@github.com:oracle-quickstart/terraform-oci-open-lz.git?ref=v1.0.0
```

For referring to a specific module version, append *ref=\<version\>* to the *source* attribute value.

&nbsp; 

###  **5.3 Change the Directory to the Terraform Orchestrator Module**

 Change the directory to the [```terraform-oci-open-lz/orchestrator```](../../../../orchestrator/) Terraform orchestrator module.

&nbsp; 

 ### **5.4 Run ```terraform init```**

Run ```terraform init`````` to download all the required external terraform providers and Terraform modules. See [command example](./tf_init_output_example.out) for more details on the expected output.

&nbsp; 

 ### **5.5 Run ```terraform plan```**

Run ```terraform plan`````` with the IAM and Network configuration.

```
terraform plan \
-var-file ../examples/oci-open-lz/op01_manage_shared_services/oci-credentials.tfvars.json \
-var-file ../examples/oci-open-lz/op01_manage_shared_services/open_lz_shared_identity.auto.tfvars.json \
-var-file ../examples/oci-open-lz/op01_manage_shared_services/open_lz_shared_network.auto.tfvars.json \
-state ../examples/oci-open-lz/op01_manage_shared_services/terraform.tfstate
```

After the execution please analyze the output of the command above and check if it corresponds to your desired configuration.

Note that the ```terraform.tfstate``` file is generated in the configuration location and not in the Terraform code location. This is the expected configuration as the Terraform automation can support any number of configurations and the **state file** will belong to the configuration and not to the code.
  
The ideal scenario regarding the **state file** will be for each configuration to have a corresponding OCI Object Storage location for the state file. For more details on the Terraform state file recommended configuration please refer to the following [documentation](https://docs.oracle.com/iaas/Content/API/SDKDocs/terraformUsingObjectStore.htm).

&nbsp; 

### **5.6 Run ```terraform apply```**

Run ```terraform apply``` with the IAM and Network configuration. After its execution the configured resources will be provisioned or updated on OCI.

```
terraform apply \
-var-file ../examples/oci-open-lz/op01_manage_shared_services/oci-credentials.tfvars.json \
-var-file ../examples/oci-open-lz/op01_manage_shared_services/open_lz_shared_identity.auto.tfvars.json \
-var-file ../examples/oci-open-lz/op01_manage_shared_services/open_lz_shared_network.auto.tfvars.json \
-state ../examples/oci-open-lz/op01_manage_shared_services/terraform.tfstate
```

Depending on your JSON configuration configurations the output of the ```terraform apply``` should be identical or similar to this [example](./tf_apply_output_example.out).

&nbsp; 

# License

Copyright (c) 2023 Oracle and/or its affiliates.

Licensed under the Universal Permissive License (UPL), Version 1.0.

See [LICENSE](LICENSE) for more details.
