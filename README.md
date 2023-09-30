# Notes
Create a secret in your tap-install namespace with a Git HTTP Token and overlay for allowing the supply chain to commit to a GitOps repo.
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: workload-git-http
  namespace: tap-install
stringData:
  content.yaml: |
    git:
      url: https://github.com
      username: USERNAME
      password: TOKEN
---
apiVersion: v1
kind: Secret
metadata:
  name: workload-git-auth-overlay
  namespace: tap-install
stringData:
  workload-git-auth-overlay.yaml: |
    #@ load("@ytt:overlay", "overlay")
    #@overlay/match by=overlay.subset({"apiVersion": "v1", "kind": "ServiceAccount","metadata":{"name":"default"}}), expects="0+"
    ---
    secrets:
    #@overlay/append
    - name: git-token
```

Can only use the OOTB basic supply chain without writing your own custom supply chain:
```yaml
supply_chain: basic
ootb_supply_chain_basic:
  carvel_package:
    workflow_enabled: true
    name_suffix: vmware.com #! Optional
  #! Configure the rest of this to point to whatever GitOps repo you created already
  gitops:
    server_address: https://github.com
    repository_owner: cdelashmutt-pivotal
    repository_name: rollback-with-package-demo-gitops
    ssh_secret: git-token #! This is the secret we created above
    branch: main
    commit_message: "Update from TAP Supply Chain Choreographer"
```

You'll need to add a secret for PackageInstall values, and a PackageInstall object YAML to your gitops repo like the process at https://docs.vmware.com/en/VMware-Tanzu-Application-Platform/1.6/tap/scc-delivery-with-carvel-app.html.

Apply the "dev-namespace-app" to whatever namespace you want to install your package to, or modify it to manage that app however you like.