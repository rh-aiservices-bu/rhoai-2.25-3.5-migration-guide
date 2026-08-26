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

**Note - Workstation Procedure**\
Run every command in this procedure from your workstation. The commands read the backups out of the **rhai-cli** pod with `oc exec` and pipe them to `jq`, which is the only tool used here that is not installed in the pod. Set the environment variables in the first step in the same shell so they are available to the later steps.

1. Set the backup directory and the namespace where the **rhai-cli** StatefulSet is deployed:

   ```bash
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
   export RHAI_CLI_NS=<RHAI_CLI_NAMESPACE>
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
   oc get pods -n redhat-ods-applications -l 'control-plane in (controller-manager,trustyai-service-operator)' -o wide
   ```

4. List the namespaces for which you have backups. This lists the backups inside the **rhai-cli** pod with `oc exec` (where `BACKUP_DIR` lives) and formats the output with `sed` and `sort`:

   ```bash
   oc exec rhai-cli-0 -n "$RHAI_CLI_NS" -- sh -c "ls ${BACKUP_DIR}/trustyai-metrics-*.json 2>/dev/null" \
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
   export NS=<NAMESPACE>
   export TAS_NAME=$(oc get trustyaiservice -n "$NS" -o jsonpath='{.items[0].metadata.name}')
   export SVC_PORT=$(oc get svc -n "$NS" "$TAS_NAME" -o jsonpath='{.spec.ports[?(@.name=="http")].port}')
   : "${NS:?NS is not set. Set it with: export NS=<NAMESPACE>}"
   : "${BACKUP_DIR:?BACKUP_DIR is not set. Set it to the backup path inside the rhai-cli pod, e.g. export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai}"
   oc port-forward -n "$NS" "svc/$TAS_NAME" 8080:${SVC_PORT} &
   export PF_PID=$!
   export CURRENT=$(curl -sk -H "Authorization: Bearer $(oc whoami -t)" \
     "http://localhost:8080/metrics/all/requests" | jq '.requests | length')
   export BACKUP=$(oc exec rhai-cli-0 -n "$RHAI_CLI_NS" -- sh -c "cat \$(ls -t ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | head -1)" | jq '.requests | length')
   kill $PF_PID 2>/dev/null
   echo "Current: $CURRENT | Backup: $BACKUP"
   [ "$CURRENT" -ge "$BACKUP" ] && echo "OK: no data loss" || echo "DATA LOSS: restore needed"
   ```

