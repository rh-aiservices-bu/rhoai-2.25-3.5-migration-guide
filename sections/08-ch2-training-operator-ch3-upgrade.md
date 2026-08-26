
## **2.11. Kubeflow Training Operator \- Before upgrade** {#2.11.-kubeflow-training-operator---before-upgrade}

You can upgrade Red Hat OpenShift AI 2.25.10 (and later) to 3.5 while PyTorchJobs are running; the jobs continue to run during the upgrade process and complete as normal.

Before you upgrade to OpenShift AI 3.5, get a list of PyTorchJob resources on your OpenShift cluster. You can then use this list to compare against PyTorchJob resources on your OpenShift cluster after you upgrade to 3.5.

**Note**

The Kubeflow Training Operator (KFTO) v1 is deprecated starting with theOpenShift AI 2.25.10 (and later) and is planned to be removed in a future release. This deprecation is part of the OpenShift AI transition to Kubeflow Trainer v2, which delivers enhanced capabilities and improved functionality.

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

Before upgrading Red Hat OpenShift AI from version 2.25.10 (and later) to 3.5, complete the following steps to ensure a successful migration of the OpenShift AI Operator.

**Note**

If you have bookmarked dashboard URLs, you must create redirects **after** the upgrade is complete. For more information, see the [Resolving dashboard URL 404 errors after upgrading from 2.x to 3.x](https://access.redhat.com/solutions/7137771).

**Prerequisites**

* You have upgraded to OpenShift 4.19.9 or later according to OpenShift documentation on [Updating clusters](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/updating_clusters/index).

* You have set the **Update approval** for the Red Hat OpenShift AI 2.25.10 (and later) subscription to **Manual**. This prevents unintended automatic upgrades and requires you to explicitly confirm the upgrade.

* Kueue is set to **Removed** or **Unmanaged** (with external Red Hat build of Kueue Operator installed) within the DSC.

* You have completed the Migrate **InferenceServices** to **RawDeployment** mode steps to convert all serving deployments to **RawDeployment** mode and removed the OpenShift Service Mesh 2 Operator and Serverless Operator.

* You have configured Model Serving to ignore hardware profile annotations to avoid inference service restarts during the upgrade, according to Update the inferenceservice-config ConfigMap.

* You have set the **CodeFlare** component to **Removed** in the DataScienceCluster resource. CodeFlare is removed in OpenShift AI 3.5 and must be disabled before upgrading, even if you have no RayClusters. If you completed the Ray pre-upgrade migration (Section 2.7), you already performed this step. Otherwise, run:

  ```bash
  oc patch dsc default-dsc --type=merge \
    -p '{"spec":{"components":{"codeflare":{"managementState":"Removed"}}}}'
  ```

* You migrated any other component workloads that require migration before the upgrade.

* You have OpenShift cluster administrator permissions to install Operators and edit **DataScienceCluster** and **DataScienceClusterInitialization** resources.

* The `rhai-cli lint --target-version 3.5` does not report any errors

**Procedure**

1. Verify that the **Update approval** for the Red Hat OpenShift AI 2.25.10 (and later) subscription is set to **Manual**.

   If the **Update approval** is not set to **Manual**, you must set it now. This prevents automatic upgrade when you change the subscription channel.

2. Edit the **Update channel** for Red Hat OpenShift AI to **support-required-upgrade-3.5**.

   **Note**: For cross-major upgrades from 2.25 to 3.5, only the **support-required-upgrade-3.5** channel provides a valid upgrade path. Other 3.x channels such as **stable-3.5** or **stable-3.x** are for same-major upgrades only and do not provide an upgrade from 2.25.

   For information about subscription channels and their lifecycle, see [Red Hat OpenShift AI Self-Managed Life Cycle](https://access.redhat.com/support/policy/updates/rhoai-sm/lifecycle#stable).

**Verification**

1. Verify that the Red Hat OpenShift AI 2.25.10 (and later) CSV status shows **Succeeded**.  
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

* Rerun the rhai-cli script to make sure all critical issues are resolved.

* For disconnected environments, you have a mirror registry and oc-mirror v2, as described in [Mirroring images for a disconnected installation by using the oc-mirror plugin v2](https://docs.redhat.com/en/documentation/openshift_container_platform/4.19/html/disconnected_environments/about-installing-oc-mirror-v2).

**Procedure**

1. You must log out of the OpenShift AI dashboard before starting the upgrade. OpenShift AI does not support Zero Downtime Upgrade.

   For connected environments, skip to Step 5\.

   For disconnected environments, continue to Step 2\.

2. For disconnected environments:

   Identify the OSSM version the Cluster Ingress Operator requires.  
   In the following steps, replace **\<OSSM_VERSION\>**  with this value (for example, servicemeshoperator3.v3.1.0):

   ```bash
   oc set env deployment/ingress-operator -n openshift-ingress-operator --list \
       | grep GATEWAY_API_OPERATOR_VERSION \
       | sed 's/.*=//'
   ```

3. Identify the OSSM channel the Cluster Ingress Operator uses to install OSSM.   
   In the following steps  replace **\<OSSM_CHANNEL\>** with this value (for example,  stable):

   ```bash
   oc set env deployment/ingress-operator -n openshift-ingress-operator --list \
       | grep GATEWAY_API_OPERATOR_CHANNEL \
       | sed 's/.*=//'
   ```

### 

4. Mirror the exact OSSM version identified in Step 2 into the disconnected registry:  
   1. Create the ImageSetConfiguration. Replace **\<OCP_VERSION\>**, **\<OSSM_VERSION\>** and **\<OSSM_CHANNEL\>** with your values:

      ```bash
      cat > imageset-config.yaml <<EOF
      apiVersion: mirror.openshift.io/v2alpha1
      kind: ImageSetConfiguration
      mirror:
        operators:
          - catalog: registry.redhat.io/redhat/redhat-operator-index:v<OCP_VERSION>
            packages:
              - name: servicemeshoperator3
                channels:
                  - name: <OSSM_CHANNEL>
                    minVersion: <OSSM_VERSION>
                    maxVersion: <OSSM_VERSION>
      EOF
      ```

      

      **b.** Run oc-mirror to mirror the images. Replace **\<MIRROR_REGISTRY\>** with your registry URL:

      

      ```bash
      oc-mirror --v2 --config=imageset-config.yaml \
          --workspace file://oc-mirror-workspace \
          docker://<MIRROR_REGISTRY>
      ```

   

      **c.** If CatalogSource redhat-operators already exists on the cluster, skip this step and continue with step 4d  to verify that it references the version you just mirrored in step 4b. 

      If CatalogSource redhat-operators doesn’t exist on the cluster, change the name of the CatalogSource generated by oc-mirror to redhat-operators and apply the generated cluster resources so that  the cluster is aware of the mirrored content: 

   ```bash
   oc apply -f oc-mirror-workspace/working-dir/cluster-resources/
   ```

   

   **d.** Verify that the required version of OSSM is available in the required channel in the mirrored CatalogSource named redhat-operators:

   ```bash
   oc get packagemanifest -o json | jq '.items[] | select (.metadata.name == "servicemeshoperator3" and .status.catalogSource == "redhat-operators") | .status.channels[] | select (.name == "<OSSM_CHANNEL>") | .entries[].name'
   ```

   

   The output should include the **\<OSSM_VERSION\>** from Step 2\. If it doesn’t include the version from Step 2, make sure that the CatalogSource named redhat-operators references it.

5. **FBC (File-Based Catalog) environments only:** If you installed Red Hat OpenShift AI using a custom FBC CatalogSource (for example, for pre-release testing), you must update the CatalogSource image to the target version before switching channels. The source FBC fragment only contains channels up to the source version.

   ```bash
   oc patch catalogsource <CATALOG_NAME> -n openshift-marketplace \
     --type=merge -p '{"spec":{"image":"<TARGET_FBC_FRAGMENT_IMAGE>"}}'
   ```

   Delete the catalog pod to force OLM to rebuild its cache. Without this step, OLM may serve stale channel data even after the CatalogSource reports READY:

   ```bash
   oc delete pod -n openshift-marketplace -l olm.catalogSource=<CATALOG_NAME>
   ```

   Wait for the CatalogSource to reach **READY** state:

   ```bash
   oc get catalogsource <CATALOG_NAME> -n openshift-marketplace \
     -o jsonpath='{.status.connectionState.lastObservedState}'
   ```

   Verify that the **support-required-upgrade-3.5** channel is now available. If your cluster also has the default **redhat-operators** CatalogSource, you must query the packagemanifest from your custom FBC CatalogSource specifically, because `oc get packagemanifest` may return data from **redhat-operators** by default:

   ```bash
   oc get packagemanifest -n openshift-marketplace -o json | \
       jq -r '.items[] | select(.metadata.name == "rhods-operator" and
       .status.catalogSource == "<CATALOG_NAME>") |
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
   Several other 3.x channels might be visible in the **Change Subscription update channels** list, such as fast-3.x, stable-3.5, and stable-3.x. However, these channels do not provide a cross-major upgrade from 2.25. Only the **support-required-upgrade-3.5** channel provides an upgrade from 2.25.10 or later to 3.5. The unversioned **support-required-upgrade** channel is for upgrading from 2.25 to 3.3 only and should not be used here.

11. Approve the install plan to begin the upgrade.

   1. In the **Upgrade status** section, click the "requires approval" link to approve the upgrade installation.  
   2. Review the upgrade install plan details and click **Approve**. The upgrade process begins.

12. While the upgrade is in progress, monitor the following:

   1. Watch the operator pods as they restart to replace the version 2.25.10 (and later) Operator.  
   2. Verify that the new operator pods reach the **Running** state and that the **Ready** condition is **True**.

13. Install the **JobSet** operator. OpenShift AI 3.5 requires the JobSet operator as a Kueue dependency. Without it, the **DataScienceCluster** remains in a **Not Ready** state with `KueueReady=False`.

   **Important**  
   The JobSet operator only supports **OwnNamespace** and **SingleNamespace** install modes. Do not install it in the `openshift-operators` namespace, which uses an **AllNamespaces** OperatorGroup.

   a. Create a dedicated namespace:

      ```bash
      oc create namespace jobset-system
      ```

   b. Create an **OwnNamespace** OperatorGroup:

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

   c. Subscribe to the operator:

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

   d. Wait for the CSV to reach **Succeeded**:

      ```bash
      oc wait csv jobset-operator.v1.0.0 -n jobset-system \
        --for=jsonpath='{.status.phase}'=Succeeded --timeout=120s
      ```

      **Tip**  
      If the CSV name differs from `jobset-operator.v1.0.0`, verify it with `oc get csv -n jobset-system`.

   e. Create the **JobSetOperator** custom resource to deploy the operand. This installs the `jobsets.jobset.x-k8s.io` CRD that Kueue requires:

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

   f. Verify that the CRD exists and KueueReady is True:

      ```bash
      oc get crd jobsets.jobset.x-k8s.io
      oc get dsc -o jsonpath='{.items[0].status.conditions[?(@.type=="KueueReady")].status}' && echo
      ```

      Expected output: the CRD is listed and KueueReady shows **True**.
      **Note* If the kueue component was set to `removed` in the DSC before upgrade then the above command will return **False**. For step to setting up kueue See [Set Kueue management to Unmanaged](#2.2-set-kueue-management-to-removed)

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

