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

3. Create a directory for backups on the PVC inside the **rhai-cli** pod:

   ```bash
   mkdir -p /tmp/rhoai-upgrade-backup/trustyai
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
   ```

   **Note**  
   The backup directory must reside on the PVC mounted at `/tmp/rhoai-upgrade-backup` inside the **rhai-cli** pod so that backups persist across the upgrade. The backup procedure in the next section re-creates this directory and re-exports `BACKUP_DIR` inside the pod, so you do not need to keep this shell session open.

### **2.7.2. TrustyAI \- Before upgrade \- Backup metrics** {#2.7.2.-trustyai---before-upgrade---backup-metrics}

You can backup scheduled TrustyAI metrics before you upgrade OpenShift AI 2.25.9 (and later) to 3.5.

**Prerequisites**

* You have cluster administrator access.

* You have the OpenShift **oc** command line interface installed.

* You completed the steps in TrustyAI \- Before upgrade \- Prepare for backup.

* You have access to the **rhai-cli** tool, as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image).

**Procedure**

For each namespace that has a TrustyAI service, follow these steps to backup scheduled TrustyAI metrics.

**Important**  
Run this entire procedure — steps 1 through 9 **and** all of the verification steps — **inside the rhai-cli pod**, in a single shell session (for example, `oc exec -it rhai-cli-0 -n rhai-migration -- bash`, then log in as described in [Deploy the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image)). The metrics backup file must be written to the PVC mounted at `/tmp/rhoai-upgrade-backup` so that it persists across the upgrade. If you run the `curl` command in step 8 from your workstation, the file is saved to your local machine instead of the PVC, and the verification steps fail. Running every step in the same in-pod session also keeps the `BACKUP_DIR`, `NS`, and `PF_PID` variables consistent.

1. Ensure the backup directory exists, then set the namespace and backup directory variables:

   ```bash
   mkdir -p /tmp/rhoai-upgrade-backup/trustyai
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
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

   ```bash
   cat ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | jq empty && echo "OK" || echo "FAIL: invalid JSON"
   ```

   Example output:

   ```
   OK
   ```

3. Verify that the backup file exists:

   ```bash
   ls ${BACKUP_DIR}/trustyai-metrics-${NS}-*
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
