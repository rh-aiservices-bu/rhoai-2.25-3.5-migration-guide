## **4.7. Workbenches \- After upgrade** {#4.7.-workbenches---after-upgrade}

### **4.7.1. Migrate your workbenches after upgrade** {#4.7.1.-migrate-your-workbenches-after-upgrade}

**Important**

All procedures in this section must be completed after upgrading the Red Hat OpenShift AI Operator from version 2.25.10 (and later) to 3.5. Failure to complete these steps can result in service disruptions for your workbenches.

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
* **Migration workbenched.patch-auth-model completed successfully!**  
* **All migrations completed successfully!**

3. Notify your users that any workbenches that were stopped in order to prepare for the upgrade can now be started again.

**Verification**

1. Verify that the workbench upgrade was successful by running the following command:


   ```bash
   rhai-cli migrate run --migration workbenches.verify-migration --target-version 3.5.0
   ```

   As the command runs, it provides output that indicates the status of your workbench upgrade. When the command completes, you should see a message similar to the following:  
    **All migrations completed successfully!**

2. Confirm with your OpenShift AI users that they can access their workbench IDE from a web browser.

### **4.7.2. Perform a deferred workbench image migration** {#4.7.2.-perform-a-deferred-workbench-image-migration}

If you are unable to stop some or all of your workbenches during the Red Hat OpenShift AI upgrade from version 2.25.10 (and later) to 3.5, your OpenShift AI users can defer their workbench image migration until after the upgrade.

**Important**

* Workbench images left unmigrated continue to operate on the older 2.25.10 (and later) authentication layer. This hybrid environment can result in redirection loops and connectivity failures, primarily due to **NB\_PREFIX** routing conflicts for RStudio, code-server, and custom images.

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

After upgrading from Red Hat OpenShift AI 2.25.10 (and later) to 3.5, use the **rhai-cli** tool to migrate your Ray clusters.

**Note**

The commands in the following procedure include an optional **\--dry-run** argument. Include this argument to preview how the command functions.

**Prerequisites**

* **WARNING:** You must follow the After upgrade steps for the Workbench component before you migrate your Ray clusters.

* You have cluster administrator access.

* You have access to the **rhai-cli** tool.

**Procedure**

1. Preview the migration status of Ray clusters on your OpenShift cluster by using the **\--dry-run** flag:

   ```bash
   rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --dry-run
   ```

   The output lists each Ray cluster and its migration status, as shown in the following example:

   ```
   === DRY RUN MODE - No changes will be made ===

   Fetching RayClusters (all namespaces)...
   Found 2 RayCluster(s)

   Analyzing clusters for migration status...
     [1/2] Checking comprehensive-mixed (ns: raytest)... needs migration
     [2/2] Checking sdk-configurations (ns: raytest)... needs migration

   Summary: 2 to migrate, 0 already migrated

     [DRY RUN] Would migrate: comprehensive-mixed (ns: raytest)
     [DRY RUN] Would migrate: sdk-configurations (ns: raytest)

   ============================================================
   Migration Summary:
     Migrated: 2
     Skipped (already migrated): 0
     Failed: 0
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

   **Important**
   The Dashboard URL uses Gateway API routing. For the URL to be accessible, the **AIGateway** component must be enabled (set to **Managed**) in the DataScienceCluster. If AIGateway is set to **Removed**, the HTTPRoute will be created but will not be programmed and the Dashboard URL will return HTTP 503. Verify:

   ```bash
   oc get datasciencecluster default-dsc -o jsonpath='{.spec.components.aigateway.managementState}' && echo
   ```

   Expected output: `Managed`

**Verification**

1. In a web browser, verify that the Dashboard URL output by the migration command provides access to the Ray cluster.

2. Verify the migration status of all Ray clusters by using the **\--dry-run** flag:

   ```bash
   rhai-cli migrate run --migration raycluster.migrate --target-version 3.5.0 --dry-run
   ```

   Example output after migrating one of three clusters:

   ```
   === DRY RUN MODE - No changes will be made ===

   Fetching RayClusters (all namespaces)...
   Found 3 RayCluster(s)

   Analyzing clusters for migration status...
     [1/3] Checking production-cluster (ns: production)... already migrated
     [2/3] Checking staging-cluster (ns: staging)... needs migration
     [3/3] Checking dev-cluster (ns: dev)... needs migration

   Summary: 2 to migrate, 1 already migrated

     [DRY RUN] Would migrate: staging-cluster (ns: staging)
     [DRY RUN] Would migrate: dev-cluster (ns: dev)

   ============================================================
   Migration Summary:
     Migrated: 2
     Skipped (already migrated): 1
     Failed: 0
   ```

   If any clusters show **needs migration** status, run the **raycluster.migrate** migration command for that Ray cluster.

**Troubleshooting**

Use the following information to help troubleshoot problems with Ray clusters after you upgrade from Red Hat OpenShift AI 2.25.10 (and later) to 3.5.

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

  During the migration process, the Ray cluster URL changes to use Gateway API access. The 2.25.10 (and later) route URL is no longer accessible for the Ray cluster. When you run the **raycluster.migrate** migration command, it outputs the new URL to access the migrated Ray cluster. If you did not make a note of the new URL, use the CodeFlare SDK inside a workbench to query all your Ray clusters:  

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

  1. If you have not already run the **pre-upgrade** migration command to create back ups of your Ray cluster Custom Resource (CR) files, follow the procedure described in *Ray Training Operator \- Before upgrade*. You can do this step before or after upgrading from OpenShift AI 2.25.10 (and later) to 3\.

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

After upgrading the Red Hat OpenShift AI operator from version 2.25.10 (and later) to 3.5, you must restore management annotations on the **inferenceservice-config** **ConfigMap** and verify that all model serving components are operational.

**Prerequisites**

* You have completed all procedures in [Migrating model serving before upgrade](#2.8.-model-serving---before-upgrade).

* You have successfully upgraded the Red Hat OpenShift AI operator from version 2.25.10 (and later) to 3.5.

* The operator shows a **Succeeded** phase in the OpenShift web console.

### 

### **4.9.1. Finalize migration configuration** {#4.9.1.-finalize-migration-configuration}

After upgrading to Red Hat OpenShift AI 3.5, restore the management annotation on the **inferenceservice-config** **ConfigMap** and restart the KServe controller.

**Note**

If you were managing a customized **inferenceservice-config** **ConfigMap** manually before the upgrade, you can skip this procedure and proceed directly to the Verification section.

**Prerequisites**

* You have successfully upgraded the Red Hat OpenShift AI operator from version 2.25.10 (and later) to 3.5.

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
      oc get replicasets -n <NAMESPACE> -o custom-columns=NAME:.metadata.name,CREATED:.metadata.creationTimestamp,REPLICAS:.status.replicas
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

