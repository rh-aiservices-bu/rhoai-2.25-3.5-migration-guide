# **Migrate from OpenShift AI 2.25.10  (and later) to 3.5 (latest) DRAFT** 

#### **Migrate from Red Hat OpenShift AI 2.25.10 (and later) to OpenShift AI 3.5 (latest)**  

## **Table of Contents**

---

[Preface](#preface)

[Chapter 1\. Assess and plan for migration](#chapter-1.-assess-and-plan-for-migration)

[1.1. Overview of OpenShift AI migration](#1.1.-overview-of-openshift-ai-migration)

[1.1.1. Overview of migration assessment steps](#1.1.1.-overview-of-migration-assessment-steps)

[1.2. Prerequisites for OpenShift AI migration](#1.2.-prerequisites-for-openshift-ai-migration)

[1.3 Deploy a persistent pod on your cluster that includes the](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image)   
[the rhai-cli container image](#1.3-deploy-a-persistent-pod-on-your-cluster-that-includes-the-the-rhai-cli-container-image)

[1.3.1 Log in to the cluster from within the pod](#1.3.1-log-in-to-the-cluster-from-within-the-pod)

[1.3.2. About the rhai-cli container image](#1.3.2.-about-the-rhai-cli-container-image)

[1.4. Run the migration assessment script](#1.4.-run-the-migration-assessment-script)

[1.4.1 Understand the migration assessment script output](#1.4.1-understand-the-migration-assessment-script-output)

[1.5 Submit the assessment output to Red Hat Technical Support](#1.5-submit-the-assessment-output-to-red-hat-technical-support)

[Chapter 2\. Before you upgrade](#chapter-2.-before-you-upgrade)

[2.1 Install the cert-manager Operator for Red Hat OpenShift](#2.1-install-the-cert-manager-operator-for-red-hat-openshift)

[2.2 Set Kueue management to Removed or Unmanaged](#2.2-set-kueue-management-to-removed)

[2.3. Model registry and catalog \- Before upgrade](#2.3.-model-registry-and-catalog---before-upgrade)

[2.4. Feature Store \- Before upgrade](#2.4.-feature-store---before-upgrade)

[2.5. Llama Stack / OGX (Open GenAI Stack) \- Before upgrade](#2.5.-llama-stack---before-upgrade)

[2.5.1. Llama Stack upgrade steps for cluster administrators](#2.5.1.-llama-stack-upgrade-steps-for-cluster-administrators)

[2.5.2. Llama Stack upgrade steps for LlamaStackDistribution resource owners](#2.5.2.-llama-stack-upgrade-steps-for-llamastackdistribution-resource-owners)

[2.6. AI Pipelines \- Before upgrade](#2.6.-ai-pipelines---before-upgrade)

[2.7. TrustyAI \- Before upgrade](#2.7.-trustyai---before-upgrade)

[2.7.1. TrustyAI \- Before upgrade \- Prepare for backup](#2.7.1.-trustyai---before-upgrade---prepare-for-backup)

[2.7.2. TrustyAI \- Before upgrade \- Backup metrics](#2.7.2.-trustyai---before-upgrade---backup-metrics)

[2.7.3. TrustyAI \- Before upgrade \- Backup data storage](#2.7.3.-trustyai---before-upgrade---backup-data-storage)

[2.7.4. TrustyAI \- Before upgrade \- Guardrails Orchestrator](#2.7.4.-trustyai---before-upgrade---guardrails-orchestrator)

[2.8. Workbenches \- Before upgrade](#2.8.-workbenches---before-upgrade)

[2.8.1. About upgrading your workbenches](#2.8.1.-about-upgrading-your-workbenches)

[2.8.2. Prepare your workbenches for migration](#2.8.2.-prepare-your-workbenches-for-migration)

[2.9. Ray Training Operator \- Before upgrade](#2.9.-ray-training-operator---before-upgrade)

[2.10. Model serving \- Before upgrade](#2.10.-model-serving---before-upgrade)

[2.10.1. Migration impact and scope](#2.10.1.-migration-impact-and-scope)

[2.10.2. Removed model serving configurations](#2.10.2.-removed-model-serving-configurations)

[2.10.3. Migration workflow for model serving](#2.10.3.-migration-workflow-for-model-serving)

[2.10.4. Prerequisites for model serving migration](#2.10.4.-prerequisites-for-model-serving-migration)

[2.10.5. Run the rhai-cli tool](#2.10.5.-run-the-rhai-cli-tool)

[2.10.6. Back up the inferenceservice-config ConfigMap](#2.10.6.-back-up-the-inferenceservice-config-configmap)

[2.10.7. Migrate InferenceServices to RawDeployment mode](#2.10.7.-migrate-inferenceservices-to-rawdeployment-mode)

[2.10.7.1 Convert Serverless InferenceServices to RawDeployment](#2.10.7.1-convert-serverless-inferenceservices-to-rawdeployment)

[2.10.7.2 Convert ModelMesh InferenceServices to RawDeployment](#2.10.7.2-convert-modelmesh-inferenceservices-to-rawdeployment)

[2.10.7.3 Verification of InferenceServices migration](#2.10.7.3-verification-of-inferenceservices-migration)

[2.10.8. Update the inferenceservice-config ConfigMap](#2.10.8.-update-the-inferenceservice-config-configmap)

[2.10.9. Update cluster configuration for migration](#2.10.9.-update-cluster-configuration-for-migration)

[2.10.10. Prepare distributed inference for migration](#2.10.10.-prepare-distributed-inference-for-migration)

[2.10.10.1. Install Red Hat Connectivity Link for distributed inference](#2.10.10.1.-install-red-hat-connectivity-link-for-distributed-inference)

[2.10.10.2. Configure Red Hat Connectivity Link for disconnected environments](#2.10.10.2.-configure-red-hat-connectivity-link-for-disconnected-environments)

[2.10.10.3. Configure authentication for LLMInferenceService resources](#2.10.10.3.-configure-authentication-for-llminferenceservice-resources)

[2.10.10.4. Freeze LLMInferenceService configuration for upgrade](#2.10.10.4.-freeze-llminferenceservice-configuration-for-upgrade)

[2.10.11. Verify migration readiness](#2.10.11.-verify-migration-readiness)

[2.11. Kubeflow Training Operator \- Before upgrade](#2.11.-kubeflow-training-operator---before-upgrade)

[2.12. OpenShift AI Operator \- Before upgrade](#2.12.-openshift-ai-operator---before-upgrade)

[Chapter 3\. Upgrade to 3.5](#chapter-3.-upgrade-to-3.5-latest)

[3.1. OpenShift AI Operator](#3.1.-openshift-ai-operator)

[Chapter 4\. After upgrading to 3.5](#chapter-4.-after-upgrading-to-3.5-latest)

[4.1. OpenShift AI Operator \- After upgrade](#4.1.-openshift-ai-operator---after-upgrade)

[4.2. AI hub registry and catalog \- After upgrade](#4.2.-ai-hub-registry-and-catalog---after-upgrade)

[4.3. Feature Store \- After upgrade](#4.3.-feature-store---after-upgrade)

[4.4. OGX (Open GenAI Stack, formerly Llama Stack) \- After upgrade](#4.4.-llama-stack---after-upgrade)

[4.5. AI Pipelines \- After upgrade](#4.5.-ai-pipelines---after-upgrade)

[4.5.1. Administrator tasks](#4.5.1.-administrator-tasks)

[4.5.2. Pipeline user tasks](#4.5.2.-pipeline-user-tasks)

[4.6. TrustyAI \- After upgrade](#4.6.-trustyai---after-upgrade)

[4.6.1. TrustyAI \- After upgrade \- Check Backups](#4.6.1.-trustyai---after-upgrade---check-backups)

[4.6.2. TrustyAI \- After upgrade \- Guardrails](#4.6.2.-trustyai---after-upgrade---guardrails)

[4.6.3. TrustyAI \- After upgrade \- Restore data](#4.6.3.-trustyai---after-upgrade---restore-data)

[4.6.4. TrustyAI \- After upgrade \- GPU deployment deadlock issue](#4.6.4.-trustyai---after-upgrade---gpu-deployment-deadlock-issue)

[4.7. Workbenches \- After upgrade](#4.7.-workbenches---after-upgrade)

[4.7.1. Migrate your workbenches after upgrade](#4.7.1.-migrate-your-workbenches-after-upgrade)

[4.7.2. Perform a deferred workbench image migration](#4.7.2.-perform-a-deferred-workbench-image-migration)

[4.8. Ray Training Operator \- After upgrade](#4.8.-ray-training-operator---after-upgrade)

[4.9. Model serving \- After upgrade](#4.9.-model-serving---after-upgrade)

[4.9.1. Finalize migration configuration](#4.9.1.-finalize-migration-configuration)

[4.9.2. Verifying upgrade completion and troubleshooting](#4.9.2.-verifying-upgrade-completion-and-troubleshooting)

[4.9.2.1. Verification steps](#4.9.2.1.-verification-steps)

[4.9.2.2.1. Serverless InferenceServices not converted before upgrade](#4.9.2.2.1.-serverless-inferenceservices-not-converted-before-upgrade)

[4.9.2.2.2. ModelMesh InferenceServices not converted before upgrade](#4.9.2.2.2.-modelmesh-inferenceservices-not-converted-before-upgrade)

[4.9.2.2.4. Authorino Operator not removed](#4.9.2.2.4.-authorino-operator-not-removed)

[4.9.2.2.5. Service Mesh v2 Operator not removed](#4.9.2.2.5.-service-mesh-v2-operator-not-removed)

[4.9.2.3. Additional resources](#4.9.2.3.-additional-resources)

[4.10. Kubeflow Training Operator \- After upgrade](#4.10.-kubeflow-training-operator---after-upgrade)

[5\. Clean up](#5.-clean-up)

[Legal Notice](#legal-notice)

[**Legal Notice**](#legal-notice)

# **Preface** {#preface}

Migrate from Red Hat OpenShift AI 2.25.10 (and later) to OpenShift AI 3.5.

