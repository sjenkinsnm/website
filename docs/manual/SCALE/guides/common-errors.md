---
title: Common Errors and Solutions
sidebar_position: 17
---

## General Errors

### nil pointer evaluating interface {}.mode

`[EFAULT] Failed to install chart release: Error: INSTALLATION FAILED: template: APPNAME/templates/common.yaml:1:3: executing "APPNAME/templates/common.yaml" at : error calling include: template: APPNAME/charts/common/templates/loader/_all.tpl:6:6: executing "tc.v1.common.loader.all" at : error calling include: template: APPNAME/charts/common/templates/loader/_apply.tpl:47:6: executing "tc.v1.common.loader.apply" at : error calling include: template: APPNAME/charts/common/templates/spawner/_pvc.tpl:25:10: executing "tc.v1.common.spawner.pvc" at : error calling include: template: APPNAME/charts/common/templates/lib/storage/_validation.tpl:18:43: executing "tc.v1.common.lib.persistence.validation" at <$objectData.static.mode>: nil pointer evaluating interface {}.mode`

Issue: This error is due to old version of Helm. Helm > 3.9.4 is required.

Solution:

Upgrade to Cobia (23.10.x). (System Settings -> Update. Select Cobia from the dropdown)
SCALE Bluefin and Angelfish releases are no longer supported.

Support Policy: https://truecharts.org/manual/SUPPORT

---

### cannot patch "APPNAME-redis" with kind StatefulSet

`[EFAULT] Failed to update App: Error: UPGRADE FAILED: cannot patch "APPNAME-redis" with kind StatefulSet: StatefulSet.apps "APPNAME-redis" is invalid: spec: Forbidden: updates to statefulset spec for fields other than 'replicas', 'ordinals', 'template', 'updateStrategy', 'persistentVolumeClaimRetentionPolicy' and 'minReadySeconds' are forbidden`

Solution:

To check which have statefulsets:

```bash
k3s kubectl get statefulsets -A | grep "ix-"
```

Then to delete the statefulset:

```bash
k3s kubectl delete statefulset STATEFULSETNAME -n ix-APPNAME
```

Example:

```bash
k3s kubectl delete statefulset nextcloud-redis -n ix-nextcloud
```

Once deleted you can attempt the update (or if you were already updated to latest versions, then edit and save without any changes)

## Operator-Related Errors

### service "cnpg-webhook-service" not found

`[EFAULT] Failed to update App: Error: UPGRADE FAILED: cannot patch "APPNAME-cnpg-main" with kind Cluster: Internal error occurred: failed calling webhook "mcluster.cnpg.io": failed to call webhook: Post "https://cnpg-webhook-service.ix-cloudnative-pg.svc/mutate-postgresql-cnpg-io-v1-cluster?timeout=10s": service "cnpg-webhook-service" not found`

Solution:

- Enter the following command

```bash
k3s kubectl delete deployment.apps/cloudnative-pg --namespace ix-cloudnative-pg
```

- Update `cloudnative-pg` to the latest version, or if you already on the latest version, edit `cloudnative-pg` and save/update it again without any changes.
- If the app stays stopped, hit the start button in the UI for `cloudnative-pg`

### "monitoring.coreos.com/v1" ensure CRDs are installed first

`[EFAULT] Failed to update App: Error: UPGRADE FAILED: unable to build kubernetes objects from current release manifest: resource mapping not found for name: "APPNAME" namespace: "ix-APPNAME" from "": no matches for kind "PodMonitor" in version "monitoring.coreos.com/v1" ensure CRDs are installed first`

Solution:

- Install `prometheus-operator` first, then go back and install the app you were trying to install.
- If you see this error with prometheus-operator already installed, then delete and reinstall.
- While deleting prometheus-operator, if you encounter the error:

`Error: [EFAULT] Unable to uninstall 'prometheus-operator' chart release: b'Error: failed to delete release: prometheus-operator\n'`

- Run the following command from system shell as root:

```bash
k3s kubectl delete namespace ix-prometheus-operator
```

Then install `prometheus-operator` again. It will fail on the first install attempt, but the second time it will work.

### Rendered manifests contain a resource that already exists

