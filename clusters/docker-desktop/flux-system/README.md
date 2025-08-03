
# monitor git repository for changes
# for Production deploy if changes are made in main branch

flux create source git demo-python-app \
  --url=https://github.com/saarwasserman/demo-python-app.git \
  --branch=dev \
  --interval=1m

# kustomize resource to point to the git repository desired overlay

flux create kustomization demo-python-app \
  --source=GitRepository/demo-python-app \
  --path="deploy/overlays/dev" \
  --prune=true \
  --interval=5m


  # watch for new images in the registry

flux create image repository demo-python-app-images \
    --image docker.io/saarwasserman/demo-python-app --interval 5m


flux create image policy demo-python-app-policy \
  --image-ref=demo-python-app-images \
  --reflect-digest=Always \
  --interval=2m \
  --select-semver=">=0.0.0"


flux create image update demo-python-app-auto-update \
  --image-repository=demo-python-app-images \
  --update-strategy=Setters \
  --interval=2m \
  --namespace=flux-system


  ### Installing the image automation controller

  flux bootstrap github \
  --token-auth \
  --owner=saarwasserman \
  --repository=fleet-infra \
  --branch=main \
  --path=clusters/docker-desktop \
  --personal \
--components-extra image-reflector-controller,image-automation-controller



# permission to flux for dockerhub

kubectl create secret docker-registry flux-dockerhub-secret \
  --namespace flux-system \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=<your-docker-username> \
  --docker-password=<your-docker-access-token> \
  --docker-email=<your-email>
