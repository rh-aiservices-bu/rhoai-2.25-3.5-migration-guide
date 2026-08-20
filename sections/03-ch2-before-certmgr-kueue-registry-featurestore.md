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

