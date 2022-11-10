## Update from upstream

Updating from upstream requires `kustomize` (https://github.com/kubernetes-sigs/kustomize), `yq` (https://github.com/mikefarah/yq) and `skopeo` (https://github.com/containers/skopeo).

### Make container images available

- Look for images in the `install.yaml` in the upstream release (For example: https://github.com/caicloud/event_exporter/blob/v1.0.0/deploy/deploy.yml)
  - For example: `grep -i ' image: ' deploy.yml`
  - Add any images not already retagged to [retagger](https://github.com/giantswarm/retagger)
    - The name rewrite rules are in `helm/caicloud-event-exporter-app/values.yaml` under `image`
    - You can double-check if the images have been retagged for example in Quay: https://quay.io/organization/giantswarm?tab=repos
- You can use `skopeo` to find the right sha digests (if they are retagged via digest and not by tag/version patters e.g.: `pattern: '>= v1.0.0'`):
  ```shell
  skopeo inspect --format "{{.Digest}}" --override-arch=amd64 --override-os=linux docker://docker.io/caicloud/event-exporter:v1.0.0
  ```

### Update helm templates

- Prepare CRD
  - Update event_exporter release version under `resources` in the `hack/kustomization.yaml` file
  - Execute `kubectl kustomize hack > helm/caicloud-event-exporter-app/templates/install.yaml`
    - Please double-check that the container images have been replaced to
      something like `'{{ .Values.image.repository }}:{{ .Values.image.tag }}'`
      If not then probably they updated the image source in upstream and you
      need to align the image rules in `hack/kustomization.yaml` under
      `images`. If so, please rerun the above command!
  - Execute `sed -i -e "/image:/b;s/'{{/{{/g" -e "/image:/b;s/}}'/}}/g" helm/caicloud-event-exporter-app/templates/install.yaml` to search and replace `'{{` with `{{` and `}}'` with `}}` in `helm/flux-app/templates/install.yaml`. But not in lines containing `image:`
- Bump the `appVersion` in `helm/caicloud-event-exporter-app/Chart.yaml`
