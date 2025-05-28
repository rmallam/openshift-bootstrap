# OpenShift Bootstrap

This repository contains scripts and resources to bootstrap an OpenShift cluster with essential components and configurations.

## ArgoCD Installation

The `scripts` folder contains a script to install ArgoCD on your OpenShift cluster. Follow these steps to use it:

1. Ensure you have a running OpenShift cluster
2. Login to your cluster with cluster-admin privileges:
   ```bash
   oc login <cluster-url> -u <admin-username> -p <admin-password>
   ```
   or
   ```bash
   oc login --token=<token> --server=<cluster-url>
   ```

3. Run the ArgoCD installation script:
   ```bash
   cd scripts
   ./install-argocd.sh
   ```

4. Verify the installation:
   ```bash
   oc get pods -n openshift-gitops
   ```

## Installing Infrastructure Components

After setting up ArgoCD, the next step is to install infrastructure components like operators:

1. Create your infrastructure resources under the `infrastructure` folder, organizing them by component type (e.g., operators, configs).

2. Apply the infrastructure ApplicationSet to deploy all resources:
   ```bash
   oc apply -f infrastructure/argo-apps/infra-appset.yaml
   ```

3. This will create an Argo CD application that automatically deploys all resources defined in the infrastructure folder.

4. Verify the deployment in the ArgoCD dashboard:
   ```bash
   oc get route openshift-gitops-server -n openshift-gitops
   ```

## Infrastructure ApplicationSet

The `infra-appset.yaml` file uses GitOps principles to discover and deploy all components defined in your infrastructure directory. It:

- Automatically syncs with a Git repository
- Creates namespaces as needed
- Deploys all discovered components under the infrastructure path
- Self-heals if configurations drift

This also includes installing operators which is defined below in detail.

### Installing Operators

Installing operators is very easy with this GitOps approach:

1. Navigate to the `infrastructure/operators/<operator-name>` directory
2. Update the `values.yaml` file with the required operator configuration
3. Commit and push your changes to the Git repository
4. ArgoCD will automatically detect and apply these changes to your cluster

For example, a typical `values.yaml` file for an operator might look like this:

```yaml
# Example values.yaml for OpenShift Virtualization operator
subscription:
  channel: "stable"
  installPlanApproval: "Automatic"
  name: "kubevirt-hyperconverged"
  source: "redhat-operators"
  sourceNamespace: "openshift-marketplace"

# Additional operator configuration
operatorGroup:
  create: true
  targetNamespaces:
    - "openshift-cnv"
```

Simply update this file with the details of the operator you want to install, push to your Git repository, and the operator will be automatically deployed to your cluster.



## Additional Resources

- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [OpenShift GitOps Documentation](https://docs.openshift.com/container-platform/latest/cicd/gitops/understanding-openshift-gitops.html)

## Support

For issues or questions, please open an issue in this repository.