# Upgrading your Helm chart

This page covers best-practices in upgrading your JupyterHub deployment via updates
to the Helm Chart.

Upgrading from one version of the Helm Chart to the
next should be as seamless as possible, and generally shouldn't require major
changes to your deployment. Check the [CHANGELOG](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/CHANGELOG.md)
for each release to find out if there are any breaking changes in the newest version.

For additional help, feel free to reach out to us on [gitter](https://gitter.im/jupyterhub/jupyterhub)
or the [mailing list](https://groups.google.com/forum/#!forum/jupyter)!

## Major helm-chart upgrades

These steps are **critical** before performing a major upgrade.

1. Always backup your database!
2. Review the [CHANGELOG](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/CHANGELOG.md) for incompatible changes and upgrade instructions.
3. Update your configuration accordingly.
4. User servers may need be stopped prior to the upgrade,
   or restarted after it.
5. If you are planning an upgrade of a critical major installation,
   we recommend you test the upgrade out on a staging cluster first
   before applying it to production.

### v0.5 to v0.6

See the [CHANGELOG](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/CHANGELOG.md#06---ellyse-perry---2017-01-29).

### v0.4 to v0.5

Release 0.5 contains a major JupyterHub version bump (from 0.7.2 to 0.8).
Since it is a major upgrade of JupyterHub that changes how authentication is
implemented, user servers must be stopped during the upgrade.
The database schema has also changed, so a database upgrade must be performed.

See the [documentation for v0.5 for the upgrade process](https://zero-to-jupyterhub.readthedocs.io/en/latest/administrator/upgrading.html#v0-4-to-v0-5)
as well as the [CHANGELOG](https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/CHANGELOG.md#05---hamid-hassan---2017-12-05)
for this release for more information about changes.

## Subtopics

This section covers upgrade information specific to the following:

- `helm upgrade` command
- Databases
- RBAC (Role Based Access Control)
- Custom Docker images

### `helm upgrade` command

After modifying your `config.yaml` file according to the CHANGELOG, you will need
`<helm-release-name>` to run the upgrade commands. To find `<helm-release-name>`, run:

```
helm list --namespace <k8s-namespace>
```

Make sure to test the upgrade on a staging environment before doing the upgrade on
a production system!

To run the upgrade:

```
helm upgrade --cleanup-on-fail <helm-release-name> jupyterhub/jupyterhub --version=<chart-version> --values config.yaml --namespace <k8s-namespace>
```

For example, to upgrade to version `1.1.1` with a helm release name of `jhub` in the k8s namespace of `jhub`:

```
helm upgrade --cleanup-on-fail jhub jupyterhub/jupyterhub --version=1.1.1 --values config.yaml --namespace jhub
```

### Database

This release contains a major JupyterHub version bump (from 0.7.2 to 0.8). If
you are using the default database provider (SQLite), then the required db upgrades
will be performed automatically when you do a `helm upgrade`.

**Default (SQLite)**: The database upgrade will be performed automatically when you
[perform the upgrade](#upgrade-command)

**MySQL / PostgreSQL**: You will execute the following steps, which includes a manual update of your database:

1. Make a full backup of your database, just in case things go bad.
2. Make sure that the database user used by JupyterHub to connect to your database
   can perform schema migrations like adding new tables, altering tables, etc.
3. In your `config.yaml`, add the following config:

   ```yaml
   hub:
     db:
       upgrade: true
   ```

4. Do a [`helm upgrade`](#upgrade-command). This should perform the database upgrade needed.
5. Remove the lines added in step 3, and do another [`helm upgrade`](#upgrade-command).

### Custom Docker Images: JupyterHub version match

If you are using a custom built image, make sure that the version of the
JupyterHub package installed in it is now 0.8.1. It needs to be 0.8.1 for it to work with
v0.6 of the helm chart.

For example, if you are using `pip` to install JupyterHub in your custom Docker Image,
you would use:

```Dockerfile
RUN pip install --no-cache-dir jupyterhub==0.8.1
```

## JupyterHub versions installed in each Helm Chart

Each Helm Chart is packaged with a specific version of JupyterHub (and
other software as well). See the [Helm Chart repository](https://jupyterhub.github.io/helm-chart/) for
information about the versions of relevant software packages.

## Troubleshooting

If the upgrade is failing on a test system or a system that does not serve users, you can try
deleting the helm chart using:

```
helm delete <helm-release-name> --namespace <k8s-namespace>
```

`helm list --namespace <k8s-namespace>` may be used to find <helm-release-name>.
