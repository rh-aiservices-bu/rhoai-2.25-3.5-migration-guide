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

