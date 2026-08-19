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
