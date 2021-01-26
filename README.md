# MailDev Helm Charts

[Helm](https://helm.sh) repo for different charts related to MailDev which can be installed on [Kubernetes](https://kubernetes.io)

## Add Helm repository

To install the repo just run:

```bash
helm repo add maildev https://splitt3r.github.io/helm/
helm repo update
```

## Helm Charts

* [maildev](https://splitt3r.github.io/helm/)

  ```bash
  helm install my-release maildev/maildev
  ```