If the result is OK, no data was lost and the service is running successfully. Repeat this step for each namespace that has a backup, then continue to [TrustyAI \- After upgrade \- Guardrails](#4.6.2.-trustyai---after-upgrade---guardrails).

If the result is DATA LOSS, see [TrustyAI \- After upgrade \- Restore data](#4.6.3.-trustyai---after-upgrade---restore-data).

**Note - Repeat the above steps for the other TrustyAI Namespaces**

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
      Auto-detected phase: post-upgrade
      Current OpenShift AI version: 3.5.0
      Target OpenShift AI version: 3.5.0
      Phase: post-upgrade


      trustyai.patch-guardrails:

      Preparing migration: trustyai.patch-guardrails

        → Patch GuardrailsOrchestrator readinessProbe
        → Found 1 GuardrailsOrchestrator CR(s)
          ✓ Found 1 GuardrailsOrchestrator CR(s)
        → Check test-guardrails-hf-upgrade/guardrails-orchestrator
          ✓ test-guardrails-hf-upgrade/guardrails-orchestrator needs readinessProbe patch

      About to patch readinessProbe on 1 deployment(s)
      Proceed with patching? [y/N]: y
        → Patch test-guardrails-hf-upgrade/guardrails-orchestrator
          ✓ Patched test-guardrails-hf-upgrade/guardrails-orchestrator
          ✓ Patched 1/1 deployment(s)

      Migration trustyai.patch-guardrails completed successfully!

      All migrations completed successfully!
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
   Auto-detected phase: post-upgrade
   Current OpenShift AI version: 3.5.0
   Target OpenShift AI version: 3.5.0
   Phase: post-upgrade

   WARNING: migration trustyai.migrate-gorch-otel-exporter has phase pre-upgrade but effective phase is post-upgrade; proceeding because --migration was explicit

   trustyai.migrate-gorch-otel-exporter:

   DRY RUN MODE: No changes will be made to the cluster

     → Migrate otelExporter schema
     → Found 1 GuardrailsOrchestrator CR(s)
       ✓ Found 1 GuardrailsOrchestrator CR(s)
     → Check test-guardrails-hf-upgrade/guardrails-orchestrator
       → guardrails-orchestrator: already on new otelExporter schema
       ✓ No GuardrailsOrchestrator CRs need otelExporter migration

   Migration trustyai.migrate-gorch-otel-exporter completed with skipped steps

   All migrations completed (some steps were skipped).
   ```

   If all **GuardrailsOrchestrator** CRs report as **already on new otelExporter schema**, skip to step 5 to verify and, if necessary, restore the exporter configuration.  
   Otherwise, continue to the next step.

4. Run the migration that patches the existing **GuardrailsOrchestrator** deployments by updating the keys under **spec.otelExporter**:

   ```bash
   rhai-cli migrate run --migration trustyai.migrate-gorch-otel-exporter --target-version 3.5.0
   ```

   Example output:
   ```
   Auto-detected phase: post-upgrade
   Current OpenShift AI version: 3.5.0
   Target OpenShift AI version: 3.5.0
   Phase: post-upgrade

   WARNING: migration trustyai.migrate-gorch-otel-exporter has phase pre-upgrade but effective phase is post-upgrade; proceeding because --migration was explicit

   trustyai.migrate-gorch-otel-exporter:

   Preparing migration: trustyai.migrate-gorch-otel-exporter

     → Migrate otelExporter schema
     → Found 1 GuardrailsOrchestrator CR(s)
       ✓ Found 1 GuardrailsOrchestrator CR(s)
     → Check test-guardrails-hf-upgrade/guardrails-orchestrator
       → guardrails-orchestrator: already on new otelExporter schema
       ✓ No GuardrailsOrchestrator CRs need otelExporter migration

   Migration trustyai.migrate-gorch-otel-exporter completed with skipped steps

   All migrations completed (some steps were skipped).
   ```

5. Restore your traces and metrics exporter configuration from the backup you created in [TrustyAI \- Before upgrade \- Guardrails Orchestrator](#2.7.4.-trustyai---before-upgrade---guardrails-orchestrator).

   The upgrade carries over only `protocol` (as `otlpProtocol`) and drops the other **spec.otelExporter** keys. Because `otlpProtocol` is present, `trustyai.migrate-gorch-otel-exporter` reports **already on new otelExporter schema** and does not restore them. Migrate them from your backup, mapping to the 3.5 schema, by running the following command:

   **Note - Workstation Step**  
   Run this from your workstation, not the **rhai-cli** pod, which does not have `jq`. Set `RHAI_CLI_NS` to the namespace where the **rhai-cli** StatefulSet is deployed, `NS` to the namespace of the **GuardrailsOrchestrator**, `GORCH_NAME` to its name, and `BACKUP_DIR` to the backup path inside the **rhai-cli** pod.

   ```bash
   export RHAI_CLI_NS=<RHAI_CLI_NAMESPACE>
   export NS=<NAMESPACE>
   export GORCH_NAME=<GORCH_NAME>
   export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai
   : "${NS:?NS is not set. Set it with: export NS=<NAMESPACE>}"
   : "${GORCH_NAME:?GORCH_NAME is not set. Set it with: export GORCH_NAME=<GORCH_NAME>}"
   : "${BACKUP_DIR:?BACKUP_DIR is not set. Set it to the backup path inside the rhai-cli pod, e.g. export BACKUP_DIR=/tmp/rhoai-upgrade-backup/trustyai}"
   oc patch guardrailsorchestrator "$GORCH_NAME" -n "$NS" --type merge -p "$(
     oc exec rhai-cli-0 -n "$RHAI_CLI_NS" -- cat "${BACKUP_DIR}/${GORCH_NAME}-${NS}-otelExporter-backup.json" | jq '{
       spec: {
         otelExporter: {
           otlpProtocol: (.protocol // "grpc"),
           otlpTracesEndpoint: (.tracesEndpoint // .otlpEndpoint),
           otlpMetricsEndpoint: (.metricsEndpoint // .otlpEndpoint),
           enableTraces: ((.otlpExport // "") | test("traces")),
           enableMetrics: ((.otlpExport // "") | test("metrics"))
         }
       }
     } | del(.spec.otelExporter[] | select(. == null or . == ""))'
   )"
   ```

   Example output:
   ```
   guardrailsorchestrator.trustyai.opendatahub.io/guardrails-orchestrator patched
   ```

   Read back `spec.otelExporter` to confirm the restore:

   ```bash
   oc get guardrailsorchestrator "$GORCH_NAME" -n "$NS" -o jsonpath='{.spec.otelExporter}{"\n"}'
   ```

   If the patch was successful, the output is similar to the following example, and the old keys (`protocol`, `otlpEndpoint`, `otlpExport`) are gone:

   ```
   {"enableMetrics":true,"enableTraces":true,"otlpProtocol":"grpc","otlpTracesEndpoint":"http://my-otelcol-collector...:4317"}
   ```

   If the old keys are still present, the 3.5 CRD is not installed yet.

   **Note**  
   The running pod keeps the OpenTelemetry settings from the 2.25 deployment, so traces continue to flow immediately after upgrade. This restore ensures the configuration survives a later reconcile of the CR.

6. Query the **info** endpoint of the **GuardrailsOrchestrator** service:

   ```bash
   export GORCH_NAME=<GORCH_NAME>
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

**Note**
Run this procedure on the **rhai-cli** pod (`rhai-cli-0`). The pod has the `rhai-cli` tool, the `oc` client (with cluster-admin), and your backups (under `BACKUP_DIR`), so every command below runs there directly — no `oc exec` and no `jq` are required. Set the environment variables in the same shell so they are available to the later steps.

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

3. Set the namespace, replacing **\<NAMESPACE\>** with a namespace that lost data:

   ```bash
   export NS=<NAMESPACE>
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

   If the output is **Ready**, skip ahead to the dry-run step.  
   If the output is other than **Ready**, continue to the next step:

6. Wait for the TrustyAIService to become ready:

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

7. Set the backup file path:

   ```bash
   export BACKUP_FILE=$(ls -t ${BACKUP_DIR}/trustyai-metrics-${NS}-*.json | head -1)
   echo "BACKUP_FILE=$BACKUP_FILE"
   ```

   The command should print the backup file path, as shown in the following example output:

   ```
   BACKUP_FILE=/tmp/rhoai-upgrade-backup/trustyai/trustyai-metrics-test-trustyaiservice-upgrade-20260218-175450.json
   ```

   **Note**  
   If no file is found, there is no backup for this namespace. Restart the steps in this procedure for the next namespace that lost data, if any.

8. Find the route label:

    ```bash
    oc get route -n "$NS" --show-labels
    ```

    Example output:

    ```
    NAME                    HOST/PORT                                                                                                PATH   SERVICES                          PORT          TERMINATION          WILDCARD   LABELS
    gaussian-credit-model   gaussian-credit-model-test-trustyaiservice-upgrade.<...>.openshiftapps.com          gaussian-credit-model-predictor   https         reencrypt/Redirect   None       inferenceservice-name=gaussian-credit-model
    trustyai-service        trustyai-service-test-trustyaiservice-upgrade.<...>.openshiftapps.com               trustyai-service-tls              oauth-proxy   reencrypt/Redirect   None       trustyai-service-name=trustyai-service
    ```

9. Set the route label by replacing **\<LABEL\_KEY\>=\<LABEL\_VALUE\>:** with your **TrustyAI service label pair:**

    ```bash
    export ROUTE_LABEL='<LABEL_KEY>=<LABEL_VALUE>'
    ```

    For example:

    ```bash
    export ROUTE_LABEL=trustyai-service-name=trustyai-service
    ```

    IMPORTANT: Ensure the route label that you specify belongs to the trustyai-service and is not a route that belongs to any other services or models.

    

10. Dry-run the restore. Pass the backup file with `--metrics-file` (without it, the action reports "No --metrics-file specified; nothing to restore" and does nothing) and the route label with `--metrics-route-label`:

    ```bash
    rhai-cli migrate run --migration trustyai.metrics --target-version 3.5.0 --metrics-file "$BACKUP_FILE" --metrics-route-label "$ROUTE_LABEL" --dry-run
    ```

    Example output:

    ```
      Auto-detected phase: post-upgrade
      Current OpenShift AI version: 3.5.0
      Target OpenShift AI version: 3.5.0
      Phase: post-upgrade

      WARNING: migration trustyai.metrics has phase pre-upgrade but effective phase is post-upgrade; proceeding because --migration was explicit

      trustyai.metrics:

      DRY RUN MODE: No changes will be made to the cluster

        → Restore TrustyAI scheduled metrics
        → Found 1 metric(s) in backup file
          ✓ Found 1 metric(s) in backup file
        → Restore metrics to test-trustyaiservice-upgrade
        → Would POST MEANSHIFT for model gaussian-credit-model to /metrics/drift/meanshift/request
          → Would POST MEANSHIFT for model gaussian-credit-model to /metrics/drift/meanshift/request
          → Would restore 1 metric(s) to test-trustyaiservice-upgrade
        → Restore metrics to test-trustyaiservice-db-upgrade
        → Would POST MEANSHIFT for model gaussian-credit-model to /metrics/drift/meanshift/request
          → Would POST MEANSHIFT for model gaussian-credit-model to /metrics/drift/meanshift/request
          → Would restore 1 metric(s) to test-trustyaiservice-db-upgrade
          ✓ Metrics restore complete

      Migration trustyai.metrics completed with skipped steps

      All migrations completed (some steps were skipped).
    ```

    The output lists each metric that the script would restore. The `Found N metric(s) to restore` and `Total metrics in backup` lines report how many metrics are in the backup. If the count is 0, there are no metrics to restore; restart this procedure for the next namespace that lost data, if any.

    **NOTE:** if the TrustyAI Service reports an **"UNKNOWN"** status, check to make sure that you selected the correct route label in Step 9\.

     If any metric has an **Unknown** metric type, it might not be supported in this version.  
      
11. Run the restore:

    ```bash
    rhai-cli migrate run --migration trustyai.metrics --target-version 3.5.0 --metrics-file "$BACKUP_FILE" --metrics-route-label "$ROUTE_LABEL"
    ```

**Verification**

* Verify that each metric shows **Successfully scheduled**. The summary at the end should show **Failed: 0**.

  If any metric fails, re-run the migration to skip already-restored metrics.

  Check the output for the HTTP code and response for each failure.

  Examples of common failures:

  Route not found: Double-check ROUTE\_LABEL from Step 9 in the procedure.  
  HTTP 400: Request body format may have changed between versions.

  HTTP 500: Model data may not be loaded yet. Check with:

  ```bash
  curl -sk "https://$(oc get route -n "$NS" -l "$ROUTE_LABEL" -o jsonpath='{.items[0].spec.host}')/info" -H "Authorization: Bearer $(oc whoami -t)"
  ```

* Verify that the restore count matches the backup:

  ```bash
  rhai-cli migrate run --migration trustyai.metrics --target-version 3.5.0 --metrics-file "$BACKUP_FILE" --metrics-route-label "$ROUTE_LABEL" --dry-run 2>&1 | tail -5
  ```

  Example output:
  ```
      ✓ Metrics restore complete

   Migration trustyai.metrics completed with skipped steps

   All migrations completed (some steps were skipped).
  ```

  The **Current scheduled metrics** count should be greater than or equal to the number of metrics reported in the dry-run (Step 10).

  **Note**  
  Restored metrics receive new UUIDs; original IDs from the backup are not preserved. Rerun the restore for other namespaces backed up.

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
