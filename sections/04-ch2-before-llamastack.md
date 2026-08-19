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

