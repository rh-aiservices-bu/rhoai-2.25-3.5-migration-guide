## **2.8. Workbenches \- Before upgrade** {#2.8.-workbenches---before-upgrade}

### **2.8.1. About upgrading your workbenches** {#2.8.1.-about-upgrading-your-workbenches}

As a Red Hat OpenShift AI administrator, you have flexibility in your upgrade strategy for your workbench images and server.

There are two major upgrade paths to follow. As a Red Hat OpenShift AI administrator, you can either manage the upgrade fully by ensuring all workbenches have been stopped and workbench images have been migrated prior to the upgrade, or defer migration and enable your users to continue their workbench image migration with user self-service. Deferred migrations come with additional risk, so it is important to understand the impacts of your upgrade path. See [Perform a deferred workbench image migration](#4.7.2.-perform-a-deferred-workbench-image-migration) for more information.

**Considerations before upgrading your workbench**

* Workbenches contain active user development environments. Coordinate with all users before upgrading to prevent loss of unsaved work.

* Workbench URLs will change after the upgrade. Users must obtain their new URLs from the OpenShift AI dashboard. Previously bookmarked URLs will no longer work.

* Organizations using custom workbench images must rebuild them to support the new authentication layer and path-based routing in OpenShift AI version 3.5. Custom workbench images built for OpenShift AI version 2.x are not compatible with the new **kube-rbac-proxy** authentication mechanism. See [Introducing the Kubernetes Gateway API for custom image migration](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/managing_resources/introducing-kubernetes-gateway-api_resource-mgmt) for more information.

* RStudio workbenches require a new build from the RStudio BuildConfig after the upgrade. The existing image will not be compatible with the new authentication layer without this rebuild.

* Workbenches created in Red Hat OpenShift AI version 2.25.9 (and later) or earlier are officially unsupported in the Red Hat OpenShift AI 3.5 environment unless they have been manually migrated.

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

2. Run a pre-upgrade check that verifies your configuration, verifies that the Ray clusters are ready for the upgrade, and backs up your Ray cluster CR configuration YAML files:  
   **Note**  
   The migration backs up your Ray cluster CR configuration YAML files only. It does not back up the state of your Ray clusters.  
   ```bash
   rhai-cli migrate run --migration raycluster.backup --target-version 3.5.0
   ```
   The migration runs pre-upgrade checks. If all pre-upgrade checks succeed, then it saves your Ray cluster CR configurations to the following subdirectories under the backup directory:

   * **Rhoai-2.x** \- Your Ray cluster CR configurations YAML files that are compatible with OpenShift AI 2.x.

   * **Rhoai-3.x** \- Your Ray cluster CR configurations YAML files that are compatible with OpenShift AI 3.x.

3. Get a list of the Ray clusters and check their status:

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

