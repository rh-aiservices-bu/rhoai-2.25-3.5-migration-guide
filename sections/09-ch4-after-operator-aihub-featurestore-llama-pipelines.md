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

3. Verify that all Operator pods in the Operator namespace have **Status** equal to **Running** and their condition **Ready** is **True\`**.

   ```bash
   oc get pods -n redhat-ods-operator -o custom-columns='NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status,STATUS:.status.phase'
   ```

4. Verify that all component controller pods in the applications namespace have **Status** equal to **Running** and their condition **Ready** is **True**.

   ```bash
   oc get pods -n redhat-ods-applications -o custom-columns='NAME:.metadata.name,READY:.status.conditions[?(@.type=="Ready")].status,STATUS:.status.phase'
   ```

5. Verify that the RHOAI gateway is ready:

   ```bash
   oc get gatewayconfigs --all-namespaces -o wide
   ```

   Expected output is: default-gateway shows READY: True

6. If the dashboard component is installed, verify that the console link exists and functions correctly.

7. Click the **Red Hat OpenShift AI** link in the OpenShift console to go to the log in page.  
8. Log in to the application and confirm that the dashboard is displayed.

9. Change the OpenShift AI Operator update channel:

10. In the OpenShift web console, select **Operators \> Installed Operators**.

11. Select the **Red Hat OpenShift AI 3.5 Operator**. 

12. Select Subscriptions and then select the Upgrade channel: **support-required-upgrade-3.5** 

13. From the list of channels, select the 3.x channel that you want to use going forward, for example: **stable-3.5**, **stable-3.x** or **eus-3.5**.

   For more information about update channels, see [Understanding update channels.](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/installing_and_uninstalling_openshift_ai_self-managed/index#understanding-update-channels_install)

14. Click **Save**.

   If there are later OpenShift AI releases available, either later on the 3.5 release or for a later release, such as 3.4 or 3.5, the Operator subscription proposes install plans for them. 

15. Optionally, you can select to upgrade your environment to the most current release of OpenShift AI 3.x.

**Troubleshooting** 

Use the following information to help troubleshoot problems after you upgrade from Red Hat OpenShift AI 2.25.9 (and later) to 3.5.

**OpenShift AI components that depend on OpenShift Service Mesh (OSSM)  do not become ready**

After upgrading Red Hat OpenShift AI on a disconnected cluster, the Red Hat OpenShift Service Mesh (OSSM) 3.0 operator fails to install. OpenShift AI components that depend on OSSM do not become ready, for example:

- The `servicemeshoperator3` subscription in the `openshift-operators` namespace shows a failed state.  
- The `InstallPlan` for `servicemeshoperator3` fails or is not created.  
- The `DSCInitialization` (`default-dsci`) does not reach `Ready` state.  
- The `data-science-gateway` Gateway in `openshift-ingress` shows `Unknown` status.

For information on how to resolve this issue, see the [OpenShift Service Mesh 3.x fails to deploy on a disconnected cluster during Red Hat OpenShift AI installation](https://access.redhat.com/solutions/7141146) knowledgebase article.

**Note** With the OCP release of 4.20.31+. 4.21.22+, or 4.22+ Service Mesh 3 is no longer required as a dependency operator. 

**Resolving dashboard URL 404 errors**  
If you have bookmarked dashboard URLs, you must recreate redirects after the upgrade is complete. For more information, see the [Resolving dashboard URL 404 errors after upgrading from 2.x to 3.x](https://access.redhat.com/solutions/7137771).

**What the operator handles automatically**

During the upgrade from OpenShift AI 2.25.9 to 3.5, the operator automatically performs the following migrations on startup. Cluster administrators do not need to perform these steps manually:

* **HardwareProfile auto-migration**: The operator automatically migrates AcceleratorProfiles to HardwareProfiles and attaches HardwareProfile annotations to Notebooks and InferenceServices. This migration uses create-only semantics and preserves any user customizations.

* **GatewayConfig ingressMode migration**: The operator preserves the LoadBalancer ingress mode for existing Gateway deployments, preventing a regression to the default OcpRoute mode.

* **Kueue ValidatingAdmissionPolicyBinding cleanup**: The operator removes deprecated ValidatingAdmissionPolicyBinding resources from pre-v2.29.

* **DSC v1 to v2 API conversion**: The operator automatically handles the conversion from DSC v1 API to v2 API. Field renames (e.g., `datasciencepipelines` to `aipipelines`, removal of `modelmeshserving` and `codeflare`) are handled by the conversion webhook. Administrators do not need to manually recreate the DSC.

***Kueue \- after upgrade***

After upgrading to OpenShift AI 3.5, verify that the Kueue component is properly configured and operational. If the Kueue migration to Red Hat build of Kueue was not completed before the upgrade, you must recover by completing the migration steps.

***Prerequisites***

* *You have cluster administrator access to the OpenShift cluster.  
* *You have upgraded to OpenShift AI 3.5.*

***Procedure***

1. Check that the Kueue component is in a ready state as follows:

   ```bash
   read STATUS REASON < <(oc get datasciencecluster -A -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status} {.items[0].status.conditions[?(@.type=="KueueReady")].reason}'); [[ "$STATUS" == "True" || ( "$STATUS" == "False" && "$REASON" == "Removed" ) ]] && echo "Ready" || echo "Not Ready"
   ```

2. If the output is Ready, you can skip all of the remaining steps.

   If the output of the preceding command is Not Ready, check the Kueue component status as follows:
   ```bash
   oc get datasciencecluster -A -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status}{"\n"}{.items[0].status.conditions[?(@.type=="KueueReady")].message}{"\n"}'
   ```  
     
   The following output indicates that Kueue was not migrated to Red Hat build of Kueue and that migration is required:  
   ```bash
   False
   ```
   Kueue managementState Managed is not supported, please use Removed or Unmanaged*  
     
   The following output indicates that you have a non-Kueue managed environment and that migration is not required:
   ```bash
   False  
   Component ManagementState is set to Removed
   ```  
3. If migration to Red Hat build of Kueue is required, you must follow the steps in Kueue \- Before upgrade. See [Set Kueue management to Unmanaged](#2.2-set-kueue-management-to-removed)

   ***Important***  
   Pay close attention to all steps and caveats in the migration procedure, particularly the framework configuration annotation if you are using the default Kueue configuration.

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

3. In the OpenShift AI dashboard, click **Settings \> Model resources and operations \> Model registry settings** to check the status of your model registries. For more information, see [Managing model registries](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/managing_model_registries).

4. Click **AI hub \> Models** to check the default catalog and any custom catalogs that were created. For more information, see [Working with the model catalog](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/working_with_the_model_catalog).

**Verification**

1. In OpenShift, there are no errors in the following pods:

   * **\<my-model-registry\>-xxx**

   * **db-\<my-model-registry\>-xxx**

   * **model-catalog-xxx**

   * **model-catalog-postgres-xxx**

2. In the OpenShift AI dashboard, on the **Settings \> Model resources and operations \> Model registry settings** page, your model registries are displayed with **Available** status.

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

* After you complete the Llama Stack before-upgrade steps and the upgrade process, you can create new **OGXServer** CRs and applications in OpenShift AI version 3.5 using the **llsd-backup.yaml** file as a reference. This file was created in "Llama Stack upgrade steps for **LlamaStackDistribution** resource owners". Refer to the [Deploying a Llama Stack server](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html/working_with_ogx/deploying-ogx-server_rag) documentation for more information.

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

