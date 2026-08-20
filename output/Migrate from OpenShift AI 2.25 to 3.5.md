# **Migrate from OpenShift AI 2.25.9  (and later) to 3.5 (latest) DRAFT** 

#### **Migrate from Red Hat OpenShift AI 2.25.9 (and later) to OpenShift AI 3.5 (latest)**  

## **Table of Contents**

---

[Preface](#preface)

[Chapter 1\. Assess and plan for migration](#chapter-1.-assess-and-plan-for-migration)

[1.1. Overview of OpenShift AI migration](#1.1.-overview-of-openshift-ai-migration)

[1.1.1. Overview of migration assessment steps](#1.1.1.-overview-of-migration-assessment-steps)

[1.2. Prerequisites for OpenShift AI migration](#1.2.-prerequisites-for-openshift-ai-migration)

[1.3 Deploy a persistent pod on your cluster that includes the](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image)   
[the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image)

[1.3.1 Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod)

[1.3.2. About the rhai-cli container image](#1.3.2.-about-the-rhai-cli-container-image)

[1.4. Run the migration assessment script](#1.4.-run-the-migration-assessment-script)

[1.4.1 Understand the migration assessment script output](#1.4.1-understand-the-migration-assessment-script-output)

[1.5 Submit the assessment output to Red Hat Technical Support](#1.5-submit-the-assessment-output-to-red-hat-technical-support)

[Chapter 2\. Before you upgrade](#chapter-2.-before-you-upgrade)

[2.1 Install the cert-manager Operator for Red Hat OpenShift](#2.1-install-the-cert-manager-operator-for-red-hat-openshift)

[2.2 Set Kueue management to Removed or Unmanaged](#2.2-set-kueue-management-to-removed)

[2.3. Model registry and catalog \- Before upgrade](#2.3.-model-registry-and-catalog---before-upgrade)

[2.4. Feature Store \- Before upgrade](#2.4.-feature-store---before-upgrade)

[2.5. Llama Stack / OGX (Open GenAI Stack) \- Before upgrade](#2.5.-llama-stack---before-upgrade)

[2.5.1. Llama Stack upgrade steps for cluster administrators](#2.5.1.-llama-stack-upgrade-steps-for-cluster-administrators)

[2.5.2. Llama Stack upgrade steps for LlamaStackDistribution resource owners](#2.5.2.-llama-stack-upgrade-steps-for-llamastackdistribution-resource-owners)

[2.6. AI Pipelines \- Before upgrade](#2.6.-ai-pipelines---before-upgrade)

[2.7. TrustyAI \- Before upgrade](#2.7.-trustyai---before-upgrade)

[2.7.1. TrustyAI \- Before upgrade \- Prepare for backup](#2.7.1.-trustyai---before-upgrade---prepare-for-backup)

[2.7.2. TrustyAI \- Before upgrade \- Backup metrics](#2.7.2.-trustyai---before-upgrade---backup-metrics)

[2.7.3. TrustyAI \- Before upgrade \- Backup data storage](#2.7.3.-trustyai---before-upgrade---backup-data-storage)

[2.7.4. TrustyAI \- Before upgrade \- Guardrails Orchestrator](#2.7.4.-trustyai---before-upgrade---guardrails-orchestrator)

[2.8. Workbenches \- Before upgrade](#2.8.-workbenches---before-upgrade)

[2.8.1. About upgrading your workbenches](#2.8.1.-about-upgrading-your-workbenches)

[2.8.2. Prepare your workbenches for migration](#2.8.2.-prepare-your-workbenches-for-migration)

[2.9. Ray Training Operator \- Before upgrade](#2.9.-ray-training-operator---before-upgrade)

[2.10. Model serving \- Before upgrade](#2.10.-model-serving---before-upgrade)

[2.10.1. Migration impact and scope](#2.10.1.-migration-impact-and-scope)

[2.10.2. Removed model serving configurations](#2.10.2.-removed-model-serving-configurations)

[2.10.3. Migration workflow for model serving](#2.10.3.-migration-workflow-for-model-serving)

[2.10.4. Prerequisites for model serving migration](#2.10.4.-prerequisites-for-model-serving-migration)

[2.10.5. Run the rhai-cli tool](#2.10.5.-run-the-rhai-cli-tool)

[2.10.6. Back up the inferenceservice-config ConfigMap](#2.10.6.-back-up-the-inferenceservice-config-configmap)

[2.10.7. Migrate InferenceServices to RawDeployment mode](#2.10.7.-migrate-inferenceservices-to-rawdeployment-mode)

[2.10.7.1 Convert Serverless InferenceServices to RawDeployment](#2.10.7.1-convert-serverless-inferenceservices-to-rawdeployment)

[2.10.7.2 Convert ModelMesh InferenceServices to RawDeployment](#2.10.7.2-convert-modelmesh-inferenceservices-to-rawdeployment)

[2.10.7.3 Verification of InferenceServices migration](#2.10.7.3-verification-of-inferenceservices-migration)

[2.10.8. Update the inferenceservice-config ConfigMap](#2.10.8.-update-the-inferenceservice-config-configmap)

[2.10.9. Update cluster configuration for migration](#2.10.9.-update-cluster-configuration-for-migration)

[2.10.10. Prepare distributed inference for migration](#2.10.10.-prepare-distributed-inference-for-migration)

[2.10.10.1. Install Red Hat Connectivity Link for distributed inference](#2.10.10.1.-install-red-hat-connectivity-link-for-distributed-inference)

[2.10.10.2. Configure Red Hat Connectivity Link for disconnected environments](#2.10.10.2.-configure-red-hat-connectivity-link-for-disconnected-environments)

[2.10.10.3. Configure authentication for LLMInferenceService resources](#2.10.10.3.-configure-authentication-for-llminferenceservice-resources)

[2.10.10.4. Freeze LLMInferenceService configuration for upgrade](#2.10.10.4.-freeze-llminferenceservice-configuration-for-upgrade)

[2.10.11. Verify migration readiness](#2.10.11.-verify-migration-readiness)

[2.11. Kubeflow Training Operator \- Before upgrade](#2.11.-kubeflow-training-operator---before-upgrade)

[2.12. OpenShift AI Operator \- Before upgrade](#2.12.-openshift-ai-operator---before-upgrade)

[Chapter 3\. Upgrade to 3.5](#chapter-3.-upgrade-to-3.5-latest)

[3.1. OpenShift AI Operator](#3.1.-openshift-ai-operator)

[Chapter 4\. After upgrading to 3.5](#chapter-4.-after-upgrading-to-3.5-latest)

[4.1. OpenShift AI Operator \- After upgrade](#4.1.-openshift-ai-operator---after-upgrade)

[4.2. AI hub registry and catalog \- After upgrade](#4.2.-ai-hub-registry-and-catalog---after-upgrade)

[4.3. Feature Store \- After upgrade](#4.3.-feature-store---after-upgrade)

[4.4. OGX (Open GenAI Stack, formerly Llama Stack) \- After upgrade](#4.4.-llama-stack---after-upgrade)

[4.5. AI Pipelines \- After upgrade](#4.5.-ai-pipelines---after-upgrade)

[4.5.1. Administrator tasks](#4.5.1.-administrator-tasks)

[4.5.2. Pipeline user tasks](#4.5.2.-pipeline-user-tasks)

[4.6. TrustyAI \- After upgrade](#4.6.-trustyai---after-upgrade)

[4.6.1. TrustyAI \- After upgrade \- Check Backups](#4.6.1.-trustyai---after-upgrade---check-backups)

[4.6.2. TrustyAI \- After upgrade \- Guardrails](#4.6.2.-trustyai---after-upgrade---guardrails)

[4.6.3. TrustyAI \- After upgrade \- Restore data](#4.6.3.-trustyai---after-upgrade---restore-data)

[4.6.4. TrustyAI \- After upgrade \- GPU deployment deadlock issue](#4.6.4.-trustyai---after-upgrade---gpu-deployment-deadlock-issue)

[4.7. Workbenches \- After upgrade](#4.7.-workbenches---after-upgrade)

[4.7.1. Migrate your workbenches after upgrade](#4.7.1.-migrate-your-workbenches-after-upgrade)

[4.7.2. Perform a deferred workbench image migration](#4.7.2.-perform-a-deferred-workbench-image-migration)

[4.8. Ray Training Operator \- After upgrade](#4.8.-ray-training-operator---after-upgrade)

[4.9. Model serving \- After upgrade](#4.9.-model-serving---after-upgrade)

[4.9.1. Finalize migration configuration](#4.9.1.-finalize-migration-configuration)

[4.9.2. Verifying upgrade completion and troubleshooting](#4.9.2.-verifying-upgrade-completion-and-troubleshooting)

[4.9.2.1. Verification steps](#4.9.2.1.-verification-steps)

[4.9.2.2.1. Serverless InferenceServices not converted before upgrade](#4.9.2.2.1.-serverless-inferenceservices-not-converted-before-upgrade)

[4.9.2.2.2. ModelMesh InferenceServices not converted before upgrade](#4.9.2.2.2.-modelmesh-inferenceservices-not-converted-before-upgrade)

[4.9.2.2.4. Authorino Operator not removed](#4.9.2.2.4.-authorino-operator-not-removed)

[4.9.2.2.5. Service Mesh v2 Operator not removed](#4.9.2.2.5.-service-mesh-v2-operator-not-removed)

[4.9.2.3. Additional resources](#4.9.2.3.-additional-resources)

[4.10. Kubeflow Training Operator \- After upgrade](#4.10.-kubeflow-training-operator---after-upgrade)

[5\. Clean up](#5.-clean-up)

[Legal Notice](#legal-notice)

[**Legal Notice**](#legal-notice)

# **Preface** {#preface}

Migrate from Red Hat OpenShift AI 2.25.9 (and later) to OpenShift AI 3.5.

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

  The container image is available at quay.io/rhoai/odh-cli-rhel9@sha256:edc0ebe9ffeac42b9dd4d34ed3d11753b40fc33a2c4824e15aef059a536241b0.

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
             image: quay.io/rhoai/odh-cli-rhel9@sha256:edc0ebe9ffeac42b9dd4d34ed3d11753b40fc33a2c4824e15aef059a536241b0
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

The container image is available at **quay.io/rhoai/odh-cli-rhel9@sha256:edc0ebe9ffeac42b9dd4d34ed3d11753b40fc33a2c4824e15aef059a536241b0**. It contains the migration assessment linting CLI and migration actions for specific component migrations.

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
| 0 | All checks passed with no findings above the info level. |
| 1 | One or more prohibited or critical findings were detected. This is expected behavior, not a tool error. |
| 2 | No prohibited or critical findings were detected, but one or more warning findings were reported. This is expected behavior, not a tool error. |

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

# 

# Chapter 2\. Before you upgrade {#chapter-2.-before-you-upgrade}

Before you upgrade OpenShift AI from version 2.25.9 (and later) to 3.5, you must migrate workload data for each of the components installed on your OpenShift AI cluster.

The **rhai-cli** utility contains migration actions to assist with pre-upgrade and post-upgrade steps for the Model Serving, Workbenches, TrustyAI, Llama Stack / OGX, AI Pipelines, and Ray Training Operator components.

To run the **rhai-cli migrate** actions, you must have write access as a cluster or namespace administrator. These actions require write access because they perform modifications to cluster resources.

## **2.1 Install the cert-manager Operator for Red Hat OpenShift** {#2.1-install-the-cert-manager-operator-for-red-hat-openshift}

**Procedure**

Install the cert-manager Operator for Red Hat OpenShift using one of the following methods:

**Method 1: Using the OpenShift web console**

1. In the OpenShift console, select **Operators** → **Operator Hub**.

2. For the **Projects** field, make sure that **All projects** are selected.

3. Search for **cert-manager Operator for Red Hat OpenShift**.

4. If the tile for the **cert-manager Operator provided by Red Hat** does not have an **Installed** label, install the cert-manager Operator and wait for it to be ready.

**Method 2: Using the OpenShift CLI**

1. Create the namespace, OperatorGroup, and Subscription:

   ```bash
   oc create namespace cert-manager-operator
   oc apply -f - <<EOF
   apiVersion: operators.coreos.com/v1
   kind: OperatorGroup
   metadata:
     name: cert-manager-operator
     namespace: cert-manager-operator
   spec:
     targetNamespaces:
     - cert-manager-operator
     upgradeStrategy: Default
   EOF
   oc apply -f - <<EOF
   apiVersion: operators.coreos.com/v1alpha1
   kind: Subscription
   metadata:
     name: openshift-cert-manager-operator
     namespace: cert-manager-operator
   spec:
     channel: stable-v1
     installPlanApproval: Automatic
     name: openshift-cert-manager-operator
     source: redhat-operators
     sourceNamespace: openshift-marketplace
   EOF
   ```

2. Wait for the cert-manager Operator to reach **Succeeded** status:

   ```bash
   oc get csv -n cert-manager-operator --watch
   ```

3. Verify that cert-manager pods are running:

   ```bash
   oc get pods -n cert-manager
   ```

   All pods should show **Running** status with all containers ready.

##  **2.2 Set Kueue management to Removed or Unmanaged** {#2.2-set-kueue-management-to-removed}

The migration assessment script checks your OpenShift AI 2.25.9 (and later) installation to determine the status of the Kueue component management. In OpenShift AI 3.5, the supported Kueue management states are **Removed** and **Unmanaged**. The **Managed** state is accepted by OLM for backwards compatibility but is rejected at runtime.

You must set the Kueue management state to either **Removed** or **Unmanaged** before upgrading:

* **Removed**: Disables Kueue entirely. Use this if you do not need Kueue workload scheduling.

* **Unmanaged**: Integrates with an externally installed Red Hat build of Kueue (RHBOK) Operator. Use this if you want to continue using Kueue for workload scheduling. Before upgrading, ensure the RHBOK Operator is installed and operational. You can use the `rhai-cli` RHBOK migration action to automate this migration.

If the current state is **Managed**, you must migrate to either **Removed** or **Unmanaged** before upgrading.

**Procedure to set Kueue to Removed**

1. In the OpenShift console, select **Ecosystem** \> **Installed Operators**.

2. Click the **Red Hat OpenShift AI Operator**.

3. Click the **Data Science Cluster** tab.

4. Click the default instance name (for example, **default-dsc**) to open the instance details page.

5. Click the **YAML** tab.

6. Edit the spec:components section. For the kueue component, set the managementState field to **Removed**:

   `spec:`

     `components:`

       `kueue:`

         `managementState: Removed`

7. Click **Save**.

**Procedure to set Kueue to Unmanaged (with external RHBOK)**

If you want to continue using Kueue with an externally managed Red Hat build of Kueue Operator, follow the detailed migration steps in the "Kueue - Before upgrade" section below.

**Verification**

1. Run the migration assessment script as described in [Run the migration assessment script](#1.4.-run-the-migration-assessment-script).  
2. Verify that Kueue has no critical findings. Kueue data-integrity findings are reported as advisory warnings — review these and resolve any inconsistencies before proceeding.
3. If migrating to Unmanaged (RHBOK), verify that the assessment confirms the Red Hat build of Kueue Operator is installed.
4. If the assessment warns about a LocalQueue named `default`, rename your default queues to custom names (for example, `rhoai-kueue-default`) before proceeding. See the queue naming guidance in step 4 below.

***Kueue \- Before upgrade***

*Before starting the upgrade to OpenShift AI 3.5, if you are currently using embedded Kueue (managementState: Managed), you must migrate to either Red Hat build of Kueue (managementState: Unmanaged) or disable Kueue entirely (managementState: Removed). Embedded Kueue is not supported in OpenShift AI 3.5 -- the Managed state is accepted by OLM for backwards compatibility but is rejected at runtime.*

*Important*  
*You must perform the migration to Red Hat build of Kueue independently and in advance of the OpenShift AI version upgrade. Completing this migration before upgrade will simplify planning and reduce the overall upgrade time.*

*The upgrade to OpenShift AI 3.5 assumes that embedded Kueue has already been migrated to Red Hat build of Kueue and does not include or perform this migration automatically. Ensure that the migration to Red Hat build of Kueue is fully completed and validated before proceeding with the upgrade to OpenShift AI 3.5.*

*Prerequisites*

* *You have cluster administrator access to the OpenShift cluster.*  
* *The DataScienceCluster resource has the Kueue component in a Ready state.*

*Procedure*

1. *Check that the Kueue component is in a ready state as follows:*

   ```bash
   read STATUS REASON < <(oc get datasciencecluster -A -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status} {.items[0].status.conditions[?(@.type=="KueueReady")].reason}'); [[ "$STATUS" == "True" || ( "$STATUS" == "False" && "$REASON" == "Removed" ) ]] && echo "Ready" || echo "Not Ready"
   ```

   *The command output must be Ready.*

2. *Check that Kueue has been migrated to Red Hat build of Kueue as follows:*  
   ```bash
   oc get datasciencecluster -A -o jsonpath='{.items[0].spec.components.kueue.managementState}{"\n"}'
   ```

   * *If the output is Removed or Unmanaged, no migration is required and you can skip the remaining steps. Confirm using the rhai-cli script that kueue reports PASS.*

   * *If the output is Managed, you must migrate to Red Hat build of Kueue.*

3. *If you are using the default OpenShift AI Kueue configuration and have not modified the kueue-manager-config config map in your applications namespace, annotate the config map to preserve the enabled frameworks as follows:*

   ```bash
   oc annotate configmap kueue-manager-config -n <applications_namespace> opendatahub.io/managed=false
   ```

   *\<applications\_namespace\> specifies the namespace where your Kueue applications are deployed. The default is redhat-ods-applications.*

   *Important*

   *This annotation ensures that your enabled frameworks remain unchanged during migration. Without this annotation, the enabled frameworks will change.*

         *Before migration, the enabled frameworks are as follows:*

   * *batch/job*  
   * *kubeflow.org/mpijob*  
   * *ray.io/rayjob*  
   * *ray.io/raycluster*  
   * *jobset.x-k8s.io/jobset*  
   * *kubeflow.org/paddlejob*  
   * *kubeflow.org/pytorchjob*  
   * *kubeflow.org/tfjob*  
   * *kubeflow.org/xgboostjob*  
   * *workload.codeflare.dev/appwrapper*

   *After migration, without this annotation, the enabled frameworks will change as follows:*

* *Deployment*  
* *Pod*  
* *PyTorchJob*  
* *RayCluster*  
* *RayJob*  
* *StatefulSet*

4. *Perform the steps in [Migrating to the Red Hat build of Kueue Operator](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.25/html/managing_openshift_ai/managing-workloads-with-kueue#migrating-to-the-rhbok-operator_kueue).*

   *Important*  
   *Do not follow the Next steps section in the Operator migration guide. Return to this procedure after completing the Operator migration steps.*

   *Note*  
   *When you activate the Red Hat build of Kueue Operator in the DataScienceCluster resource, define custom names for your local queues and cluster queues instead of using the name `default`. The Red Hat build of Kueue treats the name `default` as a system identifier for enabled frameworks. When Kueue transitions to Unmanaged, workloads in kueue-managed namespaces that are not explicitly assigned to a queue are implicitly placed on the LocalQueue named `default`. If that name is already in use for other purposes, workloads can be scheduled to unintended queues.*

   *Using custom queue names (for example, `rhoai-kueue-default` or `my-project-default`) avoids potential conflicts and ensures correct resource allocation across frameworks.*

   *To use predefined queue names, apply the following configuration:*

   ```yaml
   spec:
     components:
       kueue:
         managementState: Unmanaged
   ```

   *To specify custom queue names (recommended), apply the following configuration:*

   ```yaml
   spec:
     components:
       kueue:
         managementState: Unmanaged
         defaultClusterQueueName: <example-cluster-queue>
         defaultLocalQueueName: <example-local-queue>
   ```

5. *Enable Kueue management for existing projects using Kueue by applying the `kueue.openshift.io/managed=true` label to each project namespace:*

   ```bash
   oc label namespace <project-namespace> kueue.openshift.io/managed=true --overwrite
   ```

   *Warning*  
   *Kueue validation and queue enforcement apply only to workloads in namespaces labeled with `kueue.openshift.io/managed=true`.*

   *After you apply this label, ensure that the following resources within that namespace have the `kueue.x-k8s.io/queue-name` annotation set to the relevant Kueue queue name:*

   * *pytorchjob*
   * *notebook*
   * *rayjob*
   * *raycluster*
   * *inferenceservice*
   * *llminferenceservice*

   *If this annotation is missing in a managed namespace, any subsequent modification or creation of these resources will be rejected by the admission controller.*

*Verification*

* *Verify that the migration to Red Hat build of Kueue was successful as follows:*

  ```bash
  oc get datasciencecluster -A -o jsonpath='{.items[0].spec.components.kueue.managementState}{"\n"}{.items[0].status.conditions[?(@.type=="KueueReady")].status}{"\n"}'
  ```

  *The command output must be the following:*  
  *Unmanaged*  
      *True*

## 

## **2.3. Model registry and catalog \- Before upgrade** {#2.3.-model-registry-and-catalog---before-upgrade}

If any model registries or custom model catalog sources were created before upgrade, you must ensure that these are configured correctly and functional in the OpenShift AI dashboard.

**Prerequisites**

* You have OpenShift AI administrator access to manage model registries.

* You have the OpenShift **oc** command line interface installed.

* You have the appropriate permissions in OpenShift to access the required resources.

**Procedure**

1. In the OpenShift web console, click **Workloads \> Pods**.

2. In the **Project** field, enter **rhoai-model-registries** and check that all the pods have a status of **Running**.

3. You can also get information on pods by using the following command:  
   ```bash
   oc get pods -n rhoai-model-registries
   ```

   Check the pod logs to ensure there are no error messages as follows:  
   ```bash
   oc logs <my-model-catalog-pod-name> -n rhoai-model-registries -c catalog
   ```

4. In the OpenShift AI dashboard, click **Settings \> Model registry settings** to check the status of your model registries. For more information, see [Managing model registries](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.25/html/managing_model_registries/index).

5. Click **Models \> Model catalog** to check the default catalog and any custom catalogs that were created. For more information, see [Working with the model catalog](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/2.25/html/working_with_the_model_catalog).

**Verification**

1. In OpenShift, there are no errors in the following pods:

   * **\<my-model-registry\>-xxx**

   * **db-\<my-model-registry\>-xxx**

   * **model-catalog-xxx**

2. In the OpenShift AI dashboard, on the **Settings \> Model registry settings** page, your model registries are displayed with **Available** status.

3. On the **Models \> model registry** page, your model registries are displayed correctly.

4. On the **Models \> model catalog** page, your custom models are displayed correctly.

**Important**  
During upgrade, the model catalog or model registry might become inaccessible in the dashboard while the pods are respinned. When the model catalog, model registry, and dashboard pods all have a status of **Running**, the model catalog and model registry should be available again in the dashboard.

After upgrade, the dashboard navigation will change to **AI hub \> Registry** and **Catalog**. For more information, see Section 4.2, “AI hub registry and catalog \- After upgrade”.

## 

## **2.4. Feature Store \- Before upgrade** {#2.4.-feature-store---before-upgrade}

In Red Hat OpenShift AI 2.25.9 (and later) , the Feature Store component is a Technology Preview feature. In OpenShift AI 3.5 , it is a GA feature. Otherwise, the Feature Store component is unchanged between Red Hat OpenShift AI 2.25.9 (and later) and 3.5.

If you use the Feature Store component in OpenShift AI 2.25.9 (and later) , follow the steps in this procedure to verify that Feature Store is working correctly before you upgrade to OpenShift AI 3.5.

**Note**  
If you do not use the Feature Store component in OpenShift AI 2.25.9 (and later) , skip this section. You do not need to perform any steps for Feature Store before you upgrade to 3.5.

**Prerequisites**

* You created a Feature Store Custom Resource (CR) in OpenShift AI 2.25.9 (and later) .

* You have OpenShift AI administrator access for the procedure steps and OpenShift AI user access for the verification steps.

**Procedure**

1. As an OpenShift AI administrator, get a list of all Feature Store instances and their namespaces on the cluster:  
   ```bash
   oc get featurestores --all-namespaces
   ```

   Example output:

   ```
   NAMESPACE        NAME              AGE
   project-alpha    my-featurestore   5d
   project-beta     demo-store        2d
   project-gamma    prod-store        10d
   ```

2. As an OpenShift AI administrator, follow these steps for each Feature Store instance:  
   1. Check that the Feature Store instance is in the **Ready** state. Get the status of a Feature Store instance by running the following command and replacing **\<namespace\>** with the namespace that has the Feature Store instance:

      ```bash
      oc get featurestores -n <namespace>
      ```

      For example, given the example output from Step 1, to see the status of **my-featurestore**, run the following command:

      ```bash
      oc get featurestores -n project-alpha
      ```

      Example output:

      ```
      NAME                    STATUS   AGE
      my-featurestore         Ready    5d
      ```

   2. List CronJobs for a namespace that has a Feature Store instance by running the following command and replacing **\<namespace\>** with the name of the namespace:

      ```bash
      oc get cronjobs -n <namespace>
      ```

      For example:

      ```bash
      oc get cronjobs -n project-alpha
      ```

      Example output:

      ```
      NAME              SCHEDULE   TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
      feast-sample-git  @yearly    <none>     True      0        <none>          74m
      ```

   3. Create a Job by running the following command. Replace **\<job-name\>** with the name of the job and replace **\<cronjob-name\>** with the name of a CronJob output from the previous step:

      ```bash
      oc create job <job-name> --from=cronjob/<cronjob-name> -n <namespace>
      ```

      For example:

      ```bash
      oc create job test --from=cronjob/feast-sample-git -n project-alpha
      ```

      Example output:

      ```
      job.batch/test created
      ```

   4. Check that the CronJob for the Feature Store instance ran the Job successfully. View a list of jobs and their status by running the following command:

      ```bash
      oc get jobs -n <namespace>
      ```

      For example:

      ```bash
      oc get jobs -n project-alpha
      ```

      The output should indicate that the job completed, as shown in the following example:

      ```
      NAME    STATUS     COMPLETIONS   DURATION   AGE
      test    Complete   1/1           40s        5m53s
      ```

**Verification**

As an OpenShift AI user, for each Feature Store instance:

1. In the OpenShift AI dashboard navigation bar, click **Feature Store**.

2. Verify that the Feature Store UI shows any features, entities, feature-views, data sources, and feature services that you previously configured in your feature store.

##  **2.5. Llama Stack / OGX (Open GenAI Stack) \- Before upgrade** {#2.5.-llama-stack---before-upgrade}

**IMPORTANT** 

 If you are using a disconnected environment, you can skip this section.  
Support for Llama Stack in a disconnected environment is provided starting in OpenShift AI 3.0.

**Note**  
In OpenShift AI 3.5, Llama Stack has been renamed to **OGX (Open GenAI Stack)**. The **LlamaStackDistribution** custom resource (CR) is replaced by the **OGXServer** (v1beta1) custom resource. All references to "Llama Stack" in this section refer to the component as it exists in OpenShift AI 2.25.9 (and later). After upgrading to 3.5, you will work with OGX resources instead.

Upgrading from OpenShift AI version 2.25.9 (and later) to 3.5 requires recreating your Llama Stack deployments as **OGXServer** custom resources. The Llama Stack component was in Technology Preview in 2.25.x and the architecture changes between OpenShift AI versions 2.25.9 (and later) and 3.5 — including the rename to OGX — are incompatible with direct upgrades or data migrations.

Cluster administrators and **LlamaStackDistribution** owners perform different upgrade steps:

* **For cluster administrators** \- You need to identify all existing **LlamaStackDistribution** CRs resources and inform owners that they must archive and update their Llama Stack–based projects. The owners must then delete existing **LlamaStackDistribution** CRs before upgrade and recreate them as **OGXServer** CRs in version 3.5.

* For LlamaStackDistribution resource owners \- If necessary, archive your Llama Stack state by running an archive script, delete the old LlamaStackDistribution CRs, and after upgrading to OpenShift AI 3.5, recreate them as **OGXServer** CRs. Update your Llama Stack–based projects as needed to ensure compatibility with the OGX APIs in OpenShift AI 3.5.

**Warning**

Since Llama Stack was in Technology Preview in 2.25.x and has been renamed to OGX in 3.5, this upgrade results in complete loss of existing Llama Stack data including:

* Agent state and configurations

* Vector database metadata and embeddings

* Telemetry data

* File metadata

* All resources stored in SQLite

###  **2.5.1. Llama Stack upgrade steps for cluster administrators** {#2.5.1.-llama-stack-upgrade-steps-for-cluster-administrators}

**Prerequisites**

* You have OpenShift AI cluster administrator access.

* You have existing **LlamaStackDistribution** CRs deployed with the Llama Stack Operator.

**Procedure**

1. Before proceeding with the upgrade, OpenShift AI cluster administrators must identify all **LlamaStackDistribution** CRs resources in your cluster. List all **LlamaStackDistributions** resources across your cluster with the following command:

   ```bash
   oc get llamastackdistribution --all-namespaces -o custom-columns=NAMESPACE:.metadata.namespace,NAME:.metadata.name,PHASE:.status.phase,Created:.metadata.creationTimestamp
   ```

2. For each namespace with Llama Stack deployments, identify the owners of the namespace with the following command:

   ```bash
   oc get rolebindings -n <namespace> -o wide
   ```

3. Once you have identified all **LlamaStackDistribution** CRs and their owners, contact each owner and inform them of the following:

   * Inform each **LlamaStackDistribution** resource owner of the upgrade and the requirement to recreate their deployments as **OGXServer** CRs in OpenShift AI 3.5.

   * Ensure they archive the Llama Stack data using the backup script.

   * Schedule a maintenance window for the upgrade.

   * Provide them with the OpenShift AI 3.5 OGX documentation for recreating deployments. See [Working with Llama Stack](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/working_with_llama_stack/index).

### **2.5.2. Llama Stack upgrade steps for LlamaStackDistribution resource owners** {#2.5.2.-llama-stack-upgrade-steps-for-llamastackdistribution-resource-owners}

If you are a **LlamaStackDistribution** resource owner in OpenShift AI 2.25.9 (and later), you must prepare to delete the **LlamaStackDistribution** resources before upgrade and recreate them as **OGXServer** CRs after upgrade. To prepare, you must archive Llama Stack deployment data that can be used as reference when recreating your deployment.

**Prerequisites**

* You were contacted by your cluster administrator about upgrading Llama Stack deployments

**Procedure**

1. (Optional) To archive your LlamaStack configuration and data, use the **rhai-cli** `llamastack.backup` migration action. The `llamastack.backup` migration is a prepare-only action. You can preview what would be backed up with `--dry-run`, then run the backup:

   ```bash
   rhai-cli migrate prepare --migration llamastack.backup --target-version 3.5.0 --dry-run
   rhai-cli migrate prepare --migration llamastack.backup --target-version 3.5.0 --output-dir /backups
   ```

   Expected Output:

   ```bash
   llamastack.backup:

     → Discover LlamaStack resources
       ✓ Found 1 LlamaStack resource(s)
     → Backup LlamaStack \<resource-name\> (\<namespace\>)
       ✓ Saved \<namespace\>/\<resource-name\>/llamastackdistributions.llamastack.io-\<resource-name\>.yaml

   Preparation llamastack.backup completed successfully\!
   ```

   Next steps:

   1\. Review the archived configurations in the backup directory

   2\. Use archives as reference when creating your new OGXServer CRs in RHOAI 3.5

   3\. Update your client applications to use the new OGX APIs

   **Note**  
   Due to the PostgreSQL database requirement in OpenShift AI 3.5, the SQLite specific databases and configurations archived by this action cannot be directly imported to OpenShift AI 3.5

2. After your cluster administrator notifies you of the upgrade, and if you optionally archived your Llama Stack data, at the start of the maintenance window, delete your **LlamaStackDistribution** resources.

   ```bash
   oc get -A llsd
   ```

   Example output:

   ```bash
   NAMESPACE             NAME                                 PHASE   OPERATOR VERSION   SERVER VERSION   AVAILABLE   AGE

   ldap-user17-rag-225   lsd-llama-milvus-2323                Ready   "0.3.0"            0.2.22.2+rhai0   1           11d

   neha                  lsd-llama-milvus                     Ready   "0.3.0"            0.2.22.2+rhai0   1           11d
   ```

   For each llsd:

   ```bash
   oc delete -n <namespace> llsd/<llsd-resource-name>
   ```

3. During the recreation of deployments as **OGXServer** resources in OpenShift AI 3.5, you must complete and understand the following:

   1. Port the client application code and notebooks to be compatible with OGX (Open GenAI Stack) in OpenShift AI 3.5.

   2. Applications built for OpenShift AI 2.25.9 (and later) will require updates to work with OpenShift AI 3.5 due to API changes. These API changes include:

      * VectorDB API removed: Replace calls to the deprecated **VectorDB** API with the new **Vector\_IO** API.

      * Inference API changes: Update Completions API calls to match the OpenAI-compatible format.

      * Embedding API changes: Now uses new external embedding model endpoint.

      * Agent API changes: Review and update agent creation and interaction code for API compatibility.

   3. The usage of the **llama-stack-client** library is distinct in OpenShift AI 3.5, refer to [OpenAI-compatible APIs in Llama Stack](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/working_with_llama_stack/overview-of-llama-stack_rag#openai-compatible-apis-in-Llama-Stack_rag).

   4. It is recommended to test your new OGX deployments in an OpenShift AI 3.5 staging cluster. Testing in the staging cluster allows you to do the following:

      * Validate new **OGXServer** configurations

      * Update application code for API compatibility

      * Identify missing dependencies

      * Prepare migration scripts as well as documentation.

## **2.6. AI Pipelines \- Before upgrade** {#2.6.-ai-pipelines---before-upgrade}

Upgrading to OpenShift AI 3.5 does not introduce major functional changes to AI Pipelines, and existing pipelines are expected to continue running without modification.

However, the upgrade includes updates to API versions and RBAC permissions. Before upgrading, run the migration assessment and the **rhai-cli** AI Pipelines migration actions to identify and remediate deprecated resources or custom RBAC roles.

**Prerequisites**

* The DataSciencePipelines application is installed on the cluster.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

1. Capture the pre-upgrade DSPA pod health baseline:

   ```bash
   rhai-cli migrate prepare --migration ai-pipelines.pre-upgrade-check --target-version 3.5.0
   ```

   To preview without making changes, add `--dry-run`.

   This saves a snapshot of DSPA pod health to `/tmp/rhoai-upgrade-backup/ai_pipelines/dspa_pre_upgrade_pods.json`. The post-upgrade check in [AI Pipelines - After upgrade](#4.5.-ai-pipelines---after-upgrade) compares against this baseline.

   **Important**

   This state file must persist across the upgrade. It is stored on the PVC mounted at `/tmp/rhoai-upgrade-backup` in the **rhai-cli** pod. Ensure that you do not delete the PVC or the **rhai-cli** StatefulSet before the post-upgrade check completes.

2. Run the AI Pipelines pre-upgrade check to remediate deprecated resources:

   ```bash
   rhai-cli migrate run --migration ai-pipelines.pre-upgrade-check --target-version 3.5.0
   ```

3. Review the output.  
   The action might report:

   * Deprecated **DataSciencePipelinesApplication** resources that use the **v1alpha1** API.

   * Custom RBAC roles that require updates.

4. If the action reports issues, follow the remediation guidance provided. 

   If the action indicates that it did not find any issues, skip the remaining steps in this procedure and continue to the next section. 

5. If the action reports custom RBAC roles that require updates, consult with the teams that use AI Pipelines and run the DSP role update action:

   ```bash
   rhai-cli migrate run --migration ai-pipelines.update-dsp-role --target-version 3.5.0
   ```

6. After completing remediation, rerun the pre-upgrade check to confirm that no issues remain:

   ```bash
   rhai-cli migrate run --migration ai-pipelines.pre-upgrade-check --target-version 3.5.0
   ```

   You can ignore the following warning if it appears, because it does not affect the upgrade:

   ```bash
   Found DataSciencePipelinesApplication(s) with deprecated '.spec.apiServer.managedPipelines.instructLab' field
   ```

**Verification**

* The `ai-pipelines.pre-upgrade-check` action reports no remaining issues.

* Any required RBAC updates have been applied.

* The pre-upgrade state file exists:

  ```bash
  ls -la /tmp/rhoai-upgrade-backup/ai_pipelines/dspa_pre_upgrade_pods.json
  ```

## 

## **2.7. TrustyAI \- Before upgrade** {#2.7.-trustyai---before-upgrade}

To check whether you must complete upgrade tasks for TrustyAI, confirm that the management state of the TrustyAI component for your Data Science Cluster is Managed. You can then backup TrustyAI metrics and data storage, and migrate TrustyAI Guardrails Orchestrator services.

**Warning**

Complete all steps in the following procedures in the order presented. Do not proceed to the next step until the output matches the expected result.

* TrustyAI \- Before upgrade \- Prepare for backup

* TrustyAI \- Before upgrade \- Backup metrics

* TrustyAI \- Before upgrade \- Backup data storage

* TrustyAI \- Before upgrade \- Guardrails Orchestrator

### **2.7.1. TrustyAI \- Before upgrade \- Prepare for backup** {#2.7.1.-trustyai---before-upgrade---prepare-for-backup}

Verify that the management state of the TrustyAI component for your Data Science Cluster is **Managed** and create a directory for backups.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

1. Check the **TrustyAI** management state for your Data Science Cluster:

   ```bash
   export DSC_NAME=$(oc get datascienceclusters -o jsonpath='{.items[0].metadata.name}')
   oc get datascienceclusters "$DSC_NAME" -o jsonpath='{.spec.components.trustyai.managementState}' && echo
   ```

   Example output:

   ```
   Managed
   ```

   If the output is **Managed**, continue to the next step.

   If the output is empty, **Removed**, or **Unmanaged**, you do not need to perform any steps for the TrustyAI component before you upgrade to OpenShift AI 3.5.

2. Get a list of the namespaces that contain a TrustyAI service:

   ```bash
   oc get trustyaiservice -A
   ```

   Example output:

   ```
   NAMESPACE                 NAME               AGE
   test-trustyaiservice      trustyai-service   4h15m
   ```

   **Note**  
   If the output is **No resources found**, there are no metrics or storage data to backup. Skip to TrustyAI \- Before upgrade \- Guardrails Orchestrator.

3. Create a directory for backups:

   ```bash
   mkdir -p /tmp/rhoai-upgrade-backup/trustyai
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
   ```

### **2.7.2. TrustyAI \- Before upgrade \- Backup metrics** {#2.7.2.-trustyai---before-upgrade---backup-metrics}

You can backup scheduled TrustyAI metrics before you upgrade OpenShift AI 2.25.9 (and later) to 3.5.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You completed the steps in TrustyAI \- Before upgrade \- Prepare for backup.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

For each namespace that has a TrustyAI service, follow these steps to backup scheduled TrustyAI metrics:

1. Set the namespace:

   ```bash
   export NS=<namespace>
   ```

2. Set the **TrustyAIService** name:

   ```bash
   export TAS_NAME=$(oc get trustyaiservice -n "$NS" -o jsonpath='{.items[0].metadata.name}')
   ```

3. Confirm the namespace and **TrustyAIService** names:

   ```bash
   echo "NAMESPACE=${NS} TAS_NAME=${TAS_NAME}"
   ```

   Example output:

   ```
   NAMESPACE=test-trustyaiservice TAS_NAME=trustyai-service
   ```

4. Get the service HTTP port:

   ```bash
   export SVC_PORT=$(oc get svc -n "$NS" "$TAS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
   echo "SVC_PORT=$SVC_PORT"
   ```

   If the result is a port number as shown in the following example, skip to Step 7:

   ```
   SVC_PORT=80
   ```

   If the result is an empty value, continue to the next step to set an HTTP port number for the service.

5. List all available ports:

   ```bash
   oc get svc -n "$NS" "$TAS_NAME" -o jsonpath='{range .spec.ports[*]}{.name}:{.port}{"\n"}{end}'
   ```

   Example output:

   ```
   http:80
   https:443
   ```

6. In the following command, replace \<port\> with the port number (for example, **80**):

   ```bash
   export SVC_PORT=<port>
   ```

7. Port-forward the TrustyAI service:

   ```bash
   oc port-forward -n "$NS" "svc/$TAS_NAME" 8080:${SVC_PORT} &
   export PF_PID=$!
   sleep 3
   ```

   Example output:

   ```
   Forwarding from 127.0.0.1:8080 -> 8080
   Forwarding from [::1]:8080 -> 8080
   ```

8. Fetch the metrics:

   ```bash
   export TOKEN=$(oc whoami -t)
   curl -sk -H "Authorization: Bearer $TOKEN" \
     "http://localhost:8080/metrics/all/requests" \
     -o "${BACKUP_DIR}/trustyai-metrics-${NS}-$(date +%Y%m%d-%H%M%S).json"
   ```

9. Stop the port-forward:

   ```bash
   kill $PF_PID 2>/dev/null
   ```

**Verification**

1. Verify that the port-forward process stopped running:

   ```bash
   ps -p $PF_PID 2>/dev/null && echo "Still running" || echo "Stopped"
   ```

   Example output:

   ```
   Stopped
   ```

2. Validate that the backup is not empty:

   **Note**
   Run this command from your workstation (not from inside the **rhai-cli** pod). It reads the backup file from the pod's PVC and validates it with `jq` locally.

   ```bash
   oc exec rhai-cli-0 -n rhai-migration -- cat ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | jq empty && echo "OK" || echo "FAIL: invalid JSON"
   ```

   **Note**
   Replace `rhai-migration` with the namespace where your **rhai-cli** StatefulSet is deployed, if different.

   Example output:

   ```
   OK
   ```

3. Verify that the backup file exists:

   ```bash
   oc exec rhai-cli-0 -n rhai-migration -- ls ${BACKUP_DIR}/trustyai-metrics-${NS}-*
   ```

   Example output:

   ```
   /tmp/rhoai-upgrade-backup/trustyai/trustyai-metrics-test-trustyaiservice-upgrade-20260218-175450.json
   ```

### **2.7.3. TrustyAI \- Before upgrade \- Backup data storage** {#2.7.3.-trustyai---before-upgrade---backup-data-storage}

You can backup TrustyAI data storage before you upgrade OpenShift AI 2.25.9 (and later) to 3.5. The **rhai-cli** `trustyai.data` migration action auto-detects storage type (PVC or DATABASE) from the TrustyAIService CR and performs the appropriate backup.

The backup for each namespace is a self-contained directory with a **metadata.json** file and the data.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You completed the steps in TrustyAI \- Before upgrade \- Prepare for backup.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

For each namespace that has a TrustyAI service, follow these steps to backup TrustyAI data storage:

1. If the namespace is not already set, set it:

   ```bash
   export NS=<namespace>
   ```

2. Run the TrustyAI data backup action:

   ```bash
   rhai-cli migrate prepare --migration trustyai.data --target-version 3.5.0 \
       --output-dir /tmp/rhoai-upgrade-backup/trustyai
   ```

   To preview without making changes, add `--dry-run`.

   **Important**
   The `--output-dir` flag is required when running inside the **rhai-cli** pod. Without it, the action attempts to create a backup directory in the container's root filesystem, which is read-only, and fails with a `permission denied` error.

   Example output for PVC storage:

   ```
   trustyai.data:

     → Discover TrustyAIService storage backends
       ✓ Found 1 PVC-backed instance
     → Backup PVC data for trustyai-service (test-trustyaiservice-upgrade)
       ✓ Backed up PVC trustyai-pvc

   Preparation trustyai.data completed successfully!
   ```

   Example output for DATABASE storage:

   ```
   trustyai.data:

     → Discover TrustyAIService storage backends
       ✓ Found 1 MariaDB-backed instance
     → Backup MariaDB for trustyai-service (test-trustyaiservice-upgrade)
       ✓ Dumped database trustyai_db

   Preparation trustyai.data completed successfully!
   ```

**Verification**

The action ends with **Preparation trustyai.data completed successfully\!**.

If the action fails, it provides error messages. Common issues are: PVC not bound, MariaDB pod not running, or missing credentials secret.

### **2.7.4. TrustyAI \- Before upgrade \- Guardrails Orchestrator** {#2.7.4.-trustyai---before-upgrade---guardrails-orchestrator}

Before you upgrade OpenShift AI 2.25.9 (and later) to 3.5, set the name of all TrustyAI Guardrails Orchestrator services, validate the custom resource, and check for OpenTelemetry exporters.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You completed the steps in TrustyAI \- Before upgrade \- Prepare for backup.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

1. Get a list of the namespaces that contain the TrustyAI Guardrails Orchestrator service:

   ```bash
   oc get guardrailsorchestrator -A
   ```

   Example output:

   ```
   NAMESPACE                         NAME                      AGE
   redhat-ods-operator               guardrails-orchestrator   4h22m
   test-guardrails-builtin-upgrade   guardrails-orchestrator   4h50m
   test-guardrails-tempo-hf          guardrails-orchestrator   4h17m
   ```

   **Note**  
   If the output is **No resources found**, you can skip the remaining steps in this procedure.

1. For each namespace that has the TrustyAI Guardrails Orchestrator service:

1. Set the namespace:

   ```bash
   export NS=<namespace>
   ```

2. Set the **GuardrailsOrchestrator** name:

   ```bash
   export GORCH_NAME=<orchestrator-name>
   ```

3.  Check that **GORCH\_NAME** and **NS** environment variables are properly set:

   ```bash
   echo "GORCH_NAME=$GORCH_NAME" && echo "NS=$NS"
   ```

   The output should return values for **GORCH\_NAME** and **NS**, as shown in the following example:

   ```
   GORCH_NAME=guardrails-orchestrator
   NS=test-guardrails-tempo-hf
   ```

**Verification**

For each namespace that has the TrustyAI Guardrails Orchestrator service:

1. Validate your **GuardrailsOrchestrator** custom resource:

1. Find the **GuardrailsOrchestrator** pod:

   ```bash
   export ORCH_POD=$(oc get pods -n $NS --no-headers -o name | grep $GORCH_NAME | head -1)
   echo "ORCH_POD=$ORCH_POD"
   ```

   The output should show a value for **ORCH\_POD**, as shown in the following example:

   ```
   ORCH_POD=pod/guardrails-orchestrator-696fdbfbb4-zlx74
   ```

2. Check the pod phase and readiness:

   ```bash
   oc get "$ORCH_POD" -n "$NS" -o jsonpath='phase={.status.phase} ready={.status.conditions[?(@.type=="Ready")].status}' && echo " OK" || echo " FAIL"
   ```

   Example output:

   ```
   phase=Running ready=True OK
   ```

   The output should show **OK**. If the output shows **FAIL**, ensure you followed all previous steps and they produced the expected result.

2. Check whether you are collecting traces and metrics from your **GuardrailsOrchestrator** instance:

1. Check the **spec.otelExporter** field configuration in your **GuardrailsOrchestrator** CR:

   ```bash
   export OTEL_SPEC=$(oc get guardrailsorchestrator "$GORCH_NAME" -n "$NS" -o jsonpath='{.spec.otelExporter}' 2>/dev/null || true)
   export OTEL_LEN=$(echo "$OTEL_SPEC" | jq -r 'keys | length' 2>/dev/null)
   [ "$OTEL_LEN" -gt 0 ] 2>/dev/null && echo "spec.otelExporter present" || echo "spec.otelExporter missing"
   ```

   The output is one of the following results:  
   * **spec.otelExporter missing**: If you have additional **GuardrailsOrchestrator** instances, repeat the steps in this procedure. If you have no additional instances, you have completed the Before upgrade \- Guardrails Orchestrator steps.

     * **spec.otelExporter present**. Continue to the next step to backup your **spec.otelExporter** keys.

2. If **spec.otelExporter** is present, backup your traces and metrics exporter configuration by saving it to **${BACKUP\_DIR}/$GORCH\_NAME-$NS-otelExporter-backup.json**:

   ```bash
   oc get guardrailsorchestrator $GORCH_NAME -n $NS -o jsonpath='{.spec.otelExporter}' > ${BACKUP_DIR}/$GORCH_NAME-$NS-otelExporter-backup.json
   cat ${BACKUP_DIR}/$GORCH_NAME-$NS-otelExporter-backup.json
   ```

   Example output:

   ```
   {"otlpEndpoint":"http://my-otelcol-collector.test-guardrails-tempo-hf.svc.cluster.local:4317","otlpExport":"metrics,traces","protocol":"grpc"}
   ```

   The output should be a non-empty dictionary. If you have no additional **GuardrailsOrchestrator** instances, you have completed the Before upgrade \- Guardrails Orchestrator procedure. Otherwise, repeat the steps in this procedure for additional instances.

##
## **2.8. Workbenches \- Before upgrade** {#2.8.-workbenches---before-upgrade}

### **2.8.1. About upgrading your workbenches** {#2.8.1.-about-upgrading-your-workbenches}

As a Red Hat OpenShift AI administrator, you have flexibility in your upgrade strategy for your workbench images and server.

There are two major upgrade paths to follow. As a Red Hat OpenShift AI administrator, you can either manage the upgrade fully by ensuring all workbenches have been stopped and workbench images have been migrated prior to the upgrade, or defer migration and enable your users to continue their workbench image migration with user self-service. Deferred migrations come with additional risk, so it is important to understand the impacts of your upgrade path. See [Perform a deferred workbench image migration](#4.7.2.-perform-a-deferred-workbench-image-migration) for more information.

**Considerations before upgrading your workbench**

* Workbenches contain active user development environments. Coordinate with all users before upgrading to prevent loss of unsaved work.

* Workbench URLs will change after the upgrade. Users must obtain their new URLs from the OpenShift AI dashboard. Previously bookmarked URLs will no longer work.

* Organizations using custom workbench images must rebuild them to support the new authentication layer and path-based routing in OpenShift AI version 3.5. Custom workbench images built for OpenShift AI version 2.x are not compatible with the new **kube-rbac-proxy** authentication mechanism. See [Introducing the Kubernetes Gateway API for custom image migration](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_resources/introducing-kubernetes-gateway-api_resource-mgmt) for more information.

* RStudio workbenches require a new build from the RStudio BuildConfig after the upgrade. The existing image will not be compatible with the new authentication layer without this rebuild.

* Workbenches created in Red Hat OpenShift AI version 2.25 or earlier are officially unsupported in the Red Hat OpenShift AI 3.5 environment unless they have been manually migrated.

  **Important**  
  Workbenches that are not migrated will remain on the OpenShift AI 2.25.9 (and later) authentication layer. This legacy setup, paired with potential **NB\_PREFIX** routing conflicts, often results in redirection loops or connectivity failures—particularly for RStudio, Code Server, or custom images.

###  **2.8.2. Prepare your workbenches for migration** {#2.8.2.-prepare-your-workbenches-for-migration}

**Important**

All procedures in this section must be completed before upgrading the Red Hat OpenShift AI Operator from version 2.25.9 (and later) to 3.5. Failure to complete these steps can result in service disruptions for your workbenches.

**Prerequisites**

* You have OpenShift AI administrator access.

* You have access to a system with Bash and the OpenShift CLI (**oc**) for performing updates.

* Your OpenShift AI Operator version is at the latest patch release.

* Your OpenShift AI users have been notified of the impending upgrade and have taken appropriate measures to:

  * Prevent the loss of unsaved work.

  * Ensure that all workbenches are either in a Stopped or Running state.

    If a workbench is unable to enter a Running state, you must either stop or delete these workbenches before continuing with migration.

**Procedure**

1. Evaluate whether your image version tag needs an update:

   * If you are using OpenShift AI Jupyter-based workbenches, it is *recommended* to update your workbench image tags to the latest version (2025.2).

   * If you are using OpenShift AI code-server workbenches, it is **required** to update your workbench image tags to the latest version (2025.2).

   * If you are using the OpenShift AI RStudio buildconfig, it is **required** to be on the latest version tag, titled “latest”. The tag version will update after it is rebuilt.

     **Note:** If you are using the OpenShift AI RStudio buildconfig, you will need a new build after your workbench upgrade.

   * If you are using custom workbench images, you must migrate your workbench images to the Kubernetes Gateway API to leverage path-based routing and maintain compatibility with Red Hat OpenShift AI version 3.5. See [Introducing the Kubernetes Gateway API for custom image migration](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_resources/introducing-kubernetes-gateway-api_resource-mgmt) for more information.

     **Note**  Once you have rebuilt an image with the Kubernetes Gateway API support to enable path-based routing, publish it with a new tag and import it into your ImageStream. It is **required** to be on this latest published version tag. See [Importing a custom workbench image](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_openshift_ai/creating-custom-workbench-images#importing-a-custom-workbench-image_custom-images) for more information.

2. Stop your running workbenches.

   * Workbench images left unmigrated continue to operate on the older 2.25.9 (and later) authentication layer. This hybrid environment may result in redirection loops and connectivity failures, primarily due to **NB\_PREFIX** routing conflicts for RStudio, code-server, and custom images.

   * If your users are unable to stop all workbenches before the upgrade from OpenShift AI version 2.25.9 (and later) to 3.5, you can elect to defer image migration until after upgrading to Red Hat OpenShift AI version 3.5. Choosing to defer your image migration introduces additional complexity and risk to your upgrade process. See [Perform a deferred workbench image migration](#4.7.2.-perform-a-deferred-workbench-image-migration) for more information.

**Verification**

1. Perform a compatibility check on your workbenches by running the following command:

   **Note**  
   Your workbenches are ready to be upgraded if your compatibility check returns with **PASS** or **WARNING** results in the output. A **FAIL** result requires resolution before proceeding to upgrade your workbench.

   ```bash
   rhai-cli lint --target-version 3.5 --checks "*notebook*"
   ```

   **Example output:**

      ```bash

      │ STATUS  KIND      GROUP     CHECK                         IMPACT   MESSAGE                                                                          │  
      ├────────────────────────────────────────────────────────────────────────────┤  
      │ ⚠       notebook  workload  impacted-workloads            warning  Found 9 Notebook(s) using 6 unique images:                                       │  
      │                                                                    \- 7 compatible (4 images, OOTB ready for 3.5)                                    │  
      │                                                                    \- 1 custom (1 images, user verification needed)                                  │  
      │                                                                    \- 0 incompatible (0 images, update recommended before upgrade)                   │  
      │                                                                    \- 1 incompatible (1 images, must rebuild after upgrade to 3.x)                   │  
      │                                                                    \- 0 unverified (0 images, could not determine status)                            │  
      │ ✓       notebook  workload  acceleratorprofile-migration  info     No Notebooks found using deprecated AcceleratorProfiles \- no migration needed    │  
      │ ✓       notebook  workload  config-migration              info     No Notebooks found with container name mismatch                                  │  
      │ ✓       notebook  workload  config-migration              info     No Notebooks found with legacy hardware profile annotation \- no migration needed │  
      │ ✓       notebook  workload  data-integrity                info     All Notebook connections reference existing Secrets                              │  
      │ ✓       notebook  workload  data-integrity                info     All Notebooks reference existing HardwareProfiles                                │  
      │ ✓       notebook  workload  workload-state                info     All Notebooks are stopped                                                        │  
      └─────────────────────────────────────────────────────────────────────────────  
      Environment:  
      OpenShift AI version: 2.25.9 (and later) \-\> 3.5  
      OpenShift version:    4.20.0

      Summary:  
      Total: 7 | Passed: 6 | Warnings: 1 | Failed: 0 | Prohibited: 0

      Result:  
      WARNING \- advisory findings detected

    ```

2. Verify that all workbenches eligible for upgrading are in a stopped state across all namespaces with the following command:

   ```bash
   oc get notebooks -A -o custom-columns="NAMESPACE:.metadata.namespace,NAME:.metadata.name,RUNNING:.status.readyReplicas,STARTED:.status.containerState.running.startedAt"
   ```

   **Example output**

   ```
   NAMESPACE             NAME                     RUNNING   STARTED
   rhods-notebooks       jupyter-nb-rhoai-admin   0         <none>
   workbenches-hwp       codeserver-2025-2        0         <none>
   workbenches-hwp       jupyter-2025-1           0         <none>
   workbenches-regular   codeserver-2025-1        0         <none>
   workbenches-regular   codeserver-2025-2        0         <none>
   workbenches-regular   custom                   0         <none>
   workbenches-regular   jupyter-2025-1           0         <none>
   workbenches-regular   jupyter-2025-2           0         <none>
   workbenches-regular   rstudio-latest           0         <none>
   ```

## 

## **2.9. Ray Training Operator \- Before upgrade** {#2.9.-ray-training-operator---before-upgrade}

Before upgrading from Red Hat OpenShift AI to 3.5, you must prepare your existing Ray clusters to work with the 3.5 architecture.

**WARNING:**  Only complete the following procedure when you are ready to upgrade the OpenShift AI Operator to 3.5. The migration that you run in the following procedure removes the Codeflare Operator. The Codeflare Operator provides essential security configuration to any RayClusters that your users create. If you run the Ray pre-upgrade migration and then delay the OpenShift AI Operator upgrade, users can potentially create new RayClusters when the Codefare Operator is not installed and risk exposing those RayClusters to security issues.

The **rhai-cli** tool includes Ray cluster migration support to help with the following tasks:

* Verification that your cluster is ready for the upgrade.

* Backup of your Ray cluster configurations before the OpenShift AI upgrade.

* Migration of your Ray clusters after the upgrade is complete.

The **rhai-cli migrate** Ray migration is safe and predictable with the following features:

* Staged approach: You can target a single cluster before using it to migrate all your Ray clusters.

* Idempotent: You can safely run the migration many times.

* Non-destructive: The migration creates Ray cluster Custom Resource (CR) backups. It does not delete anything automatically, unless you use the **\--from-backup** option for recovery procedures.

**Important**

Before you begin the migration process, warn your users that it will cause temporary downtime.

**Prerequisites**

* **WARNING:** You must follow the [Before upgrade steps for the Workbench component](#2.8.-workbenches---before-upgrade) before you migrate your Ray clusters.

* You have cluster administrator access.

  **Note**  
  You should conduct the following procedure in tandem with the OpenShift AI administrator and the users who created the Ray clusters that you want to migrate.

When you run the Ray migration, it checks that you have the following permissions:

   ```yaml
rules:
  # Core resources
  - apiGroups: [""]
    resources: ["namespaces"]
    verbs: ["list"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list"]
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["list", "delete"]

  # Ray clusters - full access for migration
  - apiGroups: ["ray.io"]
    resources: ["rayclusters"]
    verbs: ["get", "list", "create", "update", "patch", "delete"]

  # Gateway API - for dashboard URL discovery
  - apiGroups: ["gateway.networking.k8s.io"]
    resources: ["httproutes"]
    verbs: ["list"]
  - apiGroups: ["gateway.networking.k8s.io"]
    resources: ["gateways"]
    verbs: ["get"]

  # OpenShift Routes - fallback for Gateway hostname
  - apiGroups: ["route.openshift.io"]
    resources: ["routes"]
    verbs: ["get"]

  # DataScienceCluster (for pre-upgrade check)
  - apiGroups: ["datasciencecluster.opendatahub.io"]
    resources: ["datascienceclusters"]
    verbs: ["list"]

  # Permission checking
  - apiGroups: ["authorization.k8s.io"]
    resources: ["selfsubjectaccessreviews"]
    verbs: ["create"]
   ```

* You have access to the **rhai-cli** tool.

* You have run the migration assessment and the result indicated that you must upgrade the Ray Training Operator.

**Procedure**

1. Set CodeFlare to **Removed** in the DataScienceCluster before running the Ray backup:

   ```bash
   oc patch datasciencecluster default-dsc --type merge \
       -p '{"spec":{"components":{"codeflare":{"managementState":"Removed"}}}}'
   ```

   **Important**
   The `raycluster.backup` action checks that CodeFlare is **Removed** as a precondition and fails if it is still **Managed**. You must remove CodeFlare before proceeding.

   Verify:

   ```bash
   oc get datasciencecluster default-dsc -o jsonpath='{.spec.components.codeflare.managementState}' && echo
   ```

   Expected output: `Removed`

2. Create a folder for Ray cluster backup in `/tmp/rhoai-upgrade-backup`

   ```bash
   mkdir /tmp/rhoai-upgrade-backup/ray_cluster
   ```
3. Run a pre-upgrade check that verifies your configuration, verifies that the Ray clusters are ready for the upgrade, and backs up your Ray cluster CR configuration YAML files:  
   **Note**  
   The migration backs up your Ray cluster CR configuration YAML files only. It does not back up the state of your Ray clusters.  
   ```bash
   rhai-cli migrate run --migration raycluster.backup --target-version 3.5.0  --raycluster-output-dir /tmp/rhoai-upgrade-backup/ray_cluster
   ```
   The migration runs pre-upgrade checks. If all pre-upgrade checks succeed, then it saves your Ray cluster CR configurations to the following subdirectories under the backup directory:

   * **Rhoai-2.x** \- Your Ray cluster CR configurations YAML files that are compatible with OpenShift AI 2.x.

   * **Rhoai-3.x** \- Your Ray cluster CR configurations YAML files that are compatible with OpenShift AI 3.x.

4. Get a list of the Ray clusters and check their status:

   ```bash
   rhai-cli migrate list --target-version 3.5.0
   ```

   The output should be similar to the following:

   ```
   Fetching RayClusters (all namespaces)...
   Found 2 RayCluster(s)
   Analyzing clusters...
   Analysis complete.

   RayCluster Migration Status:

   Name                   Namespace      Status      Workers   Migration Status
   ----------------------------------------------------------------------------
   comprehensive-mixed    raytest        ready        2        [NEEDS MIGRATION]
   sdk-configurations     raytest        ready        1        [NEEDS MIGRATION]

   Migration Summary: 0 migrated, 2 need migration
   ```

**Verification**

If the pre-upgrade check is successful, the output is similar to the following example:

```
Starting Ray cluster pre-upgrade...

Connecting to Kubernetes cluster...
Connected.

Running pre-upgrade checks...
------------------------------------------------------------
  [OK] Permissions: All required permissions to perform the
       ray upgrade are available to this user.
  [OK] cert-manager: cert-manager CRD found
  [OK] codeflare-operator: codeflare is Removed in DSC
  [OK] RayClusters: 2 RayCluster(s) on cluster.
------------------------------------------------------------
All pre-upgrade checks passed.

Backup files will be saved to: /tmp/rhoai-upgrade-backup/ray

Backup complete: 2 RayCluster(s) saved to /tmp/rhoai-upgrade-backup/ray

INFO: The 'rhoai-2.x/' backups contain CodeFlare-operator components.
Use 'rhoai-2.x/' ONLY if attempting to restore RayClusters but did not proceed with the RHOAI 3.x upgrade.
Use 'rhoai-3.x/' for proceeding with the RHOAI 3.x upgrade.

Ray Pre Upgrade Steps Completed.
```

**Troubleshooting**

If the pre-upgrade check fails, the migration provides output similar to the following example with details about what checks failed and details on how to fix the issues. You must resolve any issues and then run the pre-upgrade check successfully before you upgrade from Red Hat OpenShift AI to 3.5.

```
Running pre-upgrade checks...
------------------------------------------------------------
  [FAIL] Permissions: Missing permissions
       - List namespaces: OK
       - List RayClusters: OK
       - Get RayClusters: OK
       - Update RayClusters: DENIED
       Missing permissions: Update RayClusters: DENIED
  [FAIL] cert-manager: cert-manager not detected
       cert-manager is required for RHOAI 3.x. Install it via
       OperatorHub before proceeding with the upgrade.
------------------------------------------------------------

Pre-upgrade checks failed. Please resolve the issues above before
proceeding with the RHOAI upgrade.

WARNING: Proceeding with the RHOAI upgrade without resolving these issues may result in your Ray infrastructure becoming unavailable or unrecoverable.
```

## 

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

Before migrating Model Serving workloads from Red Hat OpenShift AI 2.25.9 (and later) to 3.5, verify that your environment meets the following requirements.

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
   oc get isvc -n <namespace> -o json | jq -r '["NAME","DEPLOYMENT_MODE","READY"], (.items[] | [.metadata.name, .status.deploymentMode, (.status.conditions[] | select(.type=="Ready") | .status)]) | @tsv' | column -t
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
      oc get isvc -n <namespace> -o json | jq -r '.items[] | select(.status.deploymentMode == "Serverless" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "Serverless") | .metadata.name'
      ```

      Expected output:

      ```
      my-serverless-isvc
      another-sl-model
      ```

   2. Delete them (run from your workstation due to the need for `jq`):

      ```bash
      oc get isvc -n <namespace> -o json | jq -r '.items[] | select(.status.deploymentMode == "Serverless" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "Serverless") | .metadata.name' | while read -r name; do echo "Deleting  Serverless InferenceService: $name"; oc delete isvc "$name" -n <namespace>; done
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
   oc get isvc -n <namespace> -o json | jq -r '["NAME","DEPLOYMENT_MODE","READY"], (.items[] | [.metadata.name, .status.deploymentMode, (.status.conditions[] | select(.type=="Ready") | .status)]) | @tsv' | column -t
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
      oc get isvc -n <namespace> -o json | jq -r '.items[] | select(.status.deploymentMode == "ModelMesh" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "ModelMesh") | .metadata.name'
      ```

      Expected output:

      ```
      My-modelmesh-isvc
      ```

   2. Delete them (run from your workstation due to the need for `jq`):

      ```bash
      oc get isvc -n <namespace> -o json | jq -r '.items[] | select(.status.deploymentMode == "ModelMesh" or .metadata.annotations["serving.kserve.io/deploymentMode"] == "ModelMesh") | .metadata.name' | while read -r name; do echo "Deleting ModelMesh InferenceService: $name"; oc delete isvc "$name" -n <namespace>; done
      ```

      Expected output:

      ```
      Deleting ModelMesh InferenceService: my-modelmesh-isvc
      inferenceservice.serving.kserve.io "my-modelmesh-isvc" deleted
      ```

   3. Delete the ModelMesh ServingRuntimes (multi-model runtimes) (run from your workstation due to the need for `jq`):

      ```bash
      oc get servingruntimes.serving.kserve.io -n <namespace> -o json | jq -r '.items[] | select(.spec.multiModel==true) | .metadata.name' | while read -r name; do echo "Deleting ServingRuntime: $name"; oc delete servingruntime "$name" -n <namespace>; done
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

   **Note**: These commands use the DSC v1 API field names (`modelmeshserving`, `serviceMesh`) because they are run against your OpenShift AI 2.25.9 cluster before upgrade. After upgrading to 3.5, the operator automatically converts to the v2 API.

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

4. If the Serverless Operator is installed, uninstall it: 

   1. In the OpenShift web console, navigate to **Operators** → **Installed Operators**.

   2. For the **Projects** field, make sure that **All projects** are selected.

   3. Locate the **Red Hat OpenShift Serverless** Operator.

   4. Click the Options menu (⋮) and select **Uninstall Operator**.

   5. In the confirmation dialog, select **Delete all operand instances for this operator**.

   6. Click **Uninstall**.

5. If the Service Mesh 2 Operator is installed, uninstall it:

   1. In the OpenShift web console, navigate to **Operators** → **Installed Operators**.

   2. For the **Projects** field, make sure that **All projects** are selected.

   3. Locate the **Red Hat OpenShift Service Mesh 2** Operator.

   4. Click the Options menu (⋮) and select **Uninstall Operator**.

   5. In the confirmation dialog, select **Delete all operand instances for this operator**.

   6. Click **Uninstall**.

6. If standalone Authorino is installed, uninstall it:

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

1. Create the Kuadrant custom resource:

   ```bash
   oc apply -f - <<EOF
   apiVersion: kuadrant.io/v1beta1
   kind: Kuadrant
   metadata:
     name: kuadrant
     namespace: kuadrant-system
   EOF
   ```

2. Wait for Kuadrant to become ready:

   ```bash
   oc wait Kuadrant -n kuadrant-system kuadrant --for=condition=Ready --timeout=10m
   ```

3. Add the **ServingCert** annotation to the Authorino service:

   ```bash
   oc annotate svc/authorino-authorino-authorization service.beta.openshift.io/serving-cert-secret-name=authorino-server-cert -n kuadrant-system
   ```

   Wait a few seconds for the annotation to be processed:

   ```bash
   sleep 2
   ```

4. Update Authorino to enable SSL:

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

5. Verify that the Authorino pods are ready:

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

* [Installing Connectivity Link on OpenShift](https://docs.redhat.com/en/documentation/red_hat_connectivity_link/1.2/html/installing_connectivity_link_on_openshift/index)

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

3. Configure Kuadrant Operator Subscription with mirrored WASM image by running the following command, replacing \<wasm-shim-sha\> with the SHA of the wasm-shim image that you identified in Step 1\.

   ```bash
   export MIRROR_REGISTRY="<bastion-mirror-registry>:<bastion-mirror-registry-port>"
   export WASM_IMAGE_DIGEST="<wasm-shim-sha>"
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
   export MIRROR_REGISTRY="<bastion-mirror-registry>:<bastion-mirror-registry-port>"
   export GATEWAY_NAME=<your-gateway-name>
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
   oc annotate llminferenceservice <LLMISVC-NAME> -n <LLMISVC-NAMESPACE> security.opendatahub.io/enable-auth=false
   ```

   Replace *\<LLMISVC-NAME\>* with your **LLMInferenceService** name and *\<LLMISVC-NAMESPACE\>* with your project namespace.  
   The model becomes public and no authentication tokens are required.  
   **Method 2: Configure RBAC access control (Recommended)**  
   To keep the model secure, create a **ServiceAccount** with permissions to access the **LLMInferenceService**:

2. Create a **ServiceAccount**:

   ```yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: my-llmisvc-sa
     namespace: <my-project>
   ```

3. Create a **Role** with **get** permission:

   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     name: my-llmisvc-role
     namespace: <my-project>
   rules:
   - apiGroups: ["serving.kserve.io"]
     resources: ["llminferenceservices"]
     resourceNames: ["<my-llm-name>"]
     verbs: ["get"]
   ```

   Create a **RoleBinding**:

   ```yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: RoleBinding
   metadata:
     name: my-llmisvc-rolebinding
     namespace: <my-project>
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
   TOKEN=$(oc create token my-llmisvc-sa -n <my-project>)
   curl -H "Authorization: Bearer $TOKEN" https://<model-url>/v2/models/...
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

Pin your **LLMInferenceService** configurations to use Red Hat OpenShift AI 2.25.9 (and later) templates to prevent scheduler pod failures during the upgrade to version 3.5.

**Prerequisites**

* You have **LLMInferenceService** resources deployed.

* You have access to projects containing **LLMInferenceService** resources.

**Procedure**

1. Pin **LLMInferenceService** configurations to use RHOAI 2.25.9 (and later) templates to prevent scheduler pod failures during upgrade:

   ```bash
   oc patch llmisvc <LLMISVC-NAME> -n <LLMISVC-NAMESPACE> \
       --subresource=status \
       --type=merge \
       -p '{ "status": { "annotations": { "serving.kserve.io/config-llm-template": "kserve-config-llm-template", "serving.kserve.io/config-llm-decode-template": "kserve-config-llm-decode-template", "serving.kserve.io/config-llm-worker-data-parallel": "kserve-config-llm-worker-data-parallel", "serving.kserve.io/config-llm-decode-worker-data-parallel": "kserve-config-llm-decode-worker-data-parallel", "serving.kserve.io/config-llm-prefill-template": "kserve-config-llm-prefill-template", "serving.kserve.io/config-llm-prefill-worker-data-parallel": "kserve-config-llm-prefill-worker-data-parallel", "serving.kserve.io/config-llm-scheduler": "kserve-config-llm-scheduler", "serving.kserve.io/config-llm-router-route": "kserve-config-llm-router-route" } } }'
   ```

   Replace *\<LLMISVC-NAME\>* with your **LLMInferenceService** name and *\<LLMISVC-NAMESPACE\>* with your project namespace.

   **Important**  
   If you are overriding **LLMInferenceService** scheduler arguments, you must update them for Red Hat OpenShift AI 3.x compatibility. The following breaking changes apply in Red Hat OpenShift AI 3.5 and later:

   * Argument naming: Arguments changed from **camelCase** to **kebab-case** (for example, **\--certPath** becomes **\--cert-path**)

   * TLS certificate path: Default location moved from **/etc/ssl/certs** to **/var/run/kserve/tls**

   * Mandatory TLS: Signed TLS certificates via OpenShift service signer are required

   * Required configuration: Must include **\--cert-path** argument and **SSL\_CERT\_DIR** environment variable

     Following is an example Diff of an LLMInferenceService configuration for the scheduler arguments and environment variables from 2.25.9 (and later) to 3.5:

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
  oc get llmisvc <LLMISVC-NAME> -n <LLMISVC-NAMESPACE> -o jsonpath='{.status.annotations}'
  ```

  The output displays the pinned template annotations.

* Verify that the **LLMInferenceService** migration steps were completed correctly from inside the **rhai-cli** container:

  ```bash
  /opt/rhai-cli/bin/rhai-cli lint --target-version 3.5
  ```

  The command completes without errors and confirms migration readiness.

### **2.10.11. Verify migration readiness** {#2.10.11.-verify-migration-readiness}

Verify that all migration prerequisites have been completed and your cluster is ready for the Red Hat OpenShift AI operator upgrade from version 2.25.9 (and later) to 3.5.

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

Do not proceed with the Red Hat OpenShift AI operator upgrade from version 2.25.9 (and later) to 3.5 if the **rhai-cli** output shows any critical issues. Address all critical issues by completing the relevant component migration procedures before upgrading.

## **2.11. Kubeflow Training Operator \- Before upgrade** {#2.11.-kubeflow-training-operator---before-upgrade}

You can upgrade Red Hat OpenShift AI 2.25.9 (and later) to 3.5 while PyTorchJobs are running; the jobs continue to run during the upgrade process and complete as normal.

Before you upgrade to OpenShift AI 3.5, get a list of PyTorchJob resources on your OpenShift cluster. You can then use this list to compare against PyTorchJob resources on your OpenShift cluster after you upgrade to 3.5.

**Note**

The Kubeflow Training Operator (KFTO) v1 is deprecated starting with theOpenShift AI 2.25.9 (and later) and is planned to be removed in a future release. This deprecation is part of the OpenShift AI transition to Kubeflow Trainer v2, which delivers enhanced capabilities and improved functionality.

**Prerequisites**

* You have cluster administrator access to your cluster.

* You have logged in to your OpenShift cluster.

**Procedure**

* Run the following command to get a list of PyTorchJob resources on your OpenShift cluster:  
  ```bash
  oc get pytorchjobs -A
  ```

**Verification**

The command returns a list of PyTorchJob resources, as shown in this example output:  

```bash
NAMESPACE          NAME                           STATE       AGE  
pytorch-training   pytorch-distributed-training   Running     4m27s
```

**Warning**

As a cluster administrator, if you want to perform an OpenShift Container Platform (OCP) upgrade, an OCP upgrade process might stop nodes which might interrupt PyTorchJobs. Ensure that either no PyTorchJobs are running during the OCP upgrade or verify that the running PyTorchJobs include checkpointing so that they are resilient to failure.

## **2.12. OpenShift AI Operator \- Before upgrade** {#2.12.-openshift-ai-operator---before-upgrade}

Before upgrading Red Hat OpenShift AI from version 2.25.9 (and later) to 3.5, complete the following steps to ensure a successful migration of the OpenShift AI Operator.

**Note**

If you have bookmarked dashboard URLs, you must recreate redirects **after** the upgrade is complete. For more information, see the [Resolving dashboard URL 404 errors after upgrading from 2.x to 3.x](https://access.redhat.com/solutions/7137771).

**Prerequisites**

* You have upgraded to OpenShift 4.19.9 or later according to OpenShift documentation on [Updating clusters](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/updating_clusters/index).

* You have set the **Update approval** for the Red Hat OpenShift AI 2.25.9 (and later) subscription to **Manual**. This prevents unintended automatic upgrades and requires you to explicitly confirm the upgrade.

* Kueue is set to **Removed** or **Unmanaged** (with external Red Hat build of Kueue Operator installed).

* You have completed the Migrate **InferenceServices** to **RawDeployment** mode steps to convert all serving deployments to **RawDeployment** mode and removed the OpenShift Service Mesh 2 Operator.

* You have configured Model Serving to ignore hardware profile annotations to avoid inference service restarts during the upgrade, according to Update the inferenceservice-config ConfigMap.

* You have set the **CodeFlare** component to **Removed** in the DataScienceCluster resource. CodeFlare is removed in OpenShift AI 3.5 and must be disabled before upgrading, even if you have no RayClusters. If you completed the Ray pre-upgrade migration (Section 2.7), you already performed this step. Otherwise, run:

  ```bash
  oc patch dsc default-dsc --type=merge \
    -p '{"spec":{"components":{"codeflare":{"managementState":"Removed"}}}}'
  ```

* You migrated any other component workloads that require migration before the upgrade.

* You have OpenShift cluster administrator permissions to install Operators and edit **DataScienceCluster** and **DataScienceClusterInitialization** resources.

**Procedure**

1. Verify that the **Update approval** for the Red Hat OpenShift AI 2.25.9 (and later) subscription is set to **Manual**.

   If the **Update approval** is not set to **Manual**, you must set it now. This prevents automatic upgrade when you change the subscription channel.

2. Edit the **Update channel** for Red Hat OpenShift AI to **support-required-upgrade-3.5**.

   **Note**: For cross-major upgrades from 2.25 to 3.5, only the **support-required-upgrade-3.5** channel provides a valid upgrade path. Other 3.x channels such as **stable-3.5** or **stable-3.x** are for same-major upgrades only and do not provide an upgrade from 2.25.

   For information about subscription channels and their lifecycle, see [Red Hat OpenShift AI Self-Managed Life Cycle](https://access.redhat.com/support/policy/updates/rhoai-sm/lifecycle#stable).

**Verification**

1. Verify that the Red Hat OpenShift AI 2.25.9 (and later) CSV status shows **Succeeded**.  
   ```bash
   oc get csv -n redhat-ods-operator
   ```

   **Note**  
   If you are using a custom operator namespace, replace **redhat-ods-operator** with your specific namespace names.

2. Verify that the **DataScienceCluster** (DSC) and **DSCInitialization** (DSCI) custom resources show a status of **Ready**.

   ```bash
   oc get dsc -o custom-columns='NAME:.metadata.name,STATUS:.status.phase'
   oc get dsci -o custom-columns='NAME:.metadata.name,STATUS:.status.phase'
   ```

   **Important**  
   The reconciliation might take time to complete. Do not proceed with the upgrade if the **DSC** and **DSCI** custom resources show errors in the **Status** sections.

3. Verify that all operator pods in the operator namespace have a status of **Running** and their **Ready** condition is **True**.

   ```bash
   oc get pods -n redhat-ods-operator -o custom-columns='NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status,STATUS:.status.phase'
   ```

   **Note**  
   If you are using a custom operator namespace, replace **redhat-ods-operator** with your specific namespace names.

4. Verify that all component controller pods in the applications namespace have a status of **Running** and their **Ready** condition is **True**.

   ```bash
   oc get pods -n redhat-ods-applications -o custom-columns='NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status,STATUS:.status.phase'
   ```

   **Note**  
   If you are using a custom operator namespace, replace **redhat-ods-applications** with your application namespace.

5. Follow the Migration assessment script steps to run a final full cluster scan, and confirm that OpenShift AI is ready for the upgrade with the summary indicating that  Failed is 0\.

# 

# 

# **Chapter 3\. Upgrade to 3.5** {#chapter-3.-upgrade-to-3.5-latest}

## **3.1. OpenShift AI Operator**  {#3.1.-openshift-ai-operator}

After preparing your cluster and changing the subscription channel, you must manually approve the upgrade plan to begin the transition to the new version.

**Prerequisites**

* You have completed all the before upgrade tasks and verified that the cluster is ready for upgrade.

* Rerun the rh-ai  assessment script to make sure all critical issues are resolved.

* For disconnected environments, you have a mirror registry and oc-mirror v2, as described in [Mirroring images for a disconnected installation by using the oc-mirror plugin v2](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/disconnected_environments/about-installing-oc-mirror-v2).

**Procedure**

1. You must log out of the OpenShift AI dashboard before starting the upgrade. OpenShift AI does not support Zero Downtime Upgrade.

   For connected environments, skip to Step 5\.

   For disconnected environments, continue to Step 2\.

2. For disconnected environments:

   Identify the OSSM version the Cluster Ingress Operator requires.  
   In the following steps, replace \<ossm-version\>  with this value (for example, servicemeshoperator3.v3.1.0):

   ```bash
   oc set env deployment/ingress-operator -n openshift-ingress-operator --list \
       | grep GATEWAY_API_OPERATOR_VERSION \
       | sed 's/.*=//'
   ```

3. Identify the OSSM channel the Cluster Ingress Operator uses to install OSSM.   
   In the following steps  replace \<ossm-channel\> with this value (for example,  stable):

   ```bash
   oc set env deployment/ingress-operator -n openshift-ingress-operator --list \
       | grep GATEWAY_API_OPERATOR_CHANNEL \
       | sed 's/.*=//'
   ```

### 

4. Mirror the exact OSSM version identified in Step 2 into the disconnected registry:  
   1. Create the ImageSetConfiguration. Replace \<ocp-version\>, \<ossm-version\> and \<ossm-channel\> with your values:

      ```bash
      cat > imageset-config.yaml <<EOF
      apiVersion: mirror.openshift.io/v2alpha1
      kind: ImageSetConfiguration
      mirror:
        operators:
          - catalog: registry.redhat.io/redhat/redhat-operator-index:v<ocp-version>
            packages:
              - name: servicemeshoperator3
                channels:
                  - name: <ossm-channel>
                    minVersion: <ossm-version>
                    maxVersion: <ossm-version>
      EOF
      ```

      

      **b.** Run oc-mirror to mirror the images. Replace \<mirror-registry\> with your registry URL:

      

      ```bash
      oc-mirror --v2 --config=imageset-config.yaml \
          --workspace file://oc-mirror-workspace \
          docker://<mirror-registry>
      ```

   

      **c.** If CatalogSource redhat-operators already exists on the cluster, skip this step and continue with step 4d  to verify that it references the version you just mirrored in step 4b. 

      If CatalogSource redhat-operators doesn’t exist on the cluster, change the name of the CatalogSource generated by oc-mirror to redhat-operators and apply the generated cluster resources so that  the cluster is aware of the mirrored content: 

   ```bash
   oc apply -f oc-mirror-workspace/working-dir/cluster-resources/
   ```

   

   **d.** Verify that the required version of OSSM is available in the required channel in the mirrored CatalogSource named redhat-operators:

   ```bash
   oc get packagemanifest -o json | jq '.items[] | select (.metadata.name == "servicemeshoperator3" and .status.catalogSource == "redhat-operators") | .status.channels[] | select (.name == "<ossm-channel>") | .entries[].name'
   ```

   

   The output should include the \<ossm-version\> from Step 2\. If it doesn’t include the version from Step 2, make sure that the CatalogSource named redhat-operators references it.

5. **FBC (File-Based Catalog) environments only:** If you installed Red Hat OpenShift AI using a custom FBC CatalogSource (for example, for pre-release testing), you must update the CatalogSource image to the target version before switching channels. The source FBC fragment only contains channels up to the source version.

   ```bash
   oc patch catalogsource <catalog-name> -n openshift-marketplace \
     --type=merge -p '{"spec":{"image":"<target-fbc-fragment-image>"}}'
   ```

   Delete the catalog pod to force OLM to rebuild its cache. Without this step, OLM may serve stale channel data even after the CatalogSource reports READY:

   ```bash
   oc delete pod -n openshift-marketplace -l olm.catalogSource=<catalog-name>
   ```

   Wait for the CatalogSource to reach **READY** state:

   ```bash
   oc get catalogsource <catalog-name> -n openshift-marketplace \
     -o jsonpath='{.status.connectionState.lastObservedState}'
   ```

   Verify that the **support-required-upgrade-3.5** channel is now available. If your cluster also has the default **redhat-operators** CatalogSource, you must query the packagemanifest from your custom FBC CatalogSource specifically, because `oc get packagemanifest` may return data from **redhat-operators** by default:

   ```bash
   oc get packagemanifest -n openshift-marketplace -o json | \
       jq -r '.items[] | select(.metadata.name == "rhods-operator" and
       .status.catalogSource == "<catalog-name>") |
       .status.channels[].name' | grep support
   ```

   If you installed Red Hat OpenShift AI from the default **redhat-operators** CatalogSource, skip this step.

6. Log in to the OpenShift cluster web console as a cluster administrator.

7. In the Administrator perspective, in the left menu, select **Operators** \>  
    **Installed Operators**.

8. Click the **Red Hat OpenShift AI Operator**.

9. Click the **Subscription** tab.

10. For the **Upgrade channel**, select **support-required-upgrade-3.5**.

   **NOTE**:   
   Several other 3.x channels might be visible in the **Change Subscription update channels** list, such as fast-3.x, stable-3.5, and stable-3.x. However, these channels do not provide a cross-major upgrade from 2.25. Only the **support-required-upgrade-3.5** channel provides an upgrade from 2.25.9 or later to 3.5. The unversioned **support-required-upgrade** channel is for upgrading from 2.25 to 3.3 only.

11. Approve the install plan to begin the upgrade.

   1. In the **Upgrade status** section, click the "requires approval" link to approve the upgrade installation.  
   2. Review the upgrade install plan details and click **Approve**. The upgrade process begins.

12. While the upgrade is in progress, monitor the following:

   1. Watch the operator pods as they restart to replace the version 2.25.9 (and later) Operator.  
   2. Verify that the new operator pods reach the **Running** state and that the **Ready** condition is **True**.

13. Install the **JobSet** operator. OpenShift AI 3.5 requires the JobSet operator as a Kueue dependency. Without it, the **DataScienceCluster** remains in a **Not Ready** state with `KueueReady=False`.

   **Important**  
   The JobSet operator only supports **OwnNamespace** and **SingleNamespace** install modes. Do not install it in the `openshift-operators` namespace, which uses an **AllNamespaces** OperatorGroup.

   1. Create a dedicated namespace:

      ```bash
      oc create namespace jobset-system
      ```

   2. Create an **OwnNamespace** OperatorGroup:

      ```bash
      oc apply -f - <<'EOF'
      apiVersion: operators.coreos.com/v1
      kind: OperatorGroup
      metadata:
        name: jobset-operator-group
        namespace: jobset-system
      spec:
        targetNamespaces:
          - jobset-system
      EOF
      ```

   3. Subscribe to the operator:

      ```bash
      oc apply -f - <<'EOF'
      apiVersion: operators.coreos.com/v1alpha1
      kind: Subscription
      metadata:
        name: job-set
        namespace: jobset-system
      spec:
        channel: stable-v1.0
        installPlanApproval: Automatic
        name: job-set
        source: redhat-operators
        sourceNamespace: openshift-marketplace
      EOF
      ```

   4. Wait for the CSV to reach **Succeeded**:

      ```bash
      oc wait csv jobset-operator.v1.0.0 -n jobset-system \
        --for=jsonpath='{.status.phase}'=Succeeded --timeout=120s
      ```

      **Tip**  
      If the CSV name differs from `jobset-operator.v1.0.0`, verify it with `oc get csv -n jobset-system`.

   5. Create the **JobSetOperator** custom resource to deploy the operand. This installs the `jobsets.jobset.x-k8s.io` CRD that Kueue requires:

      ```bash
      oc apply -f - <<'EOF'
      apiVersion: operator.openshift.io/v1
      kind: JobSetOperator
      metadata:
        name: cluster
      spec:
        managementState: Managed
        logLevel: Normal
        operatorLogLevel: Normal
      EOF
      ```

   6. Verify that the CRD exists and KueueReady is True:

      ```bash
      oc get crd jobsets.jobset.x-k8s.io
      oc get dsc -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status}' && echo
      ```

      Expected output: the CRD is listed and KueueReady shows **True**.

14. Verify that the rhai-cli pod has cluster access for post-upgrade commands. The service account ClusterRoleBinding may need to be re-applied after the upgrade:

   ```bash
   oc auth can-i list csv -A --as=system:serviceaccount:rhai-migration:default
   ```

   If the output is **no**, restore the ClusterRoleBinding:

   ```bash
   oc adm policy add-cluster-role-to-user cluster-admin -z default -n rhai-migration
   ```

   **Note**  
   Replace `rhai-migration` with the namespace where your rhai-cli pod is deployed. This binding is required for all post-upgrade rhai-cli commands in Chapter 4.

# **Chapter 4\. After upgrading to 3.5** {#chapter-4.-after-upgrading-to-3.5-latest}

After you upgrade OpenShift AI from version 2.25.9 (and later) to 3.5, you must validate that workload migration was successful for each of the components installed on your OpenShift AI cluster.

##  **4.1. OpenShift AI Operator \- After upgrade** {#4.1.-openshift-ai-operator---after-upgrade}

After the upgrade process finishes, you must verify that the environment is stable and that all components are functioning correctly.

**Prerequisites**

* The Red Hat OpenShift AI 3.5 upgrade process has finished.

* In the **Installed Operators** page of the OpenShift web console:

  * The Red Hat OpenShift AI 3.5 Operator is listed, in addition to dependent Operators, and its ClusterServiceVersion (CSV) status is **Succeeded**.

  * The Red Hat OpenShift AI 2.25.9 (and later) Operator is no longer present.

**Procedure**

1. Confirm that the **DataScienceCluster** (DSC) and **DSCInitialization** (DSCI) resources are in a **Ready** state.

   ```bash
   oc get dsc -o custom-columns='NAME:.metadata.name,STATUS:.status.phase'
   oc get dsci -o custom-columns='NAME:.metadata.name,STATUS:.status.phase'
   ```

   **Note**  
   The reconciliation might take time to complete.

2. Check pod health in the Operator and applications namespaces.

   **Note**  
   In the following commands, replace the default **redhat-ods-operator** and **redhat-ods-applications** with your specific namespace names.

1. Verify that all Operator pods in the Operator namespace have **Status** equal to **Running** and their condition **Ready** is **True\`**.

   ```bash
   oc get pods -n redhat-ods-operator -o custom-columns='NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status,STATUS:.status.phase'
   ```

2. Verify that all component controller pods in the applications namespace have **Status** equal to **Running** and their condition **Ready** is **True**.

   ```bash
   oc get pods -n redhat-ods-applications -o custom-columns='NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status,STATUS:.status.phase'
   ```

3. Verify that the RHOAI gateway is ready:

   ```bash
   oc get gatewayconfigs --all-namespaces -o wide
   ```

   Expected output is: default-gateway shows READY: True

4. If the dashboard component is installed, verify that the console link exists and functions correctly.

1. Click the **Red Hat OpenShift AI** link in the OpenShift console to go to the log in page.  
2. Log in to the application and confirm that the dashboard is displayed.

5. Change the OpenShift AI Operator update channel:

1. In the OpenShift web console, select **Operators \> Installed Operators**.

2. Select the **Red Hat OpenShift AI 3.5 Operator**. 

3. Select Subscriptions and then select the Upgrade channel: **support-required-upgrade-3.5** 

4. From the list of channels, select the 3.x channel that you want to use going forward, for example: **stable-3.5** or **stable-3.x**.

   For more information about update channels, see [Understanding update channels.](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/installing_and_uninstalling_openshift_ai_self-managed/index#understanding-update-channels_install)

5. Click **Save**.

   If there are later OpenShift AI releases available, either later on the 3.5 release or for a later release, such as 3.4 or 3.5, the Operator subscription proposes install plans for them. 

6. Optionally, you can select to upgrade your environment to the most current release of OpenShift AI 3.x.

**Troubleshooting** 

Use the following information to help troubleshoot problems after you upgrade from Red Hat OpenShift AI 2.25.9 (and later) to 3.5.

**OpenShift AI components that depend on OpenShift Service Mesh (OSSM)  do not become ready**

After upgrading Red Hat OpenShift AI on a disconnected cluster, the Red Hat OpenShift Service Mesh (OSSM) 3.0 operator fails to install. OpenShift AI components that depend on OSSM do not become ready, for example:

- The `servicemeshoperator3` subscription in the `openshift-operators` namespace shows a failed state.  
- The `InstallPlan` for `servicemeshoperator3` fails or is not created.  
- The `DSCInitialization` (`default-dsci`) does not reach `Ready` state.  
- The `data-science-gateway` Gateway in `openshift-ingress` shows `Unknown` status.

For information on how to resolve this issue, see the [OpenShift Service Mesh 3.x fails to deploy on a disconnected cluster during Red Hat OpenShift AI installation](https://access.redhat.com/solutions/7141146) knowledgebase article.

**Resolving dashboard URL 404 errors**  
If you have bookmarked dashboard URLs, you must recreate redirects after the upgrade is complete. For more information, see the [Resolving dashboard URL 404 errors after upgrading from 2.x to 3.x](https://access.redhat.com/solutions/7137771).

**What the operator handles automatically**

During the upgrade from OpenShift AI 2.25.9 to 3.5, the operator automatically performs the following migrations on startup. Cluster administrators do not need to perform these steps manually:

* **HardwareProfile auto-migration**: The operator automatically migrates AcceleratorProfiles to HardwareProfiles and attaches HardwareProfile annotations to Notebooks and InferenceServices. This migration uses create-only semantics and preserves any user customizations.

* **GatewayConfig ingressMode migration**: The operator preserves the LoadBalancer ingress mode for existing Gateway deployments, preventing a regression to the default OcpRoute mode.

* **Kueue ValidatingAdmissionPolicyBinding cleanup**: The operator removes deprecated ValidatingAdmissionPolicyBinding resources from pre-v2.29.

* **DSC v1 to v2 API conversion**: The operator automatically handles the conversion from DSC v1 API to v2 API. Field renames (e.g., `datasciencepipelines` to `aipipelines`, removal of `modelmeshserving` and `codeflare`) are handled by the conversion webhook. Administrators do not need to manually recreate the DSC.

***Kueue \- after upgrade***

*After upgrading to OpenShift AI 3.5, verify that the Kueue component is properly configured and operational. If the Kueue migration to Red Hat build of Kueue was not completed before the upgrade, you must recover by completing the migration steps.*

***Prerequisites***

* *You have cluster administrator access to the OpenShift cluster.*  
* *You have upgraded to OpenShift AI 3.5.*

***Procedure***

1. *Check that the Kueue component is in a ready state as follows:*

   ```bash
   read STATUS REASON < <(oc get datasciencecluster -A -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status} {.items[0].status.conditions[?(@.type=="KueueReady")].reason}'); [[ "$STATUS" == "True" || ( "$STATUS" == "False" && "$REASON" == "Removed" ) ]] && echo "Ready" || echo "Not Ready"
   ```

2. *If the output is Ready, you can skip all of the remaining steps.*

   *If the output of the preceding command is Not Ready, check the Kueue component status as follows:*  
   ```bash
   oc get datasciencecluster -A -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status}{"\n"}{.items[0].status.conditions[?(@.type=="KueueReady")].message}{"\n"}'
   ```  
     
   *The following output indicates that Kueue was not migrated to Red Hat build of Kueue and that migration is required:*  
   *False*  
   *Kueue managementState Managed is not supported, please use Removed or Unmanaged*  
     
   *The following output indicates that you have a non-Kueue managed environment and that migration is not required:*  
   *False*  
   *Component ManagementState is set to Removed*  
3. *If migration to to Red Hat build of Kueue is required, you must follow the steps in Kueue \- Before upgrade.*

   ***Important***  
   *Pay close attention to all steps and caveats in the migration procedure, particularly the framework configuration annotation if you are using the default Kueue configuration.*

## **4.2. AI hub registry and catalog \- After upgrade** {#4.2.-ai-hub-registry-and-catalog---after-upgrade}

If any model registries or custom model catalogs were created before upgrade, you must ensure that there are no errors in the model registry and model catalog pods in OpenShift.

You must also ensure that all model registries or custom catalog sources created before upgrade are displayed correctly in the OpenShift AI dashboard.

**Note**

In OpenShift AI version 3.x, the dashboard navigation changed from **Models \> model registry** and **model catalog** to **AI hub \> Registry** and **Catalog**. In OpenShift, the number of model catalog pods changed from one to two.

**Prerequisites**

* You have OpenShift AI administrator access to manage model registry instances.

* You have the OpenShift **oc** command line interface installed.

* You have the appropriate permissions in OpenShift to access the required resources.

**Procedure**

1. In the OpenShift web console, click **Workloads \> Pods**.

2. In the **Project** field, enter **rhoai-model-registries** and check that all the pods have a status of **Running**.

   You can also get more information on a specific pod if needed by using the following commands:  
   ```bash
   oc logs <my-model-catalog-pod-name> -n rhoai-model-registries -c catalog
   oc logs <my-model-registry-pod-name> -n rhoai-model-registries -c <my-container-name>
   ```

3. In the OpenShift AI dashboard, click **Settings \> Model resources and operations \> AI registry settings** to check the status of your model registries. For more information, see [Managing model registries](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/managing_model_registries).

4. Click **AI hub \> Catalog** to check the default catalog and any custom catalogs that were created. For more information, see [Working with the model catalog](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/working_with_the_model_catalog).

**Verification**

1. In OpenShift, there are no errors in the following pods:

   * **\<my-model-registry\>-xxx**

   * **db-\<my-model-registry\>-xxx**

   * **model-catalog-xxx**

   * **model-catalog-postgres-xxx**

2. In the OpenShift AI dashboard, on the **Settings \> Model resources and operations \> AI registry settings** page, your model registries are displayed with **Available** status.

3. On the **AI Hub \> Model registry** page, your model registries are displayed correctly.

4. On the **AI Hub \> Catalog** page, your custom models are displayed correctly.

**Additional resources**

* [OpenShift documentation on working with pods](https://docs.redhat.com/en/documentation/openshift_container_platform/latest/html/nodes/working-with-pods)

##       **4.3. Feature Store \- After upgrade** {#4.3.-feature-store---after-upgrade}

In Red Hat OpenShift AI 2.25, the Feature Store component is a Technology Preview feature. In OpenShift AI 3.5, it is a GA feature. Otherwise, the Feature Store component is unchanged between Red Hat OpenShift AI 2.25.9 (and later) and 3.5.

**Note**  If you did not use the Feature Store component in OpenShift AI 2.25, skip this section. You do not need to perform any steps for Feature Store after you upgrade to 3.5.

If you used the Feature Store component in OpenShift AI 2.25, follow the steps in this procedure to verify that Feature Store is working correctly after you upgrade to OpenShift AI 3.5.

**Note**  The URL for the OpenShift AI 3.5 dashboard uses Gateway API access and is different from the 2.25.9 (and later) URL. The 2.25.9 (and later) dashboard URL is no longer accessible. If you have bookmarked the OpenShift AI dashboard URL, you must update the bookmark to point to the 3.5 URL.

**Prerequisites**

* You created a Feature Store Custom Resource Definition (CRD) in OpenShift AI 2.25.

* You have OpenShift AI administrator access for the procedure steps and OpenShift AI user access for the verification steps.

* You upgraded OpenShift AI 2.25.9 (and later) to 3.5.

**Procedure**

1. As an OpenShift AI administrator, run the following command in a terminal to check that the Feature Store operator pod (**feast-operator-controller-manager**) is in the **Running** state:  
   ```bash
   oc get pods -n redhat-ods-applications | grep feast-operator
   ```

   Example output:  

   ```bash
   feast-operator-controller-manager-89b9dc4b-lmhsc        1/1     Running
   ```

2. As an OpenShift AI administrator, get a list of all Feature Store instances on the cluster and verify that each Feature Store instance is in the **Ready** state:

   ```bash
   oc get featurestores --all-namespaces
   ```

   ```bash
   Example output:  
   NAMESPACE        NAME              STATUS      AGE

   project-alpha    my-featurestore   Ready       5d

   project-beta     demo-store        Ready       2d     

   project-gamma    prod-store        Ready       10d
   ```

3. As an OpenShift AI administrator, follow these steps for each Feature Store instance:

1. List CronJobs for the namespace that has a Feature Store instance by running the following command and replacing **\<namespace\>** with the name of the namespace:  
   ```bash
   oc get cronjobs -n <namespace>
   ```

   For example:

   ```bash
   oc get cronjobs -n project-alpha
   ```

   Example output:

   ```
   NAME              SCHEDULE   TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
   feast-sample-git  @yearly    <none>     True      0        <none>          74m
   ```

2. Create a Job by running the following command. Replace **\<job-name\>** with the name of the job and replace **\<cronjob-name\>** with the name of a CronJob output from the previous step:  
   ```bash
   oc create job <job-name> --from=cronjob/<cronjob-name> -n <namespace>
   ```

   For example:

   ```bash
   oc create job postupgradetest --from=cronjob/feast-sample-git -n project-alpha
   ```

   Example output:

   ```
   job.batch/postupgradetest created
   ```

3. Check that the CronJob for the Feature Store instance ran the Job successfully. View a list of jobs and their status by running the following command:  
   ```bash
   oc get jobs -n <namespace>
   ```

   For example:

   ```bash
   oc get jobs -n project-alpha
   ```

   The output should indicate that the job is running or completed, as shown in the following   
   example:

   ```
   NAME             STATUS     COMPLETIONS   DURATION   AGE
   postupgradetest  Complete   1/1           40s        5m53s
   ```

**Verification**

1. As an OpenShift AI user, open the OpenShift AI 3.5 dashboard.  
   **Note**  
   The URL for the OpenShift AI dashboard has changed to use Gateway API access. The 2.25.9 (and later) URL is no longer accessible.

2. Select **Develop & train** → **Feature Store**.

3. For each Feature Store instance, verify that the Feature Store UI shows any features, entities, feature-views, data sources, and feature services that you configured in OpenShift AI 2.25.

##  **4.4. OGX (Open GenAI Stack, formerly Llama Stack) \- After upgrade** {#4.4.-llama-stack---after-upgrade}

**IMPORTANT** 

 If you are using a disconnected environment, you can skip this section.  
Support for Llama Stack in a disconnected environment is provided starting in OpenShift AI 3.0.

**Note**  
In OpenShift AI 3.5, the Llama Stack component has been renamed to **OGX (Open GenAI Stack)**. The **LlamaStackDistribution** custom resource is replaced by the **OGXServer** (v1beta1) custom resource.

Key differences between OpenShift AI versions 2.25.9 (and later) and 3.5:

* **Component rename:** Llama Stack is now **OGX (Open GenAI Stack)**. **LlamaStackDistribution** CRs are replaced by **OGXServer** CRs.

* **PostgreSQL requirement:** PostgreSQL versions 14 and above databases are now required and SQLite is no longer supported.

* **Embedding model requirement:** An embedding provider, for example **sentence-transformers**, must be explicitly enabled.

* **VectorDB API deprecation:** The Vector\_IO API has replaced the deprecated VectorDB API.

* **Llama Stack Client version:** OpenShift AI versions 3.5 uses **llama-stack-client** version 0.4.x, previously OpenShift AI versions 2.25.9 (and later) used **llama-stack-client** version 0.2.x.

* **Configuration format changes:** The **run.yaml** file has been renamed to the **config.yaml** files in application deployments.

**Prerequisites**

* You completed the "Llama Stack / OGX upgrade steps for cluster administrators" and the "Llama Stack upgrade steps for **LlamaStackDistribution** resource owners" (Section 2.5).

**Procedure**

* After you complete the Llama Stack before-upgrade steps and the upgrade process, you can create new **OGXServer** CRs and applications in OpenShift AI version 3.5 using the **llsd-backup.yaml** file as a reference. This file was created in "Llama Stack upgrade steps for **LlamaStackDistribution** resource owners". Refer to the [Deploying a Llama Stack server](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/working_with_llama_stack/deploying-llama-stack-server_rag) documentation for more information.

##  

## **4.5. AI Pipelines \- After upgrade** {#4.5.-ai-pipelines---after-upgrade}

After upgrading to OpenShift AI 3.5, confirm that the AI Pipelines platform is healthy and notify pipeline users so that they can validate their pipelines.

**Prerequisites**

* You have completed the upgrade to OpenShift AI 3.5.

* You have logged in to the OpenShift AI dashboard.  
  For instructions, see [Logging in to OpenShift AI](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/getting_started_with_red_hat_openshift_ai_self-managed/logging-in_get-started).

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

* You ran `rhai-cli migrate prepare --migration ai-pipelines.pre-upgrade-check` before upgrading, as described in [AI Pipelines - Before upgrade](#2.4.-ai-pipelines---before-upgrade).

###  **4.5.1. Administrator tasks** {#4.5.1.-administrator-tasks}

**Procedure**

1. Verify that the pre-upgrade state file exists:

   ```bash
   ls -la /tmp/rhoai-upgrade-backup/ai_pipelines/dspa_pre_upgrade_pods.json
   ```

   If the file does not exist, the `migrate prepare` step in [AI Pipelines - Before upgrade](#2.4.-ai-pipelines---before-upgrade) was not run before the upgrade. In this case, skip the automated comparison in step 2 and manually verify DSPA health:

   ```bash
   oc get dspa -A
   oc get pods -n <dspa-namespace> | grep ds-pipeline
   ```

   Confirm that all pipeline server pods are **Running** with all containers ready, then skip to step 3.

2. Run the AI Pipelines post-upgrade check action:

   ```bash
   rhai-cli migrate run --migration ai-pipelines.post-upgrade-check --target-version 3.5.0
   ```

   This command compares post-upgrade pod state against the baseline saved by `migrate prepare` during [AI Pipelines - Before upgrade](#2.4.-ai-pipelines---before-upgrade).

3. Confirm that the output indicates that all AI Pipelines server pods are healthy or in the same state as before the upgrade.

4. Notify pipeline users that the upgrade is complete.

###  **4.5.2. Pipeline user tasks** {#4.5.2.-pipeline-user-tasks}

**Procedure**

1. Import a pipeline.  
   Confirm that the pipeline appears:

   * On the **Pipeline definitions** page.

   * On the **Pipelines** tab on the project details page.  
     For information on how to install a pipeline, see [Importing a pipeline](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/working_with_ai_pipelines/managing-ai-pipelines_ai-pipelines#importing-a-pipeline_ai-pipelines).

2. Execute a pipeline run.  
   On the **Runs** page, confirm that the run appears and progresses through expected states such as **Pending**, **Running**, and **Succeeded**.  
   For information on how to execute a pipeline run, see [Executing a pipeline run](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/working_with_ai_pipelines/managing-pipeline-runs_ai-pipelines#executing-a-pipeline-run_ai-pipelines).

3. Verify scheduled pipeline runs.  
   Confirm that previously configured scheduled runs remain visible and enabled on the **Runs** page.  
   For more information, see [Viewing scheduled pipeline runs](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/working_with_ai_pipelines/managing-pipeline-runs_ai-pipelines#viewing-scheduled-pipeline-runs_ai-pipelines).

4. If any pipeline runs were in progress during the upgrade and failed, rerun those pipelines after confirming that the pipeline server is healthy.

**Verification**

* AI Pipelines server pods are healthy after the upgrade.

* Pipeline users can successfully run pipelines.

* Scheduled pipeline runs remain enabled.

## 

## 

## **4.6. TrustyAI \- After upgrade** {#4.6.-trustyai---after-upgrade}

After upgrading OpenShift AI to 3.5, verify the health and configuration of the TrustyAI component and perform the following post-upgrade tasks.

**Warning**

Complete all steps in the following procedures in the order presented. Do not proceed to the next step until the output matches the expected result.

* TrustyAI \- After upgrade \- Check Backups

* TrustyAI \- After upgrade \- Guardrails

* TrustyAI \- After upgrade \- Restore data

* TrustyAI \- After upgrade \- GPU deployment deadlock issue

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You have access to the **rhai-cli** tool.

###  **4.6.1. TrustyAI \- After upgrade \- Check Backups** {#4.6.1.-trustyai---after-upgrade---check-backups}

After upgrading OpenShift AI to 3.5, check the status of the TrustyAI component and the before-upgrade backups.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You have access to the **rhai-cli** tool.

**Procedure**

1. Verify that you set the backup directory:

   ```bash
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
   ```

3. Check that the TrustyAI Operator is healthy:

   ```bash
   oc wait --for=condition=Available deployment/trustyai-service-operator-controller-manager -n redhat-ods-applications --timeout=120s
   ```

   Expected output:

   ```
   deployment.apps/trustyai-service-operator-controller-manager condition met
   ```

   If the Operator is healthy, you should see the output shown in the preceding example and can proceed to the next step.  
   **Note**  
   If the command fails with a timeout error, inspect the Operator pod for more details:

   ```bash
   oc get pods -n redhat-ods-applications -l control-plane=controller-manager -o wide
   ```

4. List the namespaces for which you have backups:

   ```bash
   ls ${BACKUP_DIR}/trustyai-metrics-*.json 2>/dev/null \
     | sed 's|.*/trustyai-metrics-||;s|-[0-9]\{8\}-[0-9]\{6\}\.json||' \
     | sort -u
   ```

   The output should list one namespace per line, as shown in the following example:

   ```
   test-trustyaiservice-upgrade
   ```

   **Note**  
   If there is no output, there are no backups. Skip to TrustyAI \- After upgrade \- Guardrails.

5. For each namespace that has a backup, check whether data was lost:

   ```bash
   export NS=<namespace>
   export TAS_NAME=$(oc get trustyaiservice -n "$NS" -o jsonpath='{.items[0].metadata.name}')
   export SVC_PORT=$(oc get svc -n "$NS" "$TAS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
   oc port-forward -n "$NS" "svc/$TAS_NAME" 8080:${SVC_PORT} &
   export PF_PID=$!
   export CURRENT=$(curl -sk -H "Authorization: Bearer $(oc whoami -t)" \
     "http://localhost:8080/metrics/all/requests" | jq '.requests | length')
   export BACKUP=$(jq '.requests | length' "$(ls -t ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | head -1)")
   kill $PF_PID 2>/dev/null
   echo "Current: $CURRENT | Backup: $BACKUP"
   [ "$CURRENT" -ge "$BACKUP" ] && echo "OK: no data loss" || echo "DATA LOSS: restore needed"
   ```

If the result is OK, no data was lost and the service is running successfully. Repeat this step for each namespace that has a backup, then continue to [TrustyAI \- After upgrade \- Guardrails](#4.6.2.-trustyai---after-upgrade---guardrails).

If the result is DATA LOSS, see [TrustyAI \- After upgrade \- Restore data](#4.6.3.-trustyai---after-upgrade---restore-data).

### **4.6.2. TrustyAI \- After upgrade \- Guardrails** {#4.6.2.-trustyai---after-upgrade---guardrails}

After upgrading OpenShift AI to 3.5, check the status of the TrustyAI Guardrails Orchestrator service.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You have access to the **rhai-cli** tool.

**Procedure**

1. Get a list of the namespaces that contain the TrustyAI Guardrails Orchestrator service:

   ```bash
   oc get guardrailsorchestrator -A
   ```

   Example output:

   ```
   NAMESPACE                        NAME                      AGE
   redhat-ods-operator              guardrails-orchestrator   4h22m
   test-guardrails-builtin-upgrade  guardrails-orchestrator   4h50m
   test-guardrails-tempo-hf         guardrails-orchestrator   4h17m
   ```

   **Note**  
   If the result is **No resources found**, you can skip the remaining steps in this procedure.

2. Check whether **GuardrailsOrchestrator** deployments are missing the ReadinessProbe:

   ```bash
   rhai-cli migrate run --migration trustyai.patch-guardrails --target-version 3.5.0 --dry-run
   ```

   **Note**  
   The rhai-cli action operates cluster-wide across all namespaces.

   Example output:

   ```
   [INFO] Checking GuardrailsOrchestrator guardrails-orchestrator in namespace model-namespace
   [CHECK] OK deployment guardrails-orchestrator (readinessProbe already set)
   Check complete.
   ```

   If the result is **OK** for all namespaces that have the TrustyAI Guardrails Orchestrator service, skip to Step 3\.

   If the result is **NEEDS PATCH**, continue to the next step.

   1. Patch the **GuardrailsOrchestrator** deployments:

      ```bash
      rhai-cli migrate run --migration trustyai.patch-guardrails --target-version 3.5.0
      ```

      **Note**  
      The patch might take time to complete because it edits the existing deployments and forces them to restart rollout.

   2. Wait until the patch provides output.

      Example output:

      ```
      [INFO] Checking GuardrailsOrchestrator guardrails-orchestrator-otel in namespace model-namespace
      [INFO] Patching deployment guardrails-orchestrator-otel in namespace model-namespace
      deployment.apps/guardrails-orchestrator-otel patched

      [INFO] Waiting for rollout to complete...

      Waiting for deployment "guardrails-orchestrator-otel" rollout to finish: 1 old replicas are pending termination...
      Waiting for deployment "guardrails-orchestrator-otel" rollout to finish: 1 old replicas are pending termination...
      deployment "guardrails-orchestrator-otel" successfully rolled out

      Successfully patched deployment guardrails-orchestrator-otel

      ==========================================
      GuardrailsOrchestrator Deployment Patch Summary
      =========================================
      OK: guardrails-orchestrator-otel patched successfully!
      ```

3. Check whether **GuardrailsOrchestrator** CRs are exporting traces and metrics:

   ```bash
   rhai-cli migrate run --migration trustyai.migrate-gorch-otel-exporter --target-version 3.5.0 --dry-run
   ```

   **Note**  
   The rhai-cli action operates cluster-wide across all namespaces.

   **Note**  
   You can safely ignore the following warning if it appears:  
   `WARNING: migration trustyai.migrate-gorch-otel-exporter has phase pre-upgrade but effective phase is post-upgrade`  
   This is a known phase registration mismatch in rhai-cli. The migration runs correctly when invoked with `--migration`.

   Example output:

   ```
   Checking for GuardrailsOrchestrator CRs in namespace model-namespace
   Found 2 GuardrailsOrchestrator CR(s) in namespace model-namespace

   guardrails-orchestrator: already on new otelExporter schema
   guardrails-orchestrator-otel: already on new otelExporter schema
   ```

   If all **GuardrailsOrchestrator** CRs report as **already on new otelExporter schema**, skip to step 5\.  
   Otherwise, continue to the next step.

4. Run the migration that patches the existing **GuardrailsOrchestrator** deployments by updating the keys under **spec.otelExporter**:

   ```bash
   rhai-cli migrate run --migration trustyai.migrate-gorch-otel-exporter --target-version 3.5.0
   ```

5. Query the **/info** endpoint of the **GuardrailsOrchestrator** service:

   ```bash
   export GORCH_NAME=<gorch-name>
   export GORCH_ROUTE_HEALTH=$(oc get routes -n $NS "${GORCH_NAME}-health" -o jsonpath='{.spec.host}')
   curl -sSk "https://${GORCH_ROUTE_HEALTH}/info" -H "Authorization: Bearer $(oc whoami -t)" | jq .
   ```

**Verification**

* All services should have a HEALTHY status, as shown in the following example output:

  ```
  {
    "services": {
      "<detector_1>": {
        "status": "HEALTHY"
      },
      "<detector_2>": {
        "status": "HEALTHY"
      },
      "chat_generation": {
        "status": "HEALTHY"
      }
    }
  }
  ```

### **4.6.3. TrustyAI \- After upgrade \- Restore data** {#4.6.3.-trustyai---after-upgrade---restore-data}

If you ran the TrustyAI \- After upgrade \- Check Backups procedure for a namespace and the result was **DATA LOSS**, you can restore the data if you backed it up before upgrading to OpenShift AI 3.5.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You have access to the **rhai-cli** tool.

* You backed up data as described in TrustyAI \- Before upgrade.

* You upgraded to OpenShift AI 3.5

* You completed the steps in TrustyAI \- After upgrade \- Check Backups.

**Procedure**

Follow these steps for each namespace that lost data:

1. Set the backup directory:

   ```bash
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
   ```

2. Verify that you have a backup for the namespace:

   ```bash
   ls ${BACKUP_DIR}/trustyai-metrics-*.json 2>/dev/null \
     | sed 's|.*/trustyai-metrics-||;s|-[0-9]\{8\}-[0-9]\{6\}\.json||' \
     | sort -u
   ```

   The output prints one namespace per line, as shown in the following example output:

   ```
   test-trustyaiservice-upgrade
   ```

   **Note**  
   If the namespace that lost data is not listed, you cannot complete this procedure because there is no backup.

3. Run the following command to set the namespace, by replacing \<namespace\> with a namespace that lost data:

   ```bash
   export NS=<namespace>
   ```

4. Get the TrustyAIService name:

   ```bash
   export TAS_NAME=$(oc get trustyaiservice -n "$NS" -o jsonpath='{.items[0].metadata.name}')
   echo "TAS_NAME=$TAS_NAME"
   ```

   The command should print the TrustyAIService name, as shown in the following example output:

   ```
   TAS_NAME=trustyai-service
   ```

   **Note**  
   If the output is empty, the TrustyAIService no longer exists in this namespace. Restart the steps in this procedure for the next namespace that lost data, if any.

5. Verify that TrustyAIService is running:

   ```bash
   oc get trustyaiservice -n "$NS" "$TAS_NAME" -o jsonpath='{.status.phase}'
   ```

   Example output:

   ```
   Ready
   ```

   If the output is **Ready**, skip to Step 5\.  
   If the output is other than **Ready**:

6. Run the following command:

   ```bash
   oc wait --for=jsonpath='{.status.phase}'=Ready trustyaiservice/"$TAS_NAME" -n "$NS" --timeout=300s
   ```

   Example output:

   ```
   trustyaiservice.trustyai.opendatahub.io/trustyai-service condition met
   ```

   If the wait times out, the TrustyAIService might not be healthy. Check its status:

   ```bash
   oc describe trustyaiservice "$TAS_NAME" -n "$NS"
   ```

7. Find the backup file for this namespace:

   ```bash
   ls -t ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | head -1
   ```

   If the output provides a file path, continue to the next step.  
   If no file is found, there is no backup for this namespace. Restart the steps in this procedure for the next namespace that lost data, if any.

8. Set the backup file path:

   ```bash
   export BACKUP_FILE=$(ls -t ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | head -1)
   echo "BACKUP_FILE=$BACKUP_FILE"
   ```

   The command should print the backup file path, as shown in the following example output:

   ```
   BACKUP_FILE=/tmp/rhoai-upgrade-backup/trustyai/trustyai-metrics-test-trustyaiservice-upgrade-20260218-175450.json
   ```

9. Get the number of metrics that are in the backup:

   ```bash
   jq '.requests | length' "$BACKUP_FILE"
   ```

   Example output:

   ```
   1
   ```

   The result should be a number greater than 0\. Continue to the next step.  
   If the result is 0, there are no metrics to restore. Restart the steps in this procedure for the next namespace that lost data, if any.

10. Find the TrustyAI route and its label:

    ```bash
    export ROUTE_LABEL=$(oc get route -n "$NS" -o json | jq -r --arg tas "$TAS_NAME" '
        .items[] | select(.spec.to.name==$tas)
        | (.metadata.labels // {}) as $l
        | if $l["trustyai-service-name"] then "trustyai-service-name=\($l["trustyai-service-name"])"
          elif $l["app.kubernetes.io/name"] then "app.kubernetes.io/name=\($l["app.kubernetes.io/name"])"
          elif $l["app"] then "app=\($l["app"])"
          else empty end
      ' | head -1)
    echo "ROUTE_LABEL=$ROUTE_LABEL"
    ```

    The output should be a label selector, as shown in the following example. Continue to Step 9\.

    ```
    trustyai-service-name=trustyai-service
    ```

    If the output is empty (ROUTE\_LABEL=), find and set the route label manually.

    

11. Find the route label:

    ```bash
    oc get route -n "$NS" --show-labels
    ```

    Example output:

    ```
    NAME                    HOST/PORT                                                                                                PATH   SERVICES                          PORT          TERMINATION          WILDCARD   LABELS
    gaussian-credit-model   gaussian-credit-model-test-trustyaiservice-upgrade.<...>.openshiftapps.com          gaussian-credit-model-predictor   https         reencrypt/Redirect   None       inferenceservice-name=gaussian-credit-model
    trustyai-service        trustyai-service-test-trustyaiservice-upgrade.<...>.openshiftapps.com               trustyai-service-tls              oauth-proxy   reencrypt/Redirect   None       trustyai-service-name=trustyai-service
    ```

12. Set the route label by replacing **\<label\_key\>=\<label\_value\>:** with your **TrustyAI service label pair:**

    ```bash
    export ROUTE_LABEL='<label_key>=<label_value>'
    ```

    For example:

    ```bash
    export ROUTE_LABEL=trustyai-service-name=trustyai-service
    ```

    IMPORTANT: Ensure the route label that you specify belongs to the trustyai-service and is not a route that belongs to any other services or models.

    

13. Dry-run the restore:

    ```bash
    rhai-cli migrate run --migration trustyai.metrics --target-version 3.5.0 --dry-run
    ```

    Example output:

    ```
    [INFO] Starting TrustyAI metrics restore...
    [INFO] Namespace: test-trustyaiservice-upgrade
    [INFO] Backup file: backups/trustyai-metrics-test-trustyaiservice-upgrade-20260218-175450.json
    [WARN] DRY RUN MODE - No changes will be made
    [INFO] Validating backup file...
    [INFO] Found 1 metric(s) to restore
    [INFO] Checking cluster connectivity...
    [INFO] Fetching TrustyAI service route...
    [INFO] TrustyAI route: trustyai-service-test-trustyaiservice-upgrade.apps.rosa.trustyai-scyril.w4n4.p3.openshiftapps.com
    [INFO] Retrieving authentication token...
    [INFO] Checking TrustyAI service health...
    [INFO] Processing metrics...

    [INFO] Processing: MEANSHIFT for model gaussian-credit-model (original ID: 5166c098-d2f9-4285-8303-7879a645ac26)
    [INFO]   [DRY RUN] Would POST to: https://trustyai-service-test-trustyaiservice-upgrade.apps.rosa.trustyai-scyril.w4n4.p3.openshiftapps.com/metrics/drift/meanshift/request
    [DEBUG]   [DRY RUN] Payload: {"@type":"MeanshiftMetricRequest","modelId":"gaussian-credit-model","requestName":null,"metricName":"MEANSHIFT","batchSize":5000,"thresholdDelta":0.05,"referenceTag":"TRAINING","fitColumns":["credit_inputs-2","credit_inputs-3","predict-0","credit_inputs-0","credit_inputs-1"],"fitting":{"credit_inputs-2":{"mean":12.032881584334207,"variance":3.9188251284489697,"n":1000,"max":0.0,"min":0.0,"sum":0.0,"standardDeviation":1.9796022652161644},"credit_inputs-3":{"mean":19.844397164842988,"variance":24.4671876962395,"n":1000,"max":0.0,"min":0.0,"sum":0.0,"standardDeviation":4.946431814574977},"predict-0":{"mean":0.20297420065215557,"variance":0.014778104214330515,"n":1000,"max":0.0,"min":0.0,"sum":0.0,"standardDeviation":0.12156522617233316},"credit_inputs-0":{"mean":44.919118954557185,"variance":24.541570407096373,"n":1000,"max":0.0,"min":0.0,"sum":0.0,"standardDeviation":4.953944933797344},"credit_inputs-1":{"mean":502.487550504219,"variance":2620.225933216063,"n":1000,"max":0.0,"min":0.0,"sum":0.0,"standardDeviation":51.18814250601464}}}

    [INFO] ==========================================
    [INFO] Restoration Summary
    [INFO] ==========================================
    [INFO] Total metrics in backup: 1
    [INFO] Successfully restored:   1
    [INFO] Failed:                  0
    [INFO] Skipped:                 0
    [INFO] ==========================================
    [INFO] DRY RUN completed - no changes were made
    ```

    The output lists each metric that the script would restore.

    **NOTE:** if the TrustyAI Service reports an **"UNKNOWN"** status, check to make sure that you selected the correct route label in step 12\.

     If any metric has an **Unknown** metric type, it might not be supported in this version.  
      
14. Run the restore:

    ```bash
    rhai-cli migrate run --migration trustyai.metrics --target-version 3.5.0
    ```

**Verification**

* Verify that each metric shows **Successfully scheduled**. The summary at the end should show **Failed: 0**.

  If any metric fails, re-run the migration to skip already-restored metrics.

  Check the output for the HTTP code and response for each failure.

  Examples of common failures:

  Route not found: Double-check ROUTE\_LABEL from step 7 in the procedure.  
  HTTP 400: Request body format may have changed between versions.

  HTTP 500: Model data may not be loaded yet. Check with:

  ```bash
  curl -sk "https://$(oc get route -n "$NS" -l "$ROUTE_LABEL" -o jsonpath='{.items[0].spec.host}')/info" -H "Authorization: Bearer $(oc whoami -t)" | jq .
  ```

* Verify that the restore count matches the backup:

  ```bash
  rhai-cli migrate run --migration trustyai.metrics --target-version 3.5.0 --dry-run 2>&1 | tail -5
  ```

  The **Current scheduled metrics** count should be greater than or equal to the backup count from Step 8 in the procedure.

  **Note**  
  Restored metrics receive new UUIDs; original IDs from the backup are not preserved.

### 

### **4.6.4. TrustyAI \- After upgrade \- GPU deployment deadlock issue** {#4.6.4.-trustyai---after-upgrade---gpu-deployment-deadlock-issue}

If you deploy GPU-based **InferenceServices** in a namespace that has a running TrustyAI service, a deployment deadlock can occur on clusters with GPU nodes.

The symptom of this specific deployment deadlock issue is a new pod for the GPU-based **InferenceService** staying in a pending state indefinitely while the old pod keeps running.

You can investigate and resolve the GPU deadlock.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You have access to the **rhai-cli** tool.

**Procedure**

1. To determine the namespace where a potential GPU deadlock occurs:

   ```bash
   oc get pods -A | grep predictor
   ```

   Example output:

   ```
   redhat-ods-operator               guardrails-detector-ibm-hap-predictor-55cbd5bb59-5x9r9   1/1     Running     0       117m
   redhat-ods-operator               phi3-predictor-5d49548868-dsngq                          1/1     Running     0       8h
   test-guardrails-builtin-upgrade   llm-d-inference-sim-isvc-predictor-6469d9b47b-cwssb      2/2     Running     0       117m
   test-guardrails-tempo-hf          hap-detector-predictor-76497f8f99-lf42p                  2/2     Running
   test-guardrails-tempo-hf          llm-d-inference-sim-isvc-predictor-78b7c58d94-ldt5k      0/2     Pending     0       117m
   test-guardrails-tempo-hf          prompt-injection-detector-predictor-7b7bfbcfc4-9jpns     2/2     Running     0       117m
   test-trustyaiservice-upgrade      gaussian-credit-model-predictor-b74468d95-lndh4          3/3     Running     0       8h
   ```

   In an impacted namespace, the expected output is **Pending**, as shown in the following example:

   ```
   test-guardrails-tempo-hf          llm-d-inference-sim-isvc-predictor-78b7c58d94-ldt5k      0/2     Pending     0       117m
   ```

2. To check if the **Pending** state is caused by a GPU deadlock issue, run the deadlock check for the impacted namespace:

   1. Check for GPU deadlocks across the cluster:

      ```bash
      rhai-cli migrate run --migration trustyai.break-gpu-deadlock --target-version 3.5.0 --dry-run
      ```

      **Note**  
      The rhai-cli action operates cluster-wide across all namespaces.

      Example output if there is no deadlock:

      ```
      No deadlocks detected
      ```

      Example output if there is a deadlock:

      ```
      DEADLOCK: llm
        Running: hap-detector-predictor-76497f8f99-lf42p
        Pending: llm-d-inference-sim-isvc-predictor-78b7c58d94-ldt5k
        Running: prompt-injection-detector-predictor-7b7bfbcfc4-9jpns
      ```

   2. To fix the deadlocked GPU-based inference services, you can manually delete an older pod or you can run the migration:

      ```bash
      rhai-cli migrate run --migration trustyai.break-gpu-deadlock --target-version 3.5.0
      ```

      **Note**  
      The migration might take time to run as it deletes the older pod and waits for a new pod to run successfully. After it completes, you should see one instance of the **InferenceService**.

   3. Get the pod list:

      ```bash
      oc get pods -n $NS | grep predictor
      ```

      Example output:

      ```
      hap-detector-predictor-76497f8f99-lf42p                2/2     Running   0          125m
      llm-d-inference-sim-isvc-predictor-78b7c58d94-ldt5k    2/2     Running   0          125m
      prompt-injection-detector-predictor-7b7bfbcfc4-9jpns   2/2     Running   0          125m
      ```

      Note that the pod is no longer pending.

## 

## 

## 
## **4.7. Workbenches \- After upgrade** {#4.7.-workbenches---after-upgrade}

### **4.7.1. Migrate your workbenches after upgrade** {#4.7.1.-migrate-your-workbenches-after-upgrade}

**Important**

All procedures in this section must be completed after upgrading the Red Hat OpenShift AI Operator from version 2.25.9 (and later) to 3.5. Failure to complete these steps can result in service disruptions for your workbenches.

**Prerequisites**

* You have OpenShift AI administrator access to manage workbenches.

* Your OpenShift AI Operator is version 3.5.

* Your **notebook-controller** component is updated and running.

**Procedure**

1. Verify that the ODH **Notebook Controller** components are updated and running by using the following command:

   ```bash
   oc get Deployment --namespace redhat-ods-applications odh-notebook-controller-manager notebook-controller-deployment
   ```

   **Example output**

   ```
   NAME                       READY   UP-TO-DATE   AVAILABLE  AGE
   odh-notebook-controller-manager   1/1     1       1      2m
   notebook-controller-deployment    1/1     1       1      2m
   ```

2. Verify which workbenches are ready to be upgraded with your users, then execute the following command to patch the stopped workbenches:


   ```bash
   rhai-cli migrate run --migration workbenches.patch-auth-model --target-version 3.5.0 --only-stopped --with-cleanup
   ```

   **Note**
   You can safely ignore the following warning if it appears:  
   `WARNING: migration workbenches.patch-auth-model has phase pre-upgrade but effective phase is post-upgrade`  
   This is a known phase registration mismatch in rhai-cli. The migration runs correctly when invoked with `--migration`.

   **Note**

   This command prompts for interactive confirmation twice: once before patching notebooks, and once before cleaning up legacy OAuth resources. Enter **y** at each prompt to proceed.

   As the command runs, it provides output that indicates the status of the patch process. When the command completes, you should see a messages similar to the following:  
* **Processed 9 workbenches: all succeeded.**  
*  **Cleanup: all 9 workbenches completed successfully.**

3. Notify your users that any workbenches that were stopped in order to prepare for the upgrade can now be started again.

**Verification**

1. Verify that the workbench upgrade was successful by running the following command:


   ```bash
   rhai-cli migrate run --migration workbenches.verify-migration --target-version 3.5.0
   ```

   As the command runs, it provides output that indicates the status of your workbench upgrade. When the command completes, you should see a message similar to the following:  
    **OK: All workbenches have been migrated.**

2. Confirm with your OpenShift AI users that they can access their workbench IDE from a web browser.

### **4.7.2. Perform a deferred workbench image migration** {#4.7.2.-perform-a-deferred-workbench-image-migration}

If you are unable to stop some or all of your workbenches during the Red Hat OpenShift AI upgrade from version 2.25.9 (and later) to 3.5, your OpenShift AI users can defer their workbench image migration until after the upgrade.

**Important**

* Workbench images left unmigrated continue to operate on the older 2.25.9 (and later) authentication layer. This hybrid environment can result in redirection loops and connectivity failures, primarily due to **NB\_PREFIX** routing conflicts for RStudio, code-server, and custom images.

After the upgrade to Red Hat OpenShift AI version 3.5 is complete, instruct your users to migrate their workbench images by following these steps:

**Prerequisites**

* You have OpenShift AI user access.

* Your OpenShift AI Operator version is at the latest patch release.

* You have taken appropriate measures to prevent the loss of unsaved work.

**Procedure**

1. Evaluate if your image version tag needs to be updated:

   * If you are using OpenShift AI Jupyter-based workbenches, it is *recommended* to update your workbench image tags to the latest version (2025.2).

   * If you are using OpenShift AI code-server workbenches, it is **required** to update your workbench image tags to the latest version (2025.2).

   * If you are using the OpenShift AI RStudio buildconfig, it is **required** to be on the latest version tag, titled “latest”. The tag version will update after it is rebuilt.

     **Note** If you are using the OpenShift AI RStudio buildconfig, you will need a new build after your workbench upgrade.

   * If you are using custom workbench images, you must migrate your workbench images to the Kubernetes Gateway API to leverage path-based routing and maintain compatibility with Red Hat OpenShift AI version 3.5. See [Introducing the Kubernetes Gateway API for custom image migration](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_resources/introducing-kubernetes-gateway-api_resource-mgmt) for more information.

     **Note**  Once you have rebuilt an image with the Kubernetes Gateway API support to enable path-based routing, publish it with a new tag and import it into your ImageStream. It is **required** to be on this latest published version tag. See [Importing a custom workbench image](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_openshift_ai/creating-custom-workbench-images#importing-a-custom-workbench-image_custom-images) for more information.

2. Upgrade your workbench images in one of the following ways:

   * Edit the workbench description in the Dashboard and save your work. See [Updating a project workbench](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.2/html/working_on_projects/using-project-workbenches_projects#updating-a-project-workbench_projects) for more information.  
     **Note** The dashboard patches the workbenches automatically and requires no intervention.

   * Delete and then recreate the workbench using the dashboard UI or OpenShift CLI (**oc**). See [Deleting a workbench from a project](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.2/html/working_on_projects/using-project-workbenches_projects#deleting-a-workbench-from-a-project_projects) and [Creating a workbench](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.2/html/working_on_projects/using-project-workbenches_projects#creating-a-project-workbench_projects) for more information.

     **Important**  
     Use the same persistent volume claim (PVC) to prevent data loss.

**Verification**

* Ensure your workbench enters a **RUNNING** state.

## 

## **4.8. Ray Training Operator \- After upgrade** {#4.8.-ray-training-operator---after-upgrade}

After upgrading from Red Hat OpenShift AI 2.25.9 (and later) to 3.5, use the **rhai-cli** tool to migrate your Ray clusters.

**Note**

The commands in the following procedure include an optional **\--dry-run** argument. Include this argument to preview how the command functions.

**Prerequisites**

* **WARNING:** You must follow the After upgrade steps for the Workbench component before you migrate your Ray clusters.

* You have cluster administrator access.

* You have access to the **rhai-cli** tool.

**Procedure**

1. List the Ray clusters on your OpenShift cluster:

   ```bash
   rhai-cli migrate list --target-version 3.5.0
   ```

   The status of all Ray clusters is **NEEDS MIGRATION** as shown in the following example output:

   ```
   Fetching RayClusters (all namespaces)...
   Found 2 RayCluster(s)
   Analyzing clusters...
   Analysis complete.

   RayCluster Migration Status:

   Name                 Namespace    Status    Workers  Migration Status
   ----------------------------------------------------------------------
   comprehensive-mixed  raytest      ready     2        [NEEDS MIGRATION]
   sdk-configurations   raytest      ready     1        [NEEDS MIGRATION]
   Migration Summary: 0 migrated, 2 need migration
   ```
2. Choose from the following options to migrate your Ray clusters:

* **Migrate a single Ray cluster**  

  ```bash
  rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --raycluster-cluster my-cluster --raycluster-namespace my-namespace [[--dry-run]]
  ```

  Example output:

  ```
  Analyzing 1 RayCluster(s) (cluster 'my-cluster' in namespace 'my-namespace')

  [MIGRATE] my-cluster (ns: my-namespace) - needs migration


  Summary: 1 to migrate, 0 already migrated


  The following 1 cluster(s) will be migrated:

    - my-cluster (namespace: my-namespace)


  IMPORTANT: Migration will cause temporary downtime for each Ray cluster

  as the cluster pods are restarted with the updated configuration.


  Proceed with migration? (yes/no): yes


    [OK] Migrated: my-cluster (ns: my-namespace)


  ============================================================

  Migration Summary:

    Migrated: 1

    Skipped (already migrated): 0

    Failed: 0


  ============================================================

  RayCluster Routes (Gateway API):

  Note: Routes may take a moment to become available after migration.

  ------------------------------------------------------------

    my-cluster (ns: my-namespace)

    Dashboard: https://rh-ai.apps.example.com/ray/cluster
  ```

* **Migrate all Ray clusters in a namespace**  

  ```bash
  rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --raycluster-namespace my-namespace [[--dry-run]]
  ```

* **Migrate all Ray clusters on your OpenShift cluster**  

  ```bash
  rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 [[--dry-run]]
  ```

3. Make a note of the Dashboard URL that is output by the migration command. Share this URL with users that want to access the Ray cluster.

**Verification**

1. In a web browser, verify that the Dashboard URL output by the migration command provides access to the Ray cluster.

2. List the migration status of all Ray clusters:

   ```bash
   rhai-cli migrate list --target-version 3.5.0
   ```

   Example output:

   ```
   Found 3 RayCluster(s):

   Name                Namespace    Status  Workers  Migration Status

   -------------------------------------------------------------------

   production-cluster  production   ready     5      [MIGRATED]

   staging-cluster     staging      ready     3      [NEEDS MIGRATION]

   dev-cluster         dev          ready     2      [NEEDS MIGRATION]



   Migration Summary: 1 migrated, 2 need migration
   ```

   If any of the listed Ray clusters have the **NEEDS MIGRATION** status, run the **raycluster.migrate** migration command for that Ray cluster.

**Troubleshooting**

Use the following information to help troubleshoot problems with Ray clusters after you upgrade from Red Hat OpenShift AI 2.25.9 (and later) to 3.5.

* **You did not run the Ray cluster pre-upgrade migration command**  
  If you did not run the pre-upgrade migration before you upgraded to 3.5, you can run it after upgrading to 3.5 by following the procedure described in *Ray Training Operator \- Before upgrade*.

* **You ran the post-upgrade migration command and its output indicates a Ray cluster failure**  
  If a Ray cluster fails to be ready, the command identifies the Ray cluster with **\[FAIL\]** status, as shown in the following example:

  ```bash
  [OK] Migrated: cluster-a (ns: my-ns)

       Dashboard: https://cluster-a-my-ns.apps.example.com
  [OK] Migrated: cluster-b (ns: my-ns)
       Dashboard: https://cluster-b-my-ns.apps.example.com
  [FAIL] cluster-c (ns: my-ns) - did not become ready within 5 minutes
       Please check this cluster (and any others that timed out) and
       revisit as needed.


  ============================================================
  Migration Summary:
    Migrated: 2
    Skipped (already migrated): 1
    Failed: 1
    Failed clusters include those that did not become ready
    within 5 min — please verify status and revisit as needed.
  ```

Resolve any issues and then run the migration command again. Optionally, you can run the command with the **\--raycluster-from-backup $BACKUP\_DIR/rhoai-3.x** argument.

* **You migrated a Ray cluster and cannot access it from your bookmarked URL**  
  Instead of the bookmarked URL, use the URL provided in the output of the **raycluster.migrate** migration command.

  During the migration process, the Ray cluster URL changes to use Gateway API access. The 2.25.9 (and later) route URL is no longer accessible for the Ray cluster. When you run the **raycluster.migrate** migration command, it outputs the new URL to access the migrated Ray cluster. If you did not make a note of the new URL, use the CodeFlare SDK inside a workbench to query all your Ray clusters:  

  ```python
  from codeflare_sdk import list_all_clusters
  clusters = list_all_clusters("my-namespace")
  ```

  Example output:

  ```
  │            CodeFlare Cluster Details                      │                                                                  │

  │ Name: my-cluster                       Active

  │

  │ URI: ray://my-cluster-head-svc.ns.svc:10001                      │
  │ Dashboard: https://rh-ai.apps.example.com/ray/cluster    │
  ```

* **Your Ray cluster is in an unrecoverable state**  
  Your Ray cluster is in an unrecoverable state if, for example, it does not reach *ready* status or the dashboard URL output by the 3.5 migration command does not work.  
  If your Ray cluster is an unrecoverable state:

  1. If you have not already run the **pre-upgrade** migration command to create back ups of your Ray cluster Custom Resource (CR) files, follow the procedure described in *Ray Training Operator \- Before upgrade*. You can do this step before or after upgrading from OpenShift AI 2.25.9 (and later) to 3\.

  2. Run the migration command with the **\--raycluster-from-backup $BACKUP\_DIR/rhoai-3.x** argument.

     * Recover a single Ray cluster from backup (**\--dry-run** optional)  

       ```bash
       rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --raycluster-from-backup $BACKUP_DIR/rhoai-3.x --raycluster-cluster my-cluster --raycluster-namespace my-namespace [[--dry-run]]
       ```

     * Recover all Ray clusters from one namespace from backup (**\--dry-run** optional)  

       ```bash
       rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --raycluster-from-backup $BACKUP_DIR/rhoai-3.x --raycluster-namespace my-namespace [[--dry-run]]
       ```

     * Recover all Ray clusters on the OpenShift cluster from backup (**\--dry-run** optional)  

       ```bash
       rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --raycluster-from-backup $BACKUP_DIR/rhoai-3.x [[--dry-run]]
       ```

       The **\--raycluster-from-backup $BACKUP\_DIR/rhoai-3.x** argument instructs the command to delete the Ray clusters from your OpenShift cluster and then recreate them from the files in the **rhoai-3.x** backup directory.

       For example:

       ```bash
       rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --raycluster-from-backup $BACKUP_DIR/rhoai-3.x
       ```

     * Example output:

       ```
       Found 3 RayCluster(s) in backup to migrate (all namespaces):

         - ray2 (ns: ray2) from raycluster-ray2-ray2.yaml

         - ray1 (ns: ray1) from raycluster-ray1-ray1.yaml

         - ray3 (ns: ray3) from raycluster-ray3-ray3.yaml



       WARNING: Restore from backup will DELETE and RECREATE each RayCluster.

         - If a cluster currently exists, it will be deleted first.

         - All running pods, jobs, and workloads will be terminated.

         - Existing job state and logs will be lost.

         - The cluster will be recreated from the backup configuration.



       Proceed with restore from backup? (yes/no): yes



         [ray2] Deleting existing cluster...

         [ray2] Waiting for cluster deletion to complete...

         [ray2] Cluster deleted successfully

         [ray2] Creating cluster from backup...

         [ray2] Waiting for cluster to become ready...

           [DEBUG] Cluster-wide search found 1 items

           [DEBUG] Found Gateway hostname from Route:

           data-science-gateway.apps.ray-ppaszki.7ctq.s1.devshift.org

         [OK] Restored from backup: ray2 (ns: ray2)

              Dashboard:

              https://data-science-gateway.apps.ray-ppaszki.7ctq.s1.devshift.org/ray/ray2/ray2

         [ray1] Deleting existing cluster...

         [ray1] Waiting for cluster deletion to complete...

         [ray1] Cluster deleted successfully

         [ray1] Creating cluster from backup...

         [ray1] Waiting for cluster to become ready...

           [DEBUG] Cluster-wide search found 1 items

           [DEBUG] Found Gateway hostname from Route: data-science-gateway.apps.ray-ppaszki.7ctq.s1.devshift.org

         [OK] Restored from backup: ray1 (ns: ray1)

              Dashboard:

              https://data-science-gateway.apps.ray-ppaszki.7ctq.s1.devshift.org/ray/ray1/ray1

         [ray3] Deleting existing cluster...

         [ray3] Waiting for cluster deletion to complete...

         [ray3] Cluster deleted successfully

         [ray3] Creating cluster from backup...

         [ray3] Waiting for cluster to become ready...

           [DEBUG] Cluster-wide search found 1 items

           [DEBUG] Found Gateway hostname from Route: data-science-gateway.apps.ray-ppaszki.7ctq.s1.devshift.org

         [OK] Restored from backup: ray3 (ns: ray3)

              Dashboard:

              https://data-science-gateway.apps.ray-ppaszki.7ctq.s1.devshift.org/ray/ray3/ray3



       Restore from Backup Summary:

         Restored: 3

         Failed: 0
       ```

     **Note**  
       Only use the **rhoai-2.x** version if you want to recover but do not want to progress with the OpenShift AI 3.5 upgrade.

## 

## **4.9. Model serving \- After upgrade** {#4.9.-model-serving---after-upgrade}

Complete the model serving migration by restoring configurations and verifying that all components are functioning correctly after upgrading to Red Hat OpenShift AI 3.5.

After upgrading the Red Hat OpenShift AI operator from version 2.25.9 (and later) to 3.5, you must restore management annotations on the **inferenceservice-config** **ConfigMap** and verify that all model serving components are operational.

**Prerequisites**

* You have completed all procedures in [Migrating model serving before upgrade](#2.8.-model-serving---before-upgrade).

* You have successfully upgraded the Red Hat OpenShift AI operator from version 2.25.9 (and later) to 3.5.

* The operator shows a **Succeeded** phase in the OpenShift web console.

### 

### **4.9.1. Finalize migration configuration** {#4.9.1.-finalize-migration-configuration}

After upgrading to Red Hat OpenShift AI 3.5, restore the management annotation on the **inferenceservice-config** **ConfigMap** and restart the KServe controller.

**Note**

If you were managing a customized **inferenceservice-config** **ConfigMap** manually before the upgrade, you can skip this procedure and proceed directly to the Verification section.

**Prerequisites**

* You have successfully upgraded the Red Hat OpenShift AI operator from version 2.25.9 (and later) to 3.5.

* The operator shows a **Succeeded** phase.

* You have cluster administrator access to your OpenShift cluster.

**Procedure**

1. Run the following command to reset the **inferenceservice-config** **ConfigMap** back to **managed=true** and restart the KServe controller:  
   ```bash
   rhai-cli migrate run --migration modelserving.managed-isvc-config --target-version 3.5.0
   ```

2. Verify that **InferenceServices** were not redeployed by checking that each **InferenceService** has only one **ReplicaSet** with no old scaled-down **ReplicaSets** from a redeployment:

   1. List the namespaces that have InferenceServices:

      ```bash
      oc get isvc -A
      ```

   2. For each namespace that has an InferenceService:

      ```bash
      oc get replicasets -n <namespace> -o custom-columns=NAME:.metadata.name,CREATED:.metadata.creationTimestamp,REPLICAS:.status.replicas
      ```

**Verification**

* Each **InferenceService** should have only one **ReplicaSet** with active replicas. If you see multiple **ReplicaSets** for the same **InferenceService**, with some showing 0 replicas, this indicates an unwanted redeployment occurred.

### **4.9.2. Verifying upgrade completion and troubleshooting** {#4.9.2.-verifying-upgrade-completion-and-troubleshooting}

Verify that the upgrade to Red Hat OpenShift AI 3.5 completed successfully and that all model serving components are functioning correctly.

#### **4.9.2.1. Verification steps** {#4.9.2.1.-verification-steps}

1. Verify the KServe controller is running and ready:

   ```bash
   oc get pods -n redhat-ods-applications -l control-plane=kserve-controller-manager
   ```

   Expected output:

   ```
   NAME                      READY   STATUS    RESTARTS   AGE
   kserve-controller-manager-64b9f745-qxxk9    1/1     Running   0          4m44s
   ```

2. Verify the ODH Model Controller is running and ready:

   ```bash
   oc get pods -n redhat-ods-applications -l control-plane=odh-model-controller
   ```

   Expected output:

   ```
   NAME                                     READY   STATUS    RESTARTS   AGE
   odh-model-controller-78dfd6fc69-nb48v    1/1     Running   0          13m
   ```

3. Verify all **InferenceServices** are using **RawDeployment** mode and are ready:

   ```bash
   oc get isvc -A -o json | jq -r '["NAMESPACE","NAME","DEPLOYMENT_MODE","READY"], (.items[] | [.metadata.namespace, .metadata.name, .status.deploymentMode, (.status.conditions[] | select(.type=="Ready") | .status)]) | @tsv' | column -t
   ```

   Sample expected output:

   ```
   NAMESPACE            NAME          DEPLOYMENT_MODE    READY
   your-isvc-project    isvc-name     Standard           True
   ```

   **Note**

   In Red Hat OpenShift AI 3.5, **RawDeployment** mode has been renamed to **Standard**. The `DEPLOYMENT_MODE` column displays **Standard** for InferenceServices that were previously shown as **RawDeployment** in version 2.25.

4. If you have **LLMInferenceService** resources, verify their status:

   ```bash
   oc get llminferenceservices --all-namespaces
   ```

   Sample expected output:

   ```
   NAMESPACE       NAME            URL                  READY
   llmisvc-ns      llmisvc-name    http://...           True
   ```

5. The READY column must show True and a URL must be present.

**4.9.2.2. Troubleshooting**

##### **4.9.2.2.1. Serverless InferenceServices not converted before upgrade** {#4.9.2.2.1.-serverless-inferenceservices-not-converted-before-upgrade}

**Symptom**

The **InferenceService** reports **READY: True** status, but all inference calls return **HTTP 503 Service Unavailable**.

**Cause**

The **InferenceService** was not converted from **Serverless** to **RawDeployment** mode before upgrading to RHOAI 3.5.

**Resolution**

Convert the workload to **RawDeployment** mode post-upgrade by following the Red Hat Knowledge Base article [Converting ModelMesh and Serverless InferenceServices to RawDeployment (Standard) Mode](https://access.redhat.com/articles/7134025).

#####  **4.9.2.2.2. ModelMesh InferenceServices not converted before upgrade** {#4.9.2.2.2.-modelmesh-inferenceservices-not-converted-before-upgrade}

**Symptom**

The **InferenceService** appears healthy, but requests fail with **HTTP 503** and display a standard OpenShift "Application Not Available" error page.

**Cause**

The **InferenceService** was not converted from **ModelMesh** to **RawDeployment** mode before upgrading to RHOAI 3.5.

**Resolution**

Convert the workload to **RawDeployment** mode using the conversion procedure in the Red Hat Knowledge Base article [Converting ModelMesh and Serverless InferenceServices to RawDeployment (Standard) Mode](https://access.redhat.com/articles/7134025).

**4.9.2.2.3. Serverless Operator not removed**

**Symptom**

**KnativeServing** resource remains in **Ready** state with idle pods in the **knative-serving** namespace.

**Impact**

* **InferenceService**: No impact. Standard **InferenceServices** use **Deployment** and will continue to function correctly (**HTTP 200**).

* **LLMInferenceService**: No impact.

* Infrastructure: Unnecessary resource consumption.

**Resolution**

If you are not using the Serverless Operator outside of OpenShift AI use cases, you can remove it at any time:

1. Uninstall the Serverless Operator through the OpenShift web console.

2. Delete the **knative-serving** namespace:  
   ```bash
   oc delete namespace knative-serving
   ```

#####  **4.9.2.2.4. Authorino Operator not removed** {#4.9.2.2.4.-authorino-operator-not-removed}

**Symptom**

The **Authorino** operator and instance remain in **Ready** state with idle pods running.

**Impact**

* **InferenceService**: No impact. **RawDeployment** **InferenceServices** use **kube-rbac-proxy** as a sidecar for authentication and authorization. The standalone **Authorino** instance is bypassed.

* **LLMInferenceService**: Critical conflict. Distributed Inference with **llm-d** requires Red Hat Connectivity Link (RHCL). Standalone **Authorino** will not work for **LLMInferenceService**.

**Resolution**

1. Uninstall the standalone **Authorino** Operator through the OpenShift web console.

2. Follow the Red Hat Connectivity Link installation and configuration procedures in the distributed inference preparation section.

##### 

##### **4.9.2.2.5. Service Mesh v2 Operator not removed** {#4.9.2.2.5.-service-mesh-v2-operator-not-removed}

**Symptom**

OSSM v2 resources remain in the cluster and Gateway API resources do not function correctly.

**Impact**

* **LLMInferenceService**: Critical conflict. **LLMInferenceService** cannot be used.

* OpenShift AI Dashboard: Will not work correctly.

* Gateway API: The Cluster Ingress Operator fails to install OSSM v3 components due to the presence of OSSM v2.

**Resolution**

1. Verify that no other applications depend on Service Mesh v2 by consulting with application owners.

2. If other applications depend on Service Mesh v2, migrate those applications to Service Mesh v3 following the official Red Hat documentation [Migrating from Service Mesh 2 to Service Mesh 3](https://docs.redhat.com/en/documentation/red_hat_openshift_service_mesh/3.1/html/migrating_from_service_mesh_2_to_service_mesh_3/ossm-migrating-from-service-mesh-2-to-3#ossm-migrating-hub-recommendations-for-migrating_ossm-migrating-from-service-mesh-2-to-3).

3. After migration or if no dependencies exist, uninstall the Service Mesh v2 Operator through the OpenShift web console.

####  **4.9.2.3. Additional resources** {#4.9.2.3.-additional-resources}

* [Multi-model serving platform (ModelMesh) removal](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.0/html/release_notes/support-removals_relnotes#multi_model_serving_platform_modelmesh)

* [Removed functionality in RHOAI 3.5](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/release_notes/support-removals_relnotes)

* [Converting ModelMesh and Serverless InferenceServices to RawDeployment (Standard) Mode](https://access.redhat.com/articles/7134025)

* [ODH-CLI tool on GitHub](https://github.com/opendatahub-io/odh-cli)

**Next steps**

* Monitor your model serving workloads for any unexpected behavior

* Update documentation or runbooks that reference the removed **Serverless** or **ModelMesh** deployment modes

* Communicate endpoint changes to application owners who consume your models

* If the dashboard shows serving runtimes as **Outdated**, redeploy workloads using the latest global serving runtime templates

## **4.10. Kubeflow Training Operator \- After upgrade** {#4.10.-kubeflow-training-operator---after-upgrade}

After you upgrade Red Hat OpenShift AI 2.25.9 (and later) to 3.5, any running PyTorchJobs should continue to run and complete as normal.

**Note**

The Kubeflow Training Operator (KFTO) v1 is deprecated starting with theOpenShift AI 2.25.9 (and later) and is planned to be removed in a future release. This deprecation is part of the OpenShift AI transition to Kubeflow Trainer v2, which delivers enhanced capabilities and improved functionality.

**Prerequisites**

* You have cluster administrator access to your cluster. If you are unsure of your access level, you can run the following commands to confirm that you have the required permissions. Each command should result in a **yes** response:  


  ```bash
  oc auth can-i create namespaces -A
  oc auth can-i delete namespaces -A
  oc auth can-i create pytorchjobs -A
  oc auth can-i delete pytorchjobs -A
  oc auth can-i create pods -A
  oc auth can-i watch pods -A
  ```

* You generated a list of PyTorchJob resources on your OpenShift cluster before you upgraded from OpenShift AI 2.25.9 (and later) to 3.5.

* You have logged in to your OpenShift cluster.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

1. Check that the PyTorchJob resources continue to run after you upgrade to OpenShift AI 3.5:  
   **Note**  
   If there were no running PyTorchJob resources on OpenShift AI 2.25.9 (and later) before the upgrade to 3.5, you can skip this step.

1. Run the following command to get a list of PyTorchJob resources on your OpenShift cluster:  
   ```bash
   oc get pytorchjobs -A
   ```

   Example output:  
   ```bash
   NAMESPACE          NAME                           STATE       AGE  
   pytorch-training   pytorch-distributed-training   Running     4m27s
   ```

2. Compare the list from Step 1a to the list of PyTorchJob resources that you generated before upgrading to OpenShift AI 3.5.

   The before-upgrade and after-upgrade lists should return the same set of resources, unless users created or deleted jobs during or shortly after the upgrade process.

   Note that a PyTorchJob in a failed state does not indicate a failed upgrade; it likely indicates a failure of the job itself. For a command that provides more details about the job, see the *Troubleshooting* section that follows this procedure.

2. Verify training workloads by running the **rhai-cli** training verification action:

   ```bash
   rhai-cli migrate run --migration training.verify-workloads --target-version 3.5.0
   ```

   **Note**  
   You can safely ignore the following warning if it appears:  
   `WARNING: migration training.verify-workloads has phase pre-upgrade but effective phase is post-upgrade`  
   This is a known phase registration mismatch in rhai-cli. The migration runs correctly when invoked with `--migration`.

   The action performs a read-only enumeration of your Kubeflow v1 training workloads (**PyTorchJob**, **TFJob**, **MPIJob**, and **XGBoostJob**) and reports their readiness to migrate to Kubeflow Trainer v2. It does not create any resources or run a test training job, so it completes quickly and does not pull a training image.

   The output reports one of the following results:

   * **No v1 training workloads found — nothing to migrate**, or **All v1 training jobs are completed — safe to proceed**: no action is required.

   * A **\[BLOCKER\]** for any workload that is still active (in the **Running** or **Created** state): the listed jobs must complete or be stopped before you migrate them to Kubeflow Trainer v2. A blocker indicates active v1 workloads, not a failed upgrade.

**Verification**

* If there were running PyTorchJob resources before the upgrade to 3.5, the before-upgrade and after-upgrade lists of running PyTorchJobs contain the same set of resources.

* The `training.verify-workloads` action completes successfully and reports either that there are no v1 training workloads to migrate, or that all v1 training jobs are completed. Any workload reported as a **\[BLOCKER\]** (still **Running** or **Created**) must complete or be stopped before you migrate it to Kubeflow Trainer v2.

**Troubleshooting**

* If a PyTorchJob is in a failed state, it likely indicates a failure of the job itself rather than a failed upgrade. You can get details about the job by running the following command:  

  ```bash
  oc describe pytorchjobs {job_name} -n {namespace_name}
  ```

* If KFTO does not start:

  1. View the **DataScienceCluster** (DSC) state by running the following command:  
     ```bash
     oc describe dsc
     ```

  2. Scroll to the bottom of the resulting output to check the **Conditions** section for information about the issue.

* If KFTO starts but PyTorchJobs are not reconciled, you can inspect the KFTO log by running the following command:  

  ```bash
  oc logs -l app.kubernetes.io/name=trainer -n redhat-ods-applications --tail=-1
  ```

# **5\. Clean up** {#5.-clean-up}

Cleanup

```bash
oc delete statefulset rhai-cli -n <namespace>
oc delete pvc backup-rhai-cli-0 -n <namespace>
```

## **Legal Notice** {#legal-notice}

Copyright © Red Hat.  
Except as otherwise noted below, the text of and illustrations in this documentation are licensed by Red Hat under the Creative Commons Attribution–Share Alike 3.0 Unported license . If you distribute this document or an adaptation of it, you must provide the URL for the original version.

Red Hat, as the licensor of this document, waives the right to enforce, and agrees not to assert, Section 4d of CC-BY-SA to the fullest extent permitted by applicable law.

Red Hat, the Red Hat logo, JBoss, Hibernate, and RHCE are trademarks or registered trademarks of Red Hat, LLC. or its subsidiaries in the United States and other countries.

Linux® is the registered trademark of Linus Torvalds in the United States and other countries.

XFS is a trademark or registered trademark of Hewlett Packard Enterprise Development LP or its subsidiaries in the United States and other countries.

The OpenStack® Word Mark and OpenStack logo are trademarks or registered trademarks of the Linux Foundation, used under license.  
All other trademarks are the property of their respective owners.

