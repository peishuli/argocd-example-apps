# Helm + Kustomize

Sometimes Helm chart don't have all required parameters and additional customization is required. This example application demonstrates how to combine Helm and Kustomize and use it
as a config management plugin in Argo CD.

Use following steps to try the application:

* configure `kustomized-helm` tool in `argocd-cm` ConfigMap:

```yaml
  configManagementPlugins: |
    - name: kustomized-helm
      init:
        command: ["/bin/sh", "-c"]
        args: ["helm init --client-only && helm dependency build"]
      generate:
        command: [sh, -c]
        args: ["helm template . > all.yaml && kustomize build"]
```

* create application using `kustomized-helm` as a config management plugin name:


```
argocd app create kustomized-helm \
    --config-management-plugin kustomized-helm \
    --repo https://github.com/argoproj/argocd-example-apps \
    --path plugins/kustomized-helm \
    --dest-server https://kubernetes.default.svc \
    --dest-namespace default
```
**_Note_**: From Alex Collins' response on Slack:
"it's probably worth noting that in v1.2 you can use environment variables that are set in your app, and passed to your tool":
```
# plugin specific config
    plugin:
      name: mypluginname
      # environment variables passed to the plugin
      env:
        - name: FOO
          value: bar
```
