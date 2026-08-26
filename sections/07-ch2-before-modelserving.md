## **2.10. Model serving \- Before upgrade** {#2.10.-model-serving---before-upgrade}

Migrate your model serving workloads from removed deployment modes (**Serverless** and **ModelMesh**) to **RawDeployment** mode before upgrading Red Hat OpenShift AI from version to 3.5.

Red Hat OpenShift AI 3.5 does not include **Serverless** and **ModelMesh** deployment modes. This guide explains how to migrate model serving workloads by moving from KServe **Serverless** and **ModelMesh** configurations to **RawDeployment** mode. It also includes instructions for migrating distributed inference workloads based on the **LLMInferenceService** custom resource.

###  **2.10.1. Migration impact and scope** {#2.10.1.-migration-impact-and-scope}

The migration requires both cluster administrators and users to work together to update infrastructure and workloads:

**Impact of not migrating:**

* All **InferenceServices** using **Serverless** or **ModelMesh** deployment modes will return HTTP 503 Service Unavailable errors after the operator upgrade.

* Distributed inference workloads using **LLMInferenceService** will fail without proper authentication configuration.

* Cluster-wide components including OpenShift Serverless, Service Mesh v2, and standalone Authorino will become unsupported and must be removed.

**Which parts of this guide apply to you:**

Use the **rhai-cli** tool to analyze your cluster and determine your specific migration path. Depending on your workloads, you may need to:

* Migrate **InferenceServices** from **Serverless** or **ModelMesh** to **RawDeployment** mode

* Update cluster configuration and remove operators (required for all upgrades)

* Prepare distributed inference workloads using **LLMInferenceService** with Red Hat Connectivity Link

**Important**

All applicable procedures in this section must be completed before upgrading the Red Hat OpenShift AI operator from version to 3.5. Failure to complete these steps will result in service disruptions for model inference endpoints.

### **2.10.2. Removed model serving configurations** {#2.10.2.-removed-model-serving-configurations}

As of Red Hat OpenShift AI 3.5, the following model serving configurations are removed:

* **Serverless** deployment mode

* **ModelMesh** deployment mode (multi-model serving)

* OpenShift Serverless Operator integration

* OpenShift Service Mesh v2 integration

* Standalone **Authorino** Operator: replaced with Red Hat Connectivity Link for **LLMInferenceService**

All model serving workloads must migrate to **RawDeployment** mode, which provides standard Kubernetes **Deployment** resources without serverless infrastructure.

### **2.10.3. Migration workflow for model serving** {#2.10.3.-migration-workflow-for-model-serving}

Following is an overview of the steps that you must complete for the model serving component before you upgrade the Red Hat OpenShift AI operator:

1. Run the **rhai-cli** tool and analyze your cluster configuration, as described in [Run the rhai-cli script.](#2.10.5.-run-the-rhai-cli-tool)

2. Back up the inferenceservice-config ConfigMap, as described in [Back up the inferenceservice-config ConfigMap](#2.10.6.-back-up-the-inferenceservice-config-configmap).

3. Update the inferenceservice-config ConfigMap with hardware profiles ignorelist changes, as described in [Update the inferenceservice-config ConfigMap](#2.10.8.-update-the-inferenceservice-config-configmap)..

4. Convert ModelMesh and Serverless InferenceServices to RawDeployment mode (if you have impacted InferenceServices), as described in [Migrate InferenceServices to RawDeployment mode](#2.10.7.-migrate-inferenceservices-to-rawdeployment-mode).

5. Update cluster configuration to set RawDeployment as the default deployment mode and disable removed components, as described in  [Update cluster configuration for migration](#2.10.9.-update-cluster-configuration-for-migration).

6. Prepare distributed inference workloads (if you have LLMInferenceService resources): Install Red Hat Connectivity Link, configure authentication, and freeze LLMInferenceService configurations, as described in [Prepare distributed inference for migration](#2.10.10.-prepare-distributed-inference-for-migration).

7. Verify migration readiness using the rhai-cli tool, as described in [Verify migration readiness](#2.10.11.-verify-migration-readiness).

### **2.10.4. Prerequisites for model serving migration** {#2.10.4.-prerequisites-for-model-serving-migration}

Before migrating Model Serving workloads from Red Hat OpenShift AI 2.25.10 (and later) to 3.5, verify that your environment meets the following requirements.

* You have cluster administrator access to the OpenShift cluster and are authenticated via the OpenShift CLI (**oc**).

* You have project-level access to namespaces containing **InferenceServices** and **LLMInferenceServices**.

* You have audited the cluster to identify any other applications relying on OpenShift Service Mesh v2 and confirmed they can be migrated or removed. For more information about services that use the OpenShift Service Mesh, see [Adding services to a  service mesh](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/service_mesh/service-mesh-2-x#ossm-create-mesh) in the OpenShIft Container Platform documentation. 

**Important**

If you cannot remove OpenShift Service Mesh v2 due to other dependencies, you must upgrade those applications to Service Mesh v3 before upgrading Red Hat OpenShift AI to version 3.5. For Service Mesh migration, see [Migrating from Service Mesh 2 to Service Mesh 3](https://docs.redhat.com/en/documentation/red_hat_openshift_service_mesh/3.1/html/migrating_from_service_mesh_2_to_service_mesh_3/index).

If a conflicting OSSM v2.x subscription is present when you create a **GatewayClass** resource, the Cluster Ingress Operator attempts to install the required OSSM v3.x components but fails. This causes Gateway API resources to have no effect and no proxy gets configured to route traffic. Do not continue with the migration steps until after you resolve this conflict.

### **2.10.5. Run the rhai-cli tool** {#2.10.5.-run-the-rhai-cli-tool}

Set up the **rhai-cli** container to analyze your model serving configuration.

**Prerequisites**

* You have access to the **rhai-cli** tool, as described in  [Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod).

**Note**

All migration commands must be executed within the **rhai-cli** container, as it provides the necessary dependencies. However, the backup of the **inferenceservice-config** **ConfigMap** must be run on your local machine before starting the container.

**Procedure**

1. From inside the rhai-cli  container, verify the status of the kserve and modelmesh resources:

   ```bash
   /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5 --verbose --checks "*kserve*" --checks "*modelmesh*"
   ```

   The command analyzes your cluster configuration and reports any issues that must be resolved before upgrading to OpenShift AI 3.5.

   Example output:

| STATUS | KIND | GROUP | CHECK | IMPACT | MESSAGE |
| :---- | :---- | :---- | :---- | :---- | :---- |
| ✗ | kserve | component | serverless-removal | critical | KServe serverless mode enabled but will be removed |
| ✗ | modelmeshserving | component | removal | critical | ModelMesh is enabled but will be removed |
| ✓ | kserve | workload | impacted-workloads | info | No Serverless InferenceService(s) found |
| ✓ | kserve | workload | impacted-workloads | info | No ModelMesh InferenceService(s) found |
| ⚠ | servicemesh-operator-v2 | dependency | upgrade | warning | Service Mesh Operator v2 no longer required |

   The output shows the status of your cluster configuration:  
   **✓ (checkmark)**  
   Configuration meets requirements for OpenShift AI 3.5  
   **✗ (cross)**  
   Critical issue that must be resolved before upgrading  
   **⚠ (warning)**  
   Non-critical issue that should be addressed

2. Review the **GROUP** and **KIND** columns to identify which components require attention:

   * **workload with kserve**  
     Indicates **InferenceServices** that need migration.

   * **component with kserve or modelmeshserving**  
     Indicates cluster configuration changes needed.

   * **dependency**  
     Indicates operator installation or removal requirements.

### **2.10.6. Back up the inferenceservice-config ConfigMap** {#2.10.6.-back-up-the-inferenceservice-config-configmap}

Create a backup of the **inferenceservice-config** **ConfigMap** before beginning the migration process.

**Prerequisites**

* You have cluster administrator access to your OpenShift cluster.

* You have authenticated to the cluster via the OpenShift CLI (**oc**).

**Note**

Run this backup command on your local machine, not inside the **rhai-cli** container.

**Procedure**

1. Create the backup directory and back up the **inferenceservice-config** **ConfigMap**:

   ```bash
   oc get configmap inferenceservice-config -n redhat-ods-applications -o yaml > /tmp/rhoai-upgrade-backup/inferenceservice-config-backup.yaml
   ```

**Verification**

* Verify that  the backup file was created:

  ```bash
  ls -lh /tmp/rhoai-upgrade-backup/inferenceservice-config-backup.yaml
  ```

  The command displays the backup file with its size and timestamp.


### **2.10.7. Migrate InferenceServices to RawDeployment mode** {#2.10.7.-migrate-inferenceservices-to-rawdeployment-mode}

Convert your **ModelMesh** and **Serverless** **InferenceServices** to **RawDeployment** mode before upgrading to Red Hat OpenShift AI 3.5. OpenShift AI 3.5 does not include the ModelMesh serving runtime (multi-model serving) and Serverless deployment mode, requiring all model serving workloads to use **RawDeployment** mode.

**Note**

Complete this section only if you are using **ModelMesh** or **Serverless** workloads. The goal is to migrate all existing **InferenceServices** to **RawDeployment**. You can verify your current workload types by running the **rhai-cli** tool; if you are already using **RawDeployment** **exclusively**, you may skip this section and proceed to the next step.

**Important**

You must convert all **InferenceServices** to **RawDeployment** mode before upgrading to OpenShift AI 3.5. Failure to complete this conversion results in HTTP 503 Service Unavailable errors for all inference requests after upgrade.

**Prerequisites**

* You have the required RBAC permissions in the target namespace.

* You are logged into the rhai-cli container, as described in [Log in to the cluster from within the pod.](#1.3.1-log-in-to-the-cluster-from-within-the-pod)

#### **2.10.7.1 Convert Serverless InferenceServices to RawDeployment** {#2.10.7.1-convert-serverless-inferenceservices-to-rawdeployment}

The following procedure describes how to use the **rhai-cli** migrate command to convert  Serverless InferenceServices to RawDeployment.  For manual migration workflows and additional details, see the Knowledge Base article at [Converting ModelMesh and Serverless InferenceServices to RawDeployment](https://access.redhat.com/articles/7134025).

**Procedure**

1. Identify **Serverless** InferenceServices that need to be converted to RawDeployment:

   ```bash
   rhai-cli lint --target-version 3.5 --verbose --checks "*kserve*" --isvc-deployment-mode serverless
   ```

   Example output when Serverless workloads are found:

   ```
   STATUS   KIND     GROUP     CHECK              IMPACT    MESSAGE
   x        kserve   workload  impacted-workloads critical  Found 2...

   Impacted Objects:

     NAME                 NAMESPACE        DEPLOYMENT MODE
     my-serverless-isvc   ml-project-a     Serverless
     another-sl-model     ml-project-b     Serverless

   Summary:
   Total: 7 | Passed: 6 | Warnings: 0 | Failed: 1

   Result:
   FAIL - blocking findings detected
   ```

   If no Serverless workloads are found, skip to the next section, [Convert ModelMesh InferenceServices to RawDeployment](#2.10.7.2-convert-modelmesh-inferenceservices-to-rawdeployment). 

   If Serverless workloads are found, for each of the namespaces listed in the output, repeat the following steps 2 through 5\.

2. Preview the conversion without applying changes:

   ```bash
   rhai-cli migrate run --migration modelserving.serverless-to-raw --target-version 3.5.0 --dry-run
   ```

   The command checks prerequisites and discovers eligible InferenceServices across all namespaces. In dry-run mode, changes are previewed without being applied. 

   Dry-run completion example output:

   ```
   All requested conversions finished.
   ```

   Review the generated files before proceeding to the next step.

3. Run the conversion:

   ```bash
   rhai-cli migrate run --migration modelserving.serverless-to-raw --target-version 3.5.0
   ```

   The command handles resource transformation, authentication resources (ServiceAccount, Role, RoleBinding), and storage credentials automatically across all namespaces.

4. Verify that the converted InferenceServices are ready (run from your workstation due to the need for `jq`):

   ```bash
   oc get isvc -n <NAMESPACE> -o json | jq -r '["NAME","DEPLOYMENT_MODE","READY"], (.items[] | [.metadata.name, .status.deploymentMode, (.status.conditions[] | select(.type=="Ready") | .status)]) | @tsv' | column -t
   ```

   Expected output:

   ```
   NAME                DEPLOYMENT_MODE  READY
   my-serverless-isvc  RawDeployment    True
   another-sl-model    RawDeployment    True
   ```

   All InferenceServices must show **RawDeployment** and **True**.

5. Delete the legacy Serverless InferenceServices:

   1. Preview what will be deleted (run from your workstation due to the need for `jq`):

      ```bash
      oc get isvc -n <NAMESPACE> -o json | jq -r '.items[] | select(.status.deploymentMode == "Serverless" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "Serverless") | .metadata.name'
      ```

      Expected output:

      ```
      my-serverless-isvc
      another-sl-model
      ```

   2. Delete them (run from your workstation due to the need for `jq`):

      ```bash
      oc get isvc -n <NAMESPACE> -o json | jq -r '.items[] | select(.status.deploymentMode == "Serverless" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "Serverless") | .metadata.name' | while read -r name; do echo "Deleting  Serverless InferenceService: $name"; oc delete isvc "$name" -n <NAMESPACE>; done
      ```

      Expected output:

      ```
      Deleting Serverless InferenceService: my-serverless-isvc
      inferenceservice.serving.kserve.io "my-serverless-isvc" deleted
      Deleting Serverless InferenceService: another-sl-model
      inferenceservice.serving.kserve.io "another-sl-model" deleted
      ```

#### 

#### **2.10.7.2 Convert ModelMesh InferenceServices to RawDeployment** {#2.10.7.2-convert-modelmesh-inferenceservices-to-rawdeployment}

The following procedure describes how to use the **rhai-cli** migrate command to convert  ModelMesh InferenceServices to RawDeployment.  For manual migration workflows and additional details, see the Knowledge Base article at [Converting ModelMesh and Serverless InferenceServices to RawDeployment](https://access.redhat.com/articles/7134025).

**Procedure**

1. Identify **ModelMesh** InferenceServices that need to be converted to RawDeployment:

   ```bash
   rhai-cli lint --target-version 3.5 --verbose --checks "*kserve*" --isvc-deployment-mode modelmesh
   ```

   Sample expected output when ModelMesh workloads are found:

   ```
   STATUS   KIND     GROUP     CHECK              IMPACT    MESSAGE
   x        kserve   workload  impacted-workloads critical  Found 1...

   Impacted Objects:

     NAME                 NAMESPACE        DEPLOYMENT MODE
     My-modelmesh-isvc    ml-project-c     ModelMesh

   Summary:
   Total: 7 | Passed: 6 | Warnings: 0 | Failed: 1

   Result:
   FAIL - blocking findings detected
   ```

   If no ModelMesh workloads are found, skip to [Verification of InferenceServices migration](#heading-2). 

   If Serverless workloads are found, for each of the namespaces listed in the output, repeat the following steps 2 through 5\.

2. Preview the conversion without applying changes:

   ```bash
   rhai-cli migrate run --migration modelserving.modelmesh-to-raw --target-version 3.5.0 --dry-run
   ```

   The command discovers ModelMesh InferenceServices and previews runtime template selection and storage configuration. In dry-run mode, changes are previewed without being applied. 

   Dry-run completion example output:

   ```
   DRY-RUN SUMMARY
   ==================

   All YAML resources have been saved to: migration-dry-run-<timestamp>

   Directory structure:
     migration-dry-run-<timestamp>
     +-- new-resources/
     |   +-- inferenceservice/
     |   +-- servingruntime/
     |   +-- secret/
     +-- original-resources/

   Next steps:
     1. Review the generated YAML files
     2. Compare original vs new resources
     3. Apply: find <dir>/new-resources -name '*.yaml' -exec oc apply -f {} \;

   Dry-run completed successfully!
   ```

   Review the generated files before proceeding.

3. Run the conversion:

   ```bash
   rhai-cli migrate run --migration modelserving.modelmesh-to-raw --target-version 3.5.0
   ```

   The command handles runtime template selection, resource transformation, authentication resources, and storage credentials automatically.

   Example completion output:

   ```
   Migration completed successfully!
   ======================================

   Migration Summary:
     Namespace: <namespace>
     InferenceServices migrated: 1
     Models: my-modelmesh-isvc

   Next steps:
     Verify your migrated models are working: oc get inferenceservice -n <namespace>

   Migration helper completed!
   ```

4. Verify that the converted InferenceServices are ready (run from your workstation due to the need for `jq`):

   ```bash
   oc get isvc -n <NAMESPACE> -o json | jq -r '["NAME","DEPLOYMENT_MODE","READY"], (.items[] | [.metadata.name, .status.deploymentMode, (.status.conditions[] | select(.type=="Ready") | .status)]) | @tsv' | column -t
   ```

   Expected output:

   ```
   NAME               DEPLOYMENT_MODE  READY
   my-modelmesh-isvc  RawDeployment    True
   ```

   All InferenceServices must show RawDeployment and True.

5. Delete the legacy ModelMesh InferenceServices and ServingRuntimes after confirming the new RawDeployment services are working.

   1. Preview the ModelMesh InferenceServices that will be deleted (run from your workstation due to the need for `jq`):

      ```bash
      oc get isvc -n <NAMESPACE> -o json | jq -r '.items[] | select(.status.deploymentMode == "ModelMesh" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "ModelMesh") | .metadata.name'
      ```

      Expected output:

      ```
      My-modelmesh-isvc
      ```

   2. Delete them (run from your workstation due to the need for `jq`):

      ```bash
      oc get isvc -n <NAMESPACE> -o json | jq -r '.items[] | select(.status.deploymentMode == "ModelMesh" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "ModelMesh") | .metadata.name' | while read -r name; do echo "Deleting ModelMesh InferenceService: $name"; oc delete isvc "$name" -n <NAMESPACE>; done
      ```

      Expected output:

      ```
      Deleting ModelMesh InferenceService: my-modelmesh-isvc
      inferenceservice.serving.kserve.io "my-modelmesh-isvc" deleted
      ```

   3. Delete the ModelMesh ServingRuntimes (multi-model runtimes) (run from your workstation due to the need for `jq`):

      ```bash
      oc get servingruntimes.serving.kserve.io -n <NAMESPACE> -o json | jq -r '.items[] | select(.spec.multiModel==true) | .metadata.name' | while read -r name; do echo "Deleting ServingRuntime: $name"; oc delete servingruntime "$name" -n <NAMESPACE>; done
      ```

      Expected output:

      ```
      Deleting ServingRuntime: my-modelmesh-runtime
      servingruntime.serving.kserve.io "my-modelmesh-runtime" deleted
      ```

####   {#heading-2}

#### 

#### 

#### 

#### **2.10.7.3 Verification of InferenceServices migration** {#2.10.7.3-verification-of-inferenceservices-migration}

**Procedure**

1. Confirm that no **Serverless** or **ModelMesh** InferenceServices remain:

   ```bash
   rhai-cli lint --target-version 3.5 --verbose --checks "*kserve*"
   ```

   Expected output:

   ```
   STATUS   KIND     GROUP     CHECK              IMPACT   MESSAGE
   ✓        kserve   workload  impacted-workloads info     No Serverless...
   ✓        kserve   workload  impacted-workloads info     No ModelMesh..
   ✓        kserve   workload  impacted-workloads info     No ModelMesh..
   ✓        kserve   workload  impacted-workloads info     No InferenceSe...

   Summary:
     Total: 7 | Passed: 7 | Warnings: 0 | Failed: 0

   Result:
     PASS - all checks passed
   ```

### **2.10.8. Update the inferenceservice-config ConfigMap** {#2.10.8.-update-the-inferenceservice-config-configmap}

Update the **inferenceservice-config** **ConfigMap** to apply hardware profiles ignorelist changes and prevent **InferenceService** redeployment during the upgrade.

**Prerequisites**

* You have cluster administrator access to your OpenShift cluster.

* You have authenticated to the cluster via the OpenShift CLI (**oc**).

* All **InferenceServices** have been successfully migrated to **RawDeployment** mode.

**Prerequisites**

* You have access to the **rhai-cli** tool, as described in  [Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod).

**Procedure**

1. Apply the hardware profiles ignorelist changes:

   ```bash
   rhai-cli migrate run --migration modelserving.hardwareprofiles-ignorelist --target-version 3.5.0
   ```

**Verification**

* Verify that the **inferenceservice-config** **ConfigMap** was updated:

  ```bash
  oc get configmap inferenceservice-config -n redhat-ods-applications -o yaml | grep "hardware" -B 10
  ```

  The ConfigMap displays the updated hardware profiles ignorelist configuration.

### **2.10.9. Update cluster configuration for migration** {#2.10.9.-update-cluster-configuration-for-migration}

Update cluster-wide resources to prepare for the Red Hat OpenShift AI Operator upgrade by configuring the **DataScienceCluster** (DSC), uninstalling removed Operators, and cleaning up legacy components.

**Prerequisites**

* You have cluster administrator access to your OpenShift cluster.

* You have authenticated to the cluster via the OpenShift CLI (**oc**).

* All **InferenceServices** have been successfully migrated to **RawDeployment** mode.

* You have updated the **inferenceservice-config** **ConfigMap** with hardware profiles ignorelist changes.

**Procedure**

1. Update the **DataScienceCluster** (DSC) configuration to use **RawDeployment** as the default deployment mode and disable removed components.

   **Note**: These commands use the DSC v1 API field names (`modelmeshserving`, `serviceMesh`) because they are run against your OpenShift AI 2.25.10 cluster before upgrade. After upgrading to 3.5, the operator automatically converts to the v2 API.

   ```bash
   export DSC_NAME=$(oc get dsc -o jsonpath='{.items[0].metadata.name}')
   oc patch dsc $DSC_NAME --type='merge' -p '{
     "spec": {
       "components": {
         "kserve": {
           "defaultDeploymentMode": "RawDeployment",
           "serving": {
             "managementState": "Removed"
           }
         },
         "modelmeshserving": {
           "managementState": "Removed"
         }
       }
     }
   }'
   ```

2. Update the **DSCInitialization** (DSCI) configuration to remove Service Mesh management:

   ```bash
   export DSCI_NAME=$(oc get dsci -o jsonpath='{.items[0].metadata.name}')
   oc patch dsci $DSCI_NAME --type='merge' -p '{
     "spec": {
       "serviceMesh": {
         "managementState": "Removed"
       }
     }
   }'
   ```

3. If the Serverless Operator is installed, uninstall it: 

   1. In the OpenShift web console, navigate to **Operators** → **Installed Operators**.

   2. For the **Projects** field, make sure that **All projects** are selected.

   3. Locate the **Red Hat OpenShift Serverless** Operator.

   4. Click the Options menu (⋮) and select **Uninstall Operator**.

   5. In the confirmation dialog, select **Delete all operand instances for this operator**.

   6. Click **Uninstall**.

4. If the Service Mesh 2 Operator is installed, uninstall it:

   1. In the OpenShift web console, navigate to **Operators** → **Installed Operators**.

   2. For the **Projects** field, make sure that **All projects** are selected.

   3. Locate the **Red Hat OpenShift Service Mesh 2** Operator.

   4. Click the Options menu (⋮) and select **Uninstall Operator**.

   5. In the confirmation dialog, select **Delete all operand instances for this operator**.

   6. Click **Uninstall**.

5. If standalone Authorino is installed, uninstall it:

   **Note**  
   In Red Hat OpenShift AI 3.x, Authorino is exclusively necessary for use with **LLMInferenceService**. If Authorino has been installed independently as a standalone component, independent from Red Hat Connectivity Link, it must be uninstalled.

   1. Check whether the Red Hat Connectivity Link Operator is installed;

      1. In the OpenShift web console, navigate to **Operators** → **Installed Operators**.

      2. For the **Projects** field, make sure that **All projects** are selected.

      3. Locate the **Red Hat Connectivity Link** Operator.

   If the **Red Hat Connectivity Link** Operator is installed, skip the next step and continue to the next section.

   If the **Red Hat Connectivity Link** Operator is not installed, continue to the next step to uninstall Authorino standalone.

   2. If standalone Authorino is found, uninstall it:

      1. In the OpenShift web console, navigate to **Operators** → **Installed Operators**.

      2. Locate the **Authorino Operator**.

      3. Click the Options menu (⋮) and select **Uninstall Operator**.

      4. In the confirmation dialog, select **Delete all operand instances for this operator**.

      5. Click **Uninstall**.  
         For detailed instructions, see [Deleting Operators from a cluster](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/operators/administrator-tasks#olm-deleting-operators-from-cluster).

### **2.10.10. Prepare distributed inference for migration** {#2.10.10.-prepare-distributed-inference-for-migration}

Prepare your distributed inference (**llm-d**) deployments using **LLMInferenceService** for migration to Red Hat OpenShift AI 3.5 by installing required operators and configuring authentication for your models.

**Note**

Complete this section only if you have distributed inference workloads using **LLMInferenceService**. If you do not use distributed inference with **llm-d**, skip this section and proceed to the final verification step.

**Note**

This section requires collaboration between cluster administrators and users. Cluster administrators install the required operators and configure Red Hat Connectivity Link, while users configure authentication and freeze their **LLMInferenceService** resources. If you are a cluster administrator, coordinate with your users to ensure they complete the authentication and configuration freeze steps.

#### 

#### **2.10.10.1. Install Red Hat Connectivity Link for distributed inference** {#2.10.10.1.-install-red-hat-connectivity-link-for-distributed-inference}

Install and configure Red Hat Connectivity Link (RHCL) to provide authentication functionality for distributed inference workloads using **LLMInferenceService**.

**Prerequisites**

* You have **LLMInferenceService** resources deployed according to the official Red Hat documentation [Deploying models by using Distributed Inference with **llm-d**](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.25/html/deploying_models/deploying_models_on_the_single_model_serving_platform#deploying-models-using-distributed-inference_rhoai-user).

**Procedure**

1. In the OpenShift web console, navigate to **Operators** → **OperatorHub**.

2. Search for "Red Hat Connectivity Link".

   If the Red Hat Connectivity Link Operator is installed, skip to Step 7\.

   If it is not installed, continue to the next step.

3. Click the Operator tile and click **Install**.

4. In the **Install Operator** page, select the following:

   1. **Installation mode**: Select **A specific namespace on the cluster**.

   2. **Installed Namespace**: install the operator in the **kuadrant-system** namespace. If the namespace does not already exist, click this field and select **Create Project** to create the namespace.

5. Click **Install**.

6. Wait for the operator to be installed and ready.

7. Configure Red Hat Connectivity Link by creating the required resources.

   From inside the **rhai-cli** container or your terminal, execute the following commands:

8. Create the `kuadrant-system` namespace:
   ```bash
   oc new-project kuadrant-system
   ```
   
9. Create the Kuadrant custom resource in the `kuadrant-system` namespace:

   ```bash
   oc apply -f - <<EOF
   apiVersion: kuadrant.io/v1beta1
   kind: Kuadrant
   metadata:
     name: kuadrant
     namespace: kuadrant-system
   EOF
   ```

10. Wait for Kuadrant to become ready:

   ```bash
   oc wait Kuadrant -n kuadrant-system kuadrant --for=condition=Ready --timeout=10m
   ```

11. Add the **ServingCert** annotation to the Authorino service:

   ```bash
   oc annotate svc/authorino-authorino-authorization service.beta.openshift.io/serving-cert-secret-name=authorino-server-cert -n kuadrant-system
   ```

   Wait a few seconds for the annotation to be processed:

   ```bash
   sleep 2
   ```

12. Update Authorino to enable SSL:

   ```bash
   oc apply -f - <<EOF
   apiVersion: operator.authorino.kuadrant.io/v1beta1
   kind: Authorino
   metadata:
     name: authorino
     namespace: kuadrant-system
   spec:
     replicas: 1
     clusterWide: true
     listener:
       tls:
         enabled: true
         certSecretRef:
           name: authorino-server-cert
     oidcServer:
       tls:
         enabled: false
   EOF
   ```

13. Verify that the Authorino pods are ready:

   ```bash
   oc wait --for=condition=ready pod -l authorino-resource=authorino -n kuadrant-system --timeout=150s
   ```

**Verification**

* Verify the Red Hat Connectivity Link operator is installed and ready:

  ```bash
  oc get csv -n kuadrant-system | grep rhcl
  ```

  The operator displays a **Succeeded** phase.

  Verify the Kuadrant resource is ready:

  ```bash
  oc get kuadrant kuadrant -n kuadrant-system -o jsonpath='{.status.conditions}' | jq
  ```

  Expected output showing the Kuadrant resource in **Ready** state:

  ```json
  [
    {
      "lastTransitionTime": "YYYY-MM-DDThh:mm:ssZ",
      "message": "Kuadrant is ready",
      "reason": "Ready",
      "status": "True",
      "type": "Ready"
    }
  ]
  ```

* Verify Authorino is configured with TLS enabled:

  ```bash
  oc get authorino -n kuadrant-system authorino -o jsonpath='{.spec.listener.tls.enabled}'
  ```

  The command returns **true**.

* Verify Authorino pods are running:

  ```bash
  oc get pods -n kuadrant-system -l authorino-resource=authorino
  ```

  All Authorino pods display **Running** status with **1/1** ready.

**Additional resources**

* [Installing Connectivity Link on OpenShift](https://docs.redhat.com/en/documentation/red_hat_connectivity_link/1.0/html/installing_connectivity_link_on_openshift/index)

####  **2.10.10.2. Configure Red Hat Connectivity Link for disconnected environments** {#2.10.10.2.-configure-red-hat-connectivity-link-for-disconnected-environments}

Apply additional configuration for Red Hat Connectivity Link when running distributed inference in disconnected environments, including wasm-shim image configuration and mirror registry settings.

**Note**

Complete this procedure only if you are running distributed inference in a disconnected environment. For connected environments, skip this procedure.

**Prerequisites**

* You have installed Red Hat Connectivity Link in the **kuadrant-system** namespace.

* You have access to a mirror registry containing the required Red Hat Connectivity Link images.

* You have cluster administrator access.

**Procedure**

1. Identify the SHA of the wasm-shim image corresponding to the version of Red Hat Connectivity Link you are installing.  
   You can find this information in the Red Hat container catalog at [wasm-shim-rhel9](https://catalog.redhat.com/en/software/containers/rhcl-1/wasm-shim-rhel9/672a1e565d865456f8f2835f).  
   **Note**  
   As of this writing, the SHA for the latest version is  
   sha256:175a1b721a1828ee7bf4369b68722c371b85fe6e7f66b12a94a040b3b493f77f

2. Create the **wasm-plugin-pull-secret** in the **openshift-ingress** namespace by running the following command:

   ```bash
   oc get secret pull-secret -n openshift-config -o json | \
       jq 'del(.metadata.namespace, .metadata.resourceVersion, .metadata.uid, .metadata.creationTimestamp, .metadata.ownerReferences)' | \
       jq '.metadata.name="wasm-plugin-pull-secret"' | \
       oc apply -n openshift-ingress -f -
   ```

3. Configure Kuadrant Operator Subscription with mirrored WASM image by running the following command, replacing **\<WASM_SHIM_SHA\>** with the SHA of the wasm-shim image that you identified in Step 1\.

   ```bash
   export MIRROR_REGISTRY="<BASTION_MIRROR_REGISTRY>:<BASTION_MIRROR_REGISTRY_PORT>"
   export WASM_IMAGE_DIGEST="<WASM_SHIM_SHA>"
   oc patch subscription rhcl-operator -n kuadrant-system --type=merge -p '{
     "spec": {
       "config": {
         "env": [
           {
             "name": "RELATED_IMAGE_WASMSHIM",
             "value": "oci://${MIRROR_REGISTRY}/rhcl-1/wasm-shim-rhel9@${WASM_IMAGE_DIGEST}"
           },
           {
             "name": "PROTECTED_REGISTRY",
             "value": "${MIRROR_REGISTRY}"
           }
         ]
       }
     }
   }'
   ```

4. Configure the Gateway to trust the mirror registry certificate by creating a ConfigMap that injects the WASM\_INSECURE\_REGISTRIES environment variable into the Gateway pod, using the following commands:

   ```bash
   export MIRROR_REGISTRY="<BASTION_MIRROR_REGISTRY>:<BASTION_MIRROR_REGISTRY_PORT>"
   export GATEWAY_NAME=<YOUR_GATEWAY_NAME>
   oc apply -f - <<EOF
       apiVersion: v1
       kind: ConfigMap
       metadata:
         name: ${GATEWAY_NAME}-config
         namespace: openshift-ingress
       data:
         deployment: |
           Spec:
             template:
               spec:
                 containers:
                 - name: istio-proxy
                   env:
                   - name: WASM_INSECURE_REGISTRIES
                     value: ${MIRROR_REGISTRY}
   EOF
   oc patch gateway ${GATEWAY_NAME} -n openshift-ingress --type=merge -p '{"spec":{"infrastructure":{"parametersRef":{"group":"","kind":"ConfigMap","name":"'${GATEWAY_NAME}'-config"}}}}'
   ```

**Verification**

* Verify the RHCL operator subscription includes the mirror registry configuration:

  ```bash
  oc get subscription rhcl-operator -n kuadrant-system -o jsonpath='{.spec.config.env}' | jq
  ```

  The output displays your configured wasm-shim image location:

  ```json
  [
    {
      "name": "RELATED_IMAGE_WASMSHIM",
      "value": "oci://${MIRROR_REGISTRY}/rhcl-1/wasm-shim-rhel9@${WASM_IMAGE_DIGEST}"
    },
    {
      "name": "PROTECTED_REGISTRY",
      "value": "${MIRROR_REGISTRY}"
    }
  ]
  ```

#### **2.10.10.3. Configure authentication for LLMInferenceService resources** {#2.10.10.3.-configure-authentication-for-llminferenceservice-resources}

Configure authentication for your **LLMInferenceService** resources to handle security changes in Red Hat OpenShift AI 3.5 and later, where distributed inference is secure by default.

**Prerequisites**

* You have **LLMInferenceService** resources deployed.

* Red Hat Connectivity Link is installed and configured.

* You have access to projects containing **LLMInferenceService** resources.

**Procedure**

1. Configure authentication for your **LLMInferenceService** resources.  
   Choose one of the following methods:  
   **Method 1: Disable authentication for development and testing**  
   If you don't need authentication for your model, disable it by annotating the **LLMInferenceService**:

   ```bash
   oc annotate llminferenceservice <LLMISVC_NAME> -n <LLMISVC_NAMESPACE> security.opendatahub.io/enable-auth=false
   ```

   Replace **\<LLMISVC_NAME\>** with your **LLMInferenceService** name and **\<LLMISVC_NAMESPACE\>** with your project namespace.  
   The model becomes public and no authentication tokens are required.  
   **Method 2: Configure RBAC access control (Recommended)**  
   To keep the model secure, create a **ServiceAccount** with permissions to access the **LLMInferenceService**:

2. Create a **ServiceAccount**:

   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: my-llmisvc-sa
     namespace: <MY_PROJECT>
   ```

3. Create a **Role** with **get** permission:

   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: my-llmisvc-role
     namespace: <MY_PROJECT>
   rules:
   - apiGroups: ["serving.kserve.io"]
     resources: ["llminferenceservices"]
     resourceNames: ["<MY_LLM_NAME>"]
     verbs: ["get"]
   ```

   Create a **RoleBinding**:

   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: my-llmisvc-rolebinding
     namespace: <MY_PROJECT>
   subjects:
   - kind: ServiceAccount
     name: my-llmisvc-sa
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: Role
     name: my-llmisvc-role
   ```

   Update your client applications to include the **Authorization** header:

   ```bash
   TOKEN=$(oc create token my-llmisvc-sa -n <MY_PROJECT>)
   curl -H "Authorization: Bearer $TOKEN" https://<MODEL_URL>/v2/models/...
   ```

**Verification**

* Verify **LLMInferenceService** resources are ready:

  ```bash
  oc get llminferenceservices --all-namespaces
  ```

  All **LLMInferenceService** resources display a **Ready** status.

**Note**

In Red Hat OpenShift AI 3.5 and later, distributed inference with **LLMInferenceService** is secure by default. Applications that attempt to connect without proper authentication will receive HTTP 403 Forbidden errors. Choose the authentication method that best matches your security requirements.

#### **2.10.10.4. Freeze LLMInferenceService configuration for upgrade** {#2.10.10.4.-freeze-llminferenceservice-configuration-for-upgrade}

Pin your **LLMInferenceService** configurations to use Red Hat OpenShift AI 2.25.10 (and later) templates to prevent scheduler pod failures during the upgrade to version 3.5.

**Prerequisites**

* You have **LLMInferenceService** resources deployed.

* You have access to projects containing **LLMInferenceService** resources.

**Procedure**

1. Pin **LLMInferenceService** configurations to use RHOAI 2.25.10 (and later) templates to prevent scheduler pod failures during upgrade:

   ```bash
   oc patch llmisvc <LLMISVC_NAME> -n <LLMISVC_NAMESPACE> \
       --subresource=status \
       --type=merge \
       -p '{ "status": { "annotations": { "serving.kserve.io/config-llm-template": "kserve-config-llm-template", "serving.kserve.io/config-llm-decode-template": "kserve-config-llm-decode-template", "serving.kserve.io/config-llm-worker-data-parallel": "kserve-config-llm-worker-data-parallel", "serving.kserve.io/config-llm-decode-worker-data-parallel": "kserve-config-llm-decode-worker-data-parallel", "serving.kserve.io/config-llm-prefill-template": "kserve-config-llm-prefill-template", "serving.kserve.io/config-llm-prefill-worker-data-parallel": "kserve-config-llm-prefill-worker-data-parallel", "serving.kserve.io/config-llm-scheduler": "kserve-config-llm-scheduler", "serving.kserve.io/config-llm-router-route": "kserve-config-llm-router-route" } } }'
   ```

   Replace **\<LLMISVC_NAME\>** with your **LLMInferenceService** name and **\<LLMISVC_NAMESPACE\>** with your project namespace.

   **Important**  
   If you are overriding **LLMInferenceService** scheduler arguments, you must update them for Red Hat OpenShift AI 3.x compatibility. The following breaking changes apply in Red Hat OpenShift AI 3.5 and later:

   * Argument naming: Arguments changed from **camelCase** to **kebab-case** (for example, **\--certPath** becomes **\--cert-path**)

   * TLS certificate path: Default location moved from **/etc/ssl/certs** to **/var/run/kserve/tls**

   * Mandatory TLS: Signed TLS certificates via OpenShift service signer are required

   * Required configuration: Must include **\--cert-path** argument and **SSL\_CERT\_DIR** environment variable

     Following is an example Diff of an LLMInferenceService configuration for the scheduler arguments and environment variables from 2.25.10 (and later) to 3.5:

     ```yaml
     kind: LLMInferenceService
     # ...
     spec:
       router:
         scheduler:
           containers:
           - name: main
             env:
             - name: SSL_CERT_DIR
               value: "/var/run/secrets/kubernetes.io/serviceaccount:/etc/pki/tls/certs"
             args:
             - "--cert-path"
             - "/var/run/kserve/tls"
             volumeMounts:
             - mountPath: /var/run/kserve/tls
               name: tls-certs
               readOnly: true
     ```

**Verification**

* Verify that the **LLMInferenceService** configuration has been frozen:

  ```bash
  oc get llmisvc <LLMISVC_NAME> -n <LLMISVC_NAMESPACE> -o jsonpath='{.status.annotations}'
  ```

  The output displays the pinned template annotations.

* Verify that the **LLMInferenceService** migration steps were completed correctly from inside the **rhai-cli** container:

  ```bash
  /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5
  ```

  The command completes without errors and confirms migration readiness.

### **2.10.11. Verify migration readiness** {#2.10.11.-verify-migration-readiness}

Verify that all migration prerequisites have been completed and your cluster is ready for the Red Hat OpenShift AI operator upgrade from version 2.25.10 (and later) to 3.5.

**Prerequisites**

* You have completed cluster preparation procedures.

* You have converted all **InferenceServices** to **RawDeployment** mode.

* If applicable, you have prepared **LLMInferenceService** resources for migration.

**Procedure**

1. Perform a comprehensive readiness check from inside the **rhai-cli** container:

   ```bash
   /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5 --checks "*kserve*" --checks "*modelmesh*"
   ```

   Review the output to determine if your cluster is ready to proceed with the upgrade.

**Verification**

If the **rhai-cli** output shows no critical issues, you can proceed to the next section.

If the **rhai-cli** output identifies issues:

* Review the component-specific sections in this guide to address the reported issues

* Complete the migration steps for any components flagged with critical impact

* Run **rhai-cli lint --target-version 3.5** again to confirm all issues are resolved

**Important**

Do not proceed with the Red Hat OpenShift AI operator upgrade from version 2.25.10 (and later) to 3.5 if the **rhai-cli** output shows any critical issues. Address all critical issues by completing the relevant component migration procedures before upgrading.