#### certificaterequests.cert-manager.io

`[EFAULT] Failed to install App: Error: INSTALLATION FAILED: rendered manifests contain a resource that already exists. Unable to continue with install: CustomResourceDefinition "certificaterequests.cert-manager.io" in namespace "" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "cert-manager"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "ix-cert-manager"`

Solution:
The **cert-manager operator** is required for the use of cert-manager and clusterissuer to issue certificates for chart ingress.

To remove the previous automatically installed operator run this in the system shell as **root**:

```bash
k3s kubectl delete  --grace-period 30 --v=4 -k https://github.com/truecharts/manifests/delete4
```

https://truecharts.org/manual/FAQ#cert-manager

#### backups.postgresql.cnpg.io

`[EFAULT] Failed to install App: Error: INSTALLATION FAILED: rendered manifests contain a resource that already exists. Unable to continue with install: CustomResourceDefinition "backups.postgresql.cnpg.io" in namespace "" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "cloudnative-pg"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "ix-cloudnative-pg"`

Solution:
The **cloudnative-pg operator** is required for the use of any charts that utilize CloudNative Postgresql (CNPG).

:::warning DATA LOSS

The following command is destructive and will delete any existing CNPG databases.

Run the following command in system shell as **root** to see if you have any current CNPG databases to migrate:

```bash
k3s kubectl get cluster -A
```

Follow [this guide](https://truecharts.org/manual/SCALE/guides/cnpg-migration-guide/) to safely migrate any existing CNPG databases.

:::

To remove the previous automatically installed operator run this in the system shell as **root**:

```bash
k3s kubectl delete  --grace-period 30 --v=4 -k https://github.com/truecharts/manifests/delete2
```

https://truecharts.org/manual/FAQ#cloudnative-pg

#### addresspools.metallb.io

`[EFAULT] Failed to install App: Error: INSTALLATION FAILED: rendered manifests contain a resource that already exists. Unable to continue with install: CustomResourceDefinition "addresspools.metallb.io" in namespace "" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "metallb"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "ix-metallb"`

Solution:
The **metallb operator** is required for the use of MetalLB to have each chart utilize a unique IP address.

:::warning LOSS OF CONNECTIVITY

Installing the MetalLB operator will prevent the use of the TrueNAS Scale integrated load balancer. Only install this operator if you intend to use MetalLB.

:::

To remove the previous automatically installed operator run this in the system shell as **root**:

```bash
k3s kubectl delete  --grace-period 30 --v=4 -k https://github.com/truecharts/manifests/delete
```

https://truecharts.org/manual/FAQ#metallb

#### alertmanagerconfigs.monitoring.coreos.com

`[EFAULT] Failed to install chart release: Error: INSTALLATION FAILED: rendered manifests contain a resource that already exists. Unable to continue with install: CustomResourceDefinition "alertmanagerconfigs.monitoring.coreos.com" in namespace "" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "prometheus-operator"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "ix-prometheus-operator"`

Solution:
The **prometheus-operator** is required for the use of Prometheus metrics and for any charts that utilize CloudNative Postgresql (CNPG).

To remove the previous automatically installed operator run this in the system shell as **root**:

```bash
k3s kubectl delete  --grace-period 30 --v=4 -k https://github.com/truecharts/manifests/delete3
```

https://truecharts.org/manual/FAQ#prometheus-operator

### Operator [traefik] has to be installed first

`Failed to install App: Operator [traefik] has to be installed first`

Solution:

If this error appears while installing Traefik, install Traefik with its own ingress disabled first.
Once it's installed you can enable ingress for traefik.

### Operator [cloudnative-pg] has to be installed first

`Failed to install App: Operator [cloudnative-pg] has to be installed first`

Solution:

Install `cloudnative-pg`.

:::tip

Ensure the operators train is enabled in the Truecharts catalog under Apps -> Discover Apps -> Manage Catalogs

:::

### Operator [prometheus-operator] has to be installed first

`Failed to install App: Operator [prometheus-operator] has to be installed first`

Solution:

Install `prometheus-operator`.

:::tip

Ensure the operators train is enabled in the Truecharts catalog under Apps -> Discover Apps -> Manage Catalogs

:::
