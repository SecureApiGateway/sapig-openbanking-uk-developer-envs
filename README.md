This repository holds Kustomise artifacts for deploying the ForgeRock platform into developer environments. It holds no value for customers trying to use the Secure API Gateway and is solely related to the core devs developer workflow.

It also holds a set `cs_config_manager` config for the nightly env.

# Kustomize

This folder provides [Kustomize](https://kubectl.docs.kubernetes.io/pages/app_customization/introduction.html) artifacts for deploying the ForgeRock platform.

If you are not familiar with Kustomize, please read the document link above - the explanation below will make a lot more sense.

TL;DR; - Kustomize is based on patching (json patch and strategic merge patch) and overlays.
You create base assets (K8S yaml files), and patch those. Those in turn can be used as a new base, and so on. You can nest these to any
arbitrary depth.

## Organization

The base directory folder includes the products (am, idm, ig, ds) and the "overlay" folder includes the environments.
Environments pull together the products into a kustomize deployment. See `./overlay/{version}/all` for an example.

## Viewing the Kustomize output

You can use `kubectl`  (version 1.14 or higher) or `kustomize`


```bash
cd kustomize/overlay/{version}/all
# This will show you what is sent to the cluster
kustomize build
```

## Prometheus
IG comes bundled with prometheus support, the details of which can be found [here](https://backstage.forgerock.com/docs/ig/7/maintenance-guide/monitoring-eps.html). The `dev` overlay comes with an example on how to protect the ig metrics endpoint with basic auth. A simple [secret](../base/ig/secret.yaml) is used in the example with an exposed user/pass credentials. You wouldnt have this secret exposed in github like this for a production environment, you would probably use a secrets manager to create the necessary secret.

The created secret is referenced in our deployment [manifest](../base/ig/deployment.yaml). The environment variables created from that secret is interpolated by our [admin config](../../config/7.1.0/securebanking/ig/config/dev/config/admin.json). In this example `&{ig.metrics.username}` maps to the environment variable `IG_METRICS_USERNAME` and so on.

## Images

The images referenced in the kustomize files are generic (example: `am`, `ig`), and not
specific to a registry ( `gcr.io/forgerock-io/am:7.1.0.1` ).

We can not directly deploy these generic images, because we need a docker image
that has the configuration "baked in". This is where skaffold comes in to the picture.
Skaffold will build new docker images that include configuration, and will
"fix up" the docker image tags in kustomize, replacing the generic names (`am`) with
a specific image name, tagged with a sha hash (dev mode) or a git hash (prod).

See the skaffold [../README.md](../README.md).

## Further Discussion

 * This demonstrates a directory based organization. You can also use git branching. See [this discussion](https://kubectl.docs.kubernetes.io/pages/app_composition_and_deployment/diffing_local_and_remote_resources.html)
