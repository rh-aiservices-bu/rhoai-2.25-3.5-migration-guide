# **Chapter 1\. Assess and plan for migration** {#chapter-1.-assess-and-plan-for-migration}

## **1.1. Overview of OpenShift AI migration** {#1.1.-overview-of-openshift-ai-migration}

Red Hat OpenShift AI 3.5 is the first 3.x release to support migration from OpenShift AI 2.25.9 (and later) . The OpenShift AI 3.x release introduces significant technology and component changes, making a direct upgrade from 2.25.9 (and later) technically complex.

This guide provides step-by-step instructions on how to migrate from OpenShift AI 2.25.9 (and later) to 3.5.

### **1.1.1. Overview of migration assessment steps**  {#1.1.1.-overview-of-migration-assessment-steps}

1. Open a proactive support case through the Red Hat customer portal at [access.redhat.com](http://access.redhat.com) to let Red Hat know that you are considering upgrading to Red Hat OpenShift AI 3.5 , as described in [How to submit a Proactive Case](https://access.redhat.com/articles/5387111).

2. Determine your migration approach: side-by-side migration or in-place migration.

3. For in-place migration, ensure that you have a full and complete backup of your OpenShift and OpenShift AI environment before you begin the migration process. Because the migration involves many changes, you need to have a backup in case the process does not go as planned. 

   **NOTE:** OpenShift and OpenShift AI do not support rollbacks once you initiate the migration.

4. Verify that your environment meets the required prerequisites before you upgrade, as described in [Prerequisites for OpenShift AI migration](#heading).

5. Deploy a long-living pod for the container image that contains the migration assessment linting CLI and migration actions for specific component migrations. See [Deploy a persistent pod on your cluster and download the rhai-cli container image.](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image)

6. Run the migration assessment script, as described in [Using the migration assessment script](#heading-1).  
   The **rhai-cli** migration assessment script is a command-line utility designed to help administrators perform a gap analysis of your OpenShift AI environment. It identifies workloads and configurations that are impacted by the migration from OpenShift AI version 2.25.9 (and later) to 3.5.

7. Submit the results of the migration assessment script to Technical Support, as described in [Submit the assessment output to Red Hat Technical Support](#1.5-submit-the-assessment-output-to-red-hat-technical-support)

   NOTE: You must run the rhai-cli script a number of times in order to track the progress of the migration.

8. Based on the results of the migration script, complete the steps for each component in the order they are described in this guide.

   Use the provided **rhai-cli migrate** actions to perform specific migration tasks and modify resources when executed.

##   {#heading}

## **1.2. Prerequisites for OpenShift AI migration** {#1.2.-prerequisites-for-openshift-ai-migration}

Before you begin the OpenShift AI migration, verify that your environment meets the following prerequisites:

**Have OpenShift version 4.19.9 or later**

Your OpenShift cluster must be at least version 4.19.9. If it is not, follow the upgrade instructions in the product documentation. For example, for OpenShift Container Platform 4.18, see [Updating clusters | OpenShift Container Platform](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/updating_clusters/index).

**Determine your migration approach**

There are two primary approaches for migrating from OpenShift AI 2.25.9 (and later) to 3.5:

* Side-by-side environment: This approach involves setting up a second environment that runs OpenShift AI 3.5 alongside your existing 2.25.9 (and later) environment. Because your existing OpenShift AI 2.25.9 (and later) environment is left untouched, the risk of unrecoverable cluster state is greatly minimized, and a full cluster backup of the 2.25.9 (and later) environment is not a critical requirement, merely a recommended practice. This approach allows for a period of overlap where both environments are operational. You have a much longer window to either recreate or move content from your 2.25.9 (and later) environment to your 3.5 environment.

* In-place migration: This approach involves using a single environment and executing the in-place migration steps documented in this guide. An in-place migration might be necessary if you must complete the migration within a very strict timeframe, for example, within a set maintenance window. This approach carries the highest effort and risk because the existing OpenShift AI 2.25.9 (and later) environment is directly modified. If you choose this approach, a robust, full cluster backup is mandatory because it is the only mechanism for rollback.

**Notify users of your migration plan**

OpenShift AI does not support Zero Downtime Upgrade. Notify your users of your migration plan to make them aware of potential disruptions that could happen during the migration process.

**Back up your OpenShift and OpenShift AI environment** 

If you decide to use an in-place migration approach, you must create a full and complete backup of your OpenShift and OpenShift AI environment before you begin the migration process. Because the migration involves many changes, you need to have a backup in case the process does not go as planned. OpenShift and OpenShift AI do not support rollbacks once you initiate the migration. This backup is needed especially for an in-place migration method, where you modify your existing 2.25 OpenShift AI environment.

NOTE: If you opt for a side-by-side migration which leaves the 2.25 OpenShift AI environment completely untouched, the full and complete backup before migration is a recommendation, rather than a mandatory requirement.

**Check the management status of the Kueue component**  
The migration assessment script checks your OpenShift AI 2.25.9 (and later) installation to determine the status of the Kueue component management. In OpenShift AI 3.5, the supported Kueue management states are **Removed** and **Unmanaged**. The **Managed** state is accepted by OLM for backwards compatibility but is rejected at runtime.

Optionally, you can check the status of the Kueue component by running the following command:

```bash
oc get datasciencecluster -A -o jsonpath='{.items[0].spec.components.kueue.managementState}{"\n"}'
```

* If the output is **Removed**, you can migrate from OpenShift AI 2.25.9 (and later) to 3.5. Kueue will be disabled after upgrade.

* If the output is **Unmanaged**, you can migrate from OpenShift AI 2.25.9 (and later) to 3.5. OpenShift AI will integrate with an externally installed Red Hat build of Kueue (RHBOK) Operator. Before upgrading, ensure that the Red Hat build of Kueue Operator is installed and operational. You can use the `rhai-cli` RHBOK migration action to migrate from embedded Kueue to external RHBOK. See [Set Kueue management to Removed](#2.2-set-kueue-management-to-removed) for details.

* If the output is **Managed**, you must migrate to either **Removed** or **Unmanaged** before upgrading. The **Managed** state is not supported at runtime in OpenShift AI 3.5. If you want to continue using Kueue, migrate to the **Unmanaged** state with an external Red Hat build of Kueue Operator. If you do not need Kueue, set the state to **Removed**.

 When you set the Kueue component management state to Removed, Kueue is disabled in OpenShift AI. If the state was previously Managed, OpenShift AI uninstalls the embedded Kueue distribution. If the state was previously Unmanaged, OpenShift AI stops checking for the external Kueue integration but does not uninstall the Red Hat build of Kueue Operator.

**Coordinate permissions and planning**

The process of migrating from OpenShift AI 2.25.9 (and later) to 3.5 requires internal coordination planning for your company.

Different steps in the migration process require different roles and permissions:

Cluster administrator:

* The primary owner of the migration process

* Performs the upgrade and coordinates migration activities

* Requires cluster-admin privileges and access to a system with Bash and the oc CLI

* Must consult with OpenShift AI administrators and users during the planning phase.

OpenShift AI administrator:

* Has access to the OpenShift AI dashboard and projects.

* Handles workbench images and custom runtimes.

* Administers user groups.

OpenShift AI user:

* Has access to the OpenShift AI dashboard and projects

* The user can have different job descriptions in your company, such as data scientist, AI engineer, or ML Ops engineer.

* Involvement is optional if the administrator handles all migration tasks.

## **1.3 Deploy a persistent pod on your cluster that includes the**  **the rhai-cli container image** {#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image}

To prepare for the migration of OpenShift AI 2.25.9 (and later) to 3.5,  deploy a long-lived pod on your OpenShift cluster. This pod provides the following conditions for the migration process:

* A **StatefulSet** that runs `sleep infinity` to keep the pod alive. The pod name rhai-cli-0  is stable.

* A persistent volume claim (PVC) mounted at /tmp/rhoai-upgrade-backup for persisting  reports and backup artifacts between log in sessions.

  WARNING: You must make sure to preserve this PVC.

* As part of the pod configuration, specify the rhai-cli container image.

  The container image is available at {{RHAI_CLI_IMAGE}}.

  This image contains the Red Hat AI command line interface(**rhai-cli)** utility that includes the migration assessment linting CLI and migration actions to assist with pre-upgrade and post-upgrade steps for the Model Serving, Workbenches, TrustyAI, Llama Stack / OGX, AI Pipelines, and Ray Training Operator components.

  For details about the container image, including versions, see the [**rhoai/rhai-cli-rhel9** page in the Red Hat Ecosystem Catalog](https://catalog.redhat.com/en/software/containers/rhoai/rhai-cli-rhel9/69a580e6a46d08df99bffe08?image=69a7dc1675d4eb16e91cb5de).

  **Important**  
  **Note for disconnected environments**

  If you are running in an air-gapped environment with limited or no internet connectivity, follow your internal company procedure to mirror this image to a local registry or make the image available within your network

**Prerequisites**

* You have the OpenShift **oc** command line interface installed and configured to access your cluster.

* You have an existing target namespace where you want to deploy the pod.

* You have permission to create and delete StatefulSets and PVCs in the target namespace.

* You have a [redhat.com](http://redhat.com) account that allows you to pull images from the Red Hat registry. 

**Procedure**

1. In a terminal window, log in to your OpenShift cluster.  
2. Create the StatefulSet, replacing `<namespace>` with the name of the target namespace:

   ```bash
   cat <<'EOF' | oc apply -n <namespace> -f -
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: rhai-cli
   spec:
     serviceName: rhai-cli
     replicas: 1
     selector:
       matchLabels:
         app: rhai-cli
     template:
       metadata:
         labels:
           app: rhai-cli
       spec:
         containers:
           - name: rhai-cli
             image: {{RHAI_CLI_IMAGE}}
             command:
               - sleep
               - infinity
             resources:
               requests:
                 cpu: 100m
                 memory: 128Mi
             volumeMounts:
               - name: backup
                 mountPath: /tmp/rhoai-upgrade-backup
     volumeClaimTemplates:
       - metadata:
           name: backup
         spec:
           accessModes:
             - ReadWriteOnce
           # storageClassName: <your-storage-class>  # set if PVC stays Pending
           resources:
             requests:
               storage: 1Gi
   EOF
   ```

   

3.  Wait for the pod to be ready:

   ```bash
   oc wait pod/rhai-cli-0 -n <namespace> --for=condition=Ready --timeout=120s
   ```

   If the wait times out, run the following commands

   ```bash
   oc get pods -n <namespace>
   oc describe pod rhai-cli-0 -n <namespace>
   ```

4. Depending on your cluster policy and workload, customize the container (`cpu`/`memory`) and PVC (`storage`) `resources.requests` values.

   If the PVC stays `Pending`, set `spec.volumeClaimTemplates[0].spec.storageClassName` to match a StorageClass in your cluster. Ask your OpenShift administrator for the correct value if you are unsure.

**Verification**

The rhai-cli-0  pod is in a **Ready** state.

**Next step**

[Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod)


### **1.3.1 Log in to the cluster from within the pod** {#1.3.1-log-in-to-the-cluster-from-within-the-pod}

Authentication for the cluster is handled when you log in from inside the pod. The migration assessment script uses your own credentials; no additional service account or role binding is required for the pod itself. The pod uses the default ServiceAccount namespace; it is sufficient because API access uses your user token after you log in, not the pod service account.

**Prerequisites**

* You deployed a pod, as described in [Deploy a persistent pod on your cluster that includes the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

* You have your OpenShift API server URL and a valid authentication token.

**Procedure**

1. Open a shell in the pod:

   ```bash
   oc exec -it rhai-cli-0 -n <namespace> -- /bin/bash
   ```

2.  Inside that shell, point `KUBECONFIG` to a writable path and log in:

   ```bash
   export KUBECONFIG=/tmp/.kubeconfig
   oc login --token=<token> --server=<api-server-url>
   ```

   **NOTE:** If your cluster uses a custom or self-signed API serving certificate, `oc login` fails with `certificate signed by unknown authority`. To resolve this, either provide the cluster CA certificate (recommended) or bypass the certificate check:

   ```bash
   # Recommended: provide the cluster CA certificate
   oc login --token=<token> --server=<api-server-url> --certificate-authority=<ca-file>

   # Alternative: bypass the certificate check (insecure)
   oc login --token=<token> --server=<api-server-url> --insecure-skip-tls-verify=true
   ```

**NOTE: If you have closed your session or open a new session you will need to peform the `export` and `oc login` again in the new session. 

**Verification**

1. You successfully logged in to OpenShift from within the rhai-cli-0 pod.

2. Verify the rhai-cli version:

   ```bash
   /opt/rhai-cli/bin/rhai-cli version
   ```

   Expected output:

   ```
   kubectl-odh version 1.26.4 (commit: unknown, built: unknown)
   ```

**Next steps**

* [Use the rhai-cli migration assessment script](#heading-1)

### **1.3.2. About the rhai-cli container image** {#1.3.2.-about-the-rhai-cli-container-image}

The container image is available at **{{RHAI_CLI_IMAGE}}**. It contains the migration assessment linting CLI and migration actions for specific component migrations.

For details about the container image, including versions, see the [**rhoai/rhai-cli-rhel9** page in the Red Hat Ecosystem Catalog](https://catalog.redhat.com/en/software/containers/rhoai/rhai-cli-rhel9/69a580e6a46d08df99bffe08?image=69a7dc1675d4eb16e91cb5de).

**Important**  
**Note for disconnected environments** If you are running in an air-gapped environment with limited or no internet connectivity, follow your internal company procedure to mirror this image to a local registry or make the image available within your network.

The following table lists the locations of the resources included in the **rhai-cli** container image:

| Resource | Path | Description |
| :---- | :---- | :---- |
| CLI binary | /opt/rhai-cli/bin/rhai-cli | Performs the cluster scan, migration readiness analysis, and component migration tasks. |

The following table lists the **rhai-cli migrate** actions that are registered in the rhai-cli tool for automated migration tasks:

| Component | Action name | Description |
| :---- | :---- | :---- |
| Kueue/RHBOK | Migrate Kueue to Red Hat build of Kueue | Migrates from OpenShift AI built-in Kueue to Red Hat Build of Kueue operator. |
| AI Pipelines | AI Pipelines pre-upgrade check | Captures DSPA pod health, migrates v1alpha1 resources to v1, and detects RBAC gaps. |
| AI Pipelines | Update custom DSP roles | Patches custom RBAC roles to add datasciencepipelinesapplications/api subresource. |
| AI Pipelines | AI Pipelines post-upgrade check | Verifies pipeline server pods are healthy post-upgrade against pre-upgrade baseline. |
| Model Serving | Convert Serverless InferenceServices to RawDeployment | Converts InferenceServices using Serverless deployment mode to RawDeployment and creates associated auth resources (ServiceAccount, Role, RoleBinding). |
| Model Serving | Convert ModelMesh InferenceServices to RawDeployment | Converts InferenceServices using ModelMesh deployment mode to RawDeployment, updates associated ServingRuntimes, and creates auth resources. |
| Model Serving | Add hardware profile annotations to ignore list | Patches inferenceservice-config ConfigMap to set opendatahub.io/managed=false and add hardware-profile annotations to serviceAnnotationDisallowedList before upgrading to RHOAI 3.x. |
| Model Serving | Add owner references to auth resources | Patches ServiceAccounts, Roles, and RoleBindings associated with RawDeployment InferenceServices with ownerReferences pointing to the ISVC. |
| Model Serving | Restore managed inferenceservice-config | Sets opendatahub.io/managed=true on inferenceservice-config ConfigMap and restarts KServe controller after upgrading to RHOAI 3.x. |
| Workbenches | Upgrade workbenches from 2.x to 3.x | Updates workbench notebook container names for 3.x compatibility. |
| Workbenches | Attach Kueue queue-name label to workbenches | Adds the kueue.x-k8s.io/queue-name label to notebooks in Kueue-managed namespaces. |
| Workbenches | Clean up legacy OAuth resources from workbenches | Removes stale OAuth-proxy resources (Route, Service, Secrets, OAuthClient). |
| Workbenches | Patch workbench auth model from OAuth-proxy to kube-rbac-proxy | Migrates Notebook CRs from oauth-proxy (2.x) to kube-rbac-proxy (3.x) auth. |
| Workbenches | Verify workbench migration status | Verifies migration and cleanup status of workbench notebooks. |
| TrustyAI | Break GPU deployment deadlocks | Detects and fixes GPU deployment deadlocks caused by TrustyAI patching InferenceServices. |
| TrustyAI | Patch GuardrailsOrchestrator readinessProbe | Adds readinessProbe to GuardrailsOrchestrator deployments. |
| TrustyAI | Migrate GuardrailsOrchestrator otelExporter schema | Migrates otelExporter from RHOAI 2.25 schema to current schema. |
| TrustyAI | Backup and restore TrustyAI scheduled metrics | Backups scheduled metrics via REST API before upgrade, restores after upgrade. |
| TrustyAI | Backup and restore TrustyAI data storage | Backups and restores TrustyAI PVC files or MariaDB database. |
| Training | Verify training workloads | Pre-upgrade check for Kubeflow v1 training workloads requiring migration to Trainer v2 TrainJob. |
| Llama Stack | Backup LlamaStack Resources | Backups all LlamaStack resources before upgrade (LlamaStack renamed to OGX in 3.5). |
| Ray | Backup RayCluster resources | Creates backup of RayCluster configurations before migration. |
| Ray | Migrate RayClusters | Migrates RayClusters in-place for RHOAI 3.x compatibility. |

##  {#heading-1}

## **1.4. Run the migration assessment script** {#1.4.-run-the-migration-assessment-script}

The **rhai-cli** migration assessment script is a command-line utility designed to help administrators perform a gap analysis of your OpenShift AI environment. It identifies workloads and configurations that are impacted by the migration from OpenShift AI version 2.25.9 (and later) to 3.5.

**Important**

**\*** The **rhai-cli** script is a diagnostic utility and will not perform any changes to your cluster. It is intended to guide your migration strategy by identifying gaps.

**\*** You must run this script on your 2.25.9 (and later) cluster. If you run the **rhai-cli** script on a cluster that has already been upgraded, it will not accurately report issues or required actions.

While the lint script itself is non-intrusive, the **rhai-cli migrate** actions are designed to perform specific migration tasks and will modify resources when executed.

You must run the **rhai-cli** script a number of times in order to track the progress of the migration via the script execution output. If there are blocking items (items listed with **critical or prohibited**  impact) in the script output, you must resolve them by using the pre-upgrade steps in the respective component tab, or the **rhai-cli migrate** actions where available.

As you move through the pre-upgrade steps, you should see a reduction in the number of critical items. Depending on the cluster set up, you might see additional items appear in your script output as you resolve certain items.

**Note**  
Migration blockers appear with **prohibited** or **critical** impact in the script output.   
After you resolve a blocker, re-run the lint command to confirm that the critical item no longer appears.

 For a description of the script output, see  [Understand the migration assessment script output](#1.4.1-understand-the-migration-assessment-script-output).

**Prerequisites**

* You have logged in to OpenShift as described in [Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod).

* You have cluster administrator privileges for your OpenShift cluster.

**Procedure**

1. For a full cluster scan, run the following command:

   ```bash
   /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5
   ```

2. Optional. Review the output. For more information, see  [Understand the migration assessment script output](#1.4.1-understand-the-migration-assessment-script-output).

3. Optional. For information about other options that you can use to filter the script output, run the rhai-cli lint command with the \--help flag:

   ```bash
   /opt/rhai-cli/bin/rhai-cli lint --help
   ```

**Verification**

The migration assessment script provides output about your pre-migration status.

**Next steps**

[Submit the assessment output to Red Hat Technical Support](#1.5-submit-the-assessment-output-to-red-hat-technical-support)

### **1.4.1 Understand the migration assessment script output** {#1.4.1-understand-the-migration-assessment-script-output}

The **rhai-cli** **lint** command produces a migration assessment report that includes the following categories:

* **STATUS**: A status icon that indicates the severity of the item: ✗ for **critical**, ⚠ for **warning**, and ✓ for **info**.

* **KIND:**  A specific OpenShift AI component, dependency, or resource. For example, kserve, notebook, cert-manager, or datasciencepipelines.

* **GROUP**:  The diagnostic category that classifies the assessment check. Groups include: dependency, service, component, and workload. 

* **CHECK**: The type of check for the item. For example, a **version-requirement** check might validate if your environment meets the minimum version requirement for a particular software.

* **IMPACT**: The severity of the item. Blocking issues have a **critical** or **prohibited** impact.

* **MESSAGE**: A description of the item. For **critical** items, this field indicates required actions to resolve blocking issues.

You can assess output items by impact:

| Impact | Description | Action required                                                                                                                                          |
| :---- | :---- |:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| prohibited | Upgrade is not possible. | Do not continue with the upgrade. Check the component's section of this document to see if a solution is documented. If not, check with Red Hat Support. |
| critical | A blocker for the upgrade. The component or workload will fail. | You must fix this blocking item by using the pre-upgrade steps in the respective component tab, or the **rhai-cli migrate** actions where available.     |
| warning | Potential issues or deprecated fields. | Review and remediate this item to ensure the long-term stability of your OpenShift AI environment.                                                       |
| info | Prerequisite met or no migration required. | No action required.                                                                                                                                      |

**Note**

The `rhai-cli lint` command uses the following exit codes:

| Exit code | Meaning |
| :---- | :---- |
| 0 | All checks passed with no prohibited or critical findings. |
| 1 | One or more prohibited or critical findings were detected. This is expected behavior, not a tool error. |

**Important**

Before upgrading to OpenShift AI 3.5, ensure that no items with **prohibited** or **critical** impact appear in the migration assessment script output.

**Additional rhai-cli lint script commands**

To reduce output noise, you can also run a focused check for a specific component by using the \--checks flag for a component listed in the following table.  Enclose the component string value with wildcard (\*) characters:

```bash
/opt/rhai-cli/bin/rhai-cli lint --target-version 3.5 --checks *<component-string>*
```

For example, to perform a targeted check on the AI Pipelines component, run the following command :

```bash
/opt/rhai-cli/bin/rhai-cli lint --target-version 3.5 --checks *datasciencepipelines*
```

| Component | Value for – \- check option  |
| :---- |:-----------------------------|
| OpenShift AI dashboard | **\*dashboard\***            |
| AI Pipelines | **\*datasciencepipelines\*** |
| TrustyAI Guardrails | **\*guardrails\***           |
| KServe | **\*kserve\***               |
| Kueue | **\*kueue\***                |
| Llama Stack / OGX | **\*llamastack\***           |
| Model Serving | **\*modelmesh\***            |
| Workbenches | **\*notebook\***             |
| Ray Training Operator | **\*ray\***                  |
| Kubeflow Training Operator | **\*trainingoperator\***     |


After you resolve each blocking item, re-run the **lint** command to confirm that the item no longer appears in the script output.

**Important**

Before upgrading to OpenShift AI 3.5, ensure that no items with **critical** or **prohibited** impact appear in the migration assessment script output.

**Retrieve Data**

Run the following command from outside the pod:

```bash
oc cp -n <namespace> rhai-cli-0:/tmp/rhoai-upgrade-backup ./local-reports
```

## 

## 

## **1.5 Submit the assessment output to Red Hat Technical Support** {#1.5-submit-the-assessment-output-to-red-hat-technical-support}

Submit the results of the migration assessment script to Technical Support.

**Prerequisites**

* You deployed a pod on your OpenShift cluster as described in [Deploy a persistent pod on your cluster that includes the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

  * The pod name is rhai-cli-0.  
  * The PVC is mounted at /tmp/rhoai-upgrade-backup.

* You have logged in to OpenShift as described in [Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod).

* You have cluster administrator privileges for your OpenShift cluster.

**Procedure**

1. Run the following command that sends the output of the migration script to a YAML file.

   ```bash
   /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5 --output yaml > /tmp/rhoai-upgrade-backup/<filename>.yaml
   ```

   For example,  to send the output of the migration script to a YAML file named rhai-cli-output.yaml:

   ```bash
   /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5 --output yaml > /tmp/rhoai-upgrade-backup/rhai-cli-output.yaml
   ```

2. Copy the output file from the rhai-cli container to your local workstation:

   1. Open a new terminal window for your local workstation.

   2. Copy the output file from the rhai-cli container to your local workstation. \<namespace\> is the namespace where you deployed the pod that includes the rhai-cli container image:

      ```bash
      oc cp <namespace>/rhai-cli-0:/tmp/rhoai-upgrade-backup/<filename>.yaml ./<filename>.yaml
      ```

      For example, if you deployed the pod that includes the rhai-cli container image in the rhai-migration namespace, run the following command to copy a YAML file named rhai-cli-output.yaml located in the /tmp/rhoai-upgrade-backup directory of the rhai-cli container to the current directory of your local workstation:

      ```bash
      oc cp rhai-migration/rhai-cli-0:/tmp/rhoai-upgrade-backup/rhai-cli-output.yaml ./rhai-cli-output.yaml
      ```

3. If you have not already done so, open a proactive support case through the Red Hat customer portal at [access.redhat.com](http://access.redhat.com). to let Red Hat know that you are considering upgrading Red Hat OpenShift AI 2.25.9 (and later) to 3.5 , as described in [How to submit a Proactive Case](https://access.redhat.com/articles/5387111).

4. Upload the YAML output file as an attachment to your Red Hat support case. For more information about uploading a file to your support case, see [How to provide files to Red Hat Support](https://access.redhat.com/solutions/2112).

**Verification**

The YAML file is an attachment that you can see in the case history for your Red Hat support case.

