We discussed the possibility of using the API directly instead of the ArgoCD CLI to get application manifest differences.

Replicating the ArgoCD CLI's functionality for `app diff` would involve a significant amount of code. Argocd cli makes multiple calls to fetch manifests of both previous and running application versions based on source revisions and then does local diff of these manifests. It seems more practical to continue using the ArgoCD CLI, as this allows us to easily incorporate new features and updates from the open-source project.
