Getting started
===

Add Certificate Health Check to ArgoCD (✅ Recommended Enhancement - Best Practice)
---

To ensure ArgoCD waits for the Certificate to become Ready, apply the following configuration to your ArgoCD control plane.

⛳️ This step goes into the `argocd-cm` ConfigMap in the `argocd` namespace.


```sh
kubectl -n argocd patch configmap argocd-cm \
  --type merge \
  -p '{"data":{"resource.customizations.health.cert-manager.io_Certificate":"hs = {}\nif obj.status ~= nil and obj.status.conditions ~= nil then\n  for _, condition in ipairs(obj.status.conditions) do\n    if condition.type == \"Ready\" and condition.status == \"True\" then\n      hs.status = \"Healthy\"\n      hs.message = condition.message\n      return hs\n    end\n  end\nend\nhs.status = \"Progressing\"\nhs.message = \"Waiting for certificate to become ready\"\nreturn hs"}}'
```

Or, add `wait` condition in ArgoCD:

```sh
data:
  resource.customizations.health.cert-manager.io_Certificate: |
    hs = {}
    if obj.status ~= nil and obj.status.conditions ~= nil then
      for _, condition in ipairs(obj.status.conditions) do
        if condition.type == "Ready" and condition.status == "True" then
          hs.status = "Healthy"
          hs.message = condition.message
          return hs
        end
      end
    end
    hs.status = "Progressing"
    hs.message = "Waiting for certificate to become ready"
    return hs
```

This makes ArgoCD wait for `Certificate` to become `Ready` before moving to next sync wave (Ingress).

After patching, restart the ArgoCD server:

```sh
kubectl -n argocd rollout restart deployment argocd-server
```

Deploy ArgoCD Apps🚀
===

You apply this once:

```sh
kubectl apply -f root-app.yaml
```

After that, ArgoCD:
* Reads the `appsets/` folder

* Applies each `ApplicationSet` to generate apps for each environment

* Recursively manages your full environment