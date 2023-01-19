# Generate Rancher Chart files for Longhorn

## Purpose
When Longhorn releases a new version to the community, the most important customers use Rancher to upgrade. There are some steps to follow but not easy to understand the context. Therefore, I try to summarize the scenarios and steps with the simplest description for easy understanding.

## Cluster Explorer
The lastest chart management since Rancher 2.6

### Steps
 > This example targets Longhorn v1.4.0
### Upload all mirrored container images to [Docker Hub with Rancher Labs account](https://hub.docker.com/u/rancher) 
 1. If there is no corresponding repository on Docker Hub, you need to add a ticket to request to add, e.g., https://github.com/rancherlabs/eio/issues/1426
 2. Create PR to update image list on [Rancher Image-mirror Repo.](https://github.com/rancher/image-mirror/blob/master/images-list), e.g., [PR#329](https://github.com/rancher/image-mirror/pull/329)
 - List of mirror files as of version 1.4.0
    - Longhorn 
        - rancher/mirrored-longhornio-backing-image-manager
        - rancher/mirrored-longhornio-longhorn-engine
        - rancher/mirrored-longhornio-longhorn-instance-manager
        - rancher/mirrored-longhornio-longhorn-manager
        - rancher/mirrored-longhornio-longhorn-share-manager
        - rancher/mirrored-longhornio-longhorn-ui
        - rancher/mirrored-longhornio-support-bundle-kit (***New add since 1.4.0***)
    - CSI
        - rancher/mirrored-longhornio-csi-attacher
        - rancher/mirrored-longhornio-csi-node-driver-registrar
        - rancher/mirrored-longhornio-csi-provisioner
        - rancher/mirrored-longhornio-csi-resizer
        - rancher/mirrored-longhornio-csi-snapshotter
        - rancher/mirrored-longhornio-livenessprobe (***New add since 1.4.0***)
### Generate Rancher Chart files
 > Example [dev-2.7](https://github.com/rancher/charts/pull/2304) | [dev-2.6](https://github.com/rancher/charts/pull/2305)
 1. According to target Rancher version to fork corresponing develop branch, e.g., `dev-2.7`, from [Rancher Charts repo.](https://github.com/rancher/charts). But if want to relase an OOB, then the target branch should from release, e.g., `release-2.7`.
 2. There two stages: `Make Prepare & Patch` and `Make Charts` to follow.
 ### Stage 1: Make Prepare & Patch
 #### Longhorn
 1. Update the values in `/packages/longhorn/longhorn-1.4/package.yaml` includes
    - `commit`: value is the release commit id from [Longhorn Charts repo.](https://github.com/longhorn/charts)
    - `version`: the rule can reference [here](https://github.com/rancher/charts/pull/1428#issuecomment-915892789), taking `1.4.0` for `Rancher-2.7` as an example
        - Major
            - Rancher version is `2.7` then `101`
            - Rancher version is `2.6` then `100`
        - Minor: The continuation of `1.3.2` is numbered `1` in Rancher, so it is `2` in `1.4.0`
        - patch: Since this is the first release of `1.4` in Rancher `2.7`, it is `0`
 2. Run `PACKAGE=longhorn/longhorn-1.4 make prepare` generate staging chart files under `./package/longhorn/longhorn-1.4/charts` which pull from the commit that last step configured. Below files are the common must check first:
    - `/charts/templates/valid-install-crd.yaml`: Make sure includes new add CRDs. There are three CRDs introduced in Longhorn 1.4.0, so add three correspond conditions.
        ```yaml
        # {{- set $found "longhorn.io/v1beta2/SupportBundle" false -}}
        # {{- set $found "longhorn.io/v1beta2/SystemBackup" false -}}
        # {{- set $found "longhorn.io/v1beta2/SystemRestore" false -}}
        ```
    - `/charts/templates/userroles.yaml`: If there are new add CRDs, reference the `apiGroups: ["longhorn.io"]` in `/charts//templates/clusterrole.yaml` to include new resources
 3. Run `PACKAGE=longhorn/longhorn-1.4 make patch`, which will patched the files under `charts` folder and export the chages in `generated-changes` folder. There are four files should be chaged after this step
    - `/charts/README.md`: Since the user is performing the installation on Rancher, the `Installation` block will be removed, and the `Uninstallation` block will replace the corresponding description. 
    - `/charts/questions.yaml`
        - Ref. `/deploy/longhorn-images.txt` from [Longhorn repo.](https://github.com/longhorn/longhorn/blob/master/deploy/longhorn-images.txt) to check if include every image and correct version number
        - modify the image repo as mirrord image name, e.g., `longhornio/longhorn-manager` => `rancher/mirrored-longhornio-longhorn-manager`
    - `/charts/Chart.yaml`
        - Make sure include annotations for Rancher charts, especially check to include values ​​for `kube-version`, `rancher-version` and `upstream-version` fields
            ```yaml
            annotations:
                catalog.cattle.io/auto-install: longhorn-crd=match
                catalog.cattle.io/certified: rancher
                catalog.cattle.io/display-name: Longhorn
                catalog.cattle.io/kube-version: '>= 1.21.0-0'
                catalog.cattle.io/namespace: longhorn-system
                catalog.cattle.io/os: linux
                catalog.cattle.io/provides-gvr: longhorn.io/v1beta1
                catalog.cattle.io/rancher-version: '>= 2.7.0-0 < 2.8.0-0'
                catalog.cattle.io/release-name: longhorn
                catalog.cattle.io/type: cluster-tool
                catalog.cattle.io/ui-component: longhorn
                catalog.cattle.io/upstream-version: 1.4.0
            ```
        - `/charts/values.yaml`: make sure the images repos. are replaced as mirrored image

#### Longhorn-crd
 4. Update the values in `/packages/longhorn/longhorn-1.4/package.yaml` includes `commit` and `version`. Please reference Longhorn section step 1.
 5. Run `PACKAGE=longhorn-crd/longhorn-1.4 make prepare` generate staging chart files under `./package/longhorn-crd/longhorn-1.4/charts` which pull from the commit that step 1 configured. Below files are the common must check first:
    - `/charts/Chart.yaml`: Check the `appVersion` is the current version number
 6. Run `PACKAGE=longhorn-crd/longhorn-1.4 make patch`, which will patched the files under `charts` folder and export the chages in `generated-changes` folder. There are four files should be chaged after this step
    - `/charts/Chart.yaml`
        - Make sure include annotations for Rancher charts, especially check to include values ​​for `kube-version`, `rancher-version` and `upstream-version` fields
            ```yaml
            annotations:
              catalog.cattle.io/certified: rancher
              catalog.cattle.io/hidden: "true"
              catalog.cattle.io/namespace: longhorn-system
              catalog.cattle.io/release-name: longhorn-crd
            ```
 7. Commit and push with comment like `make prepare/patch: release longhorn v1.4.0 into Rancher 2.7 `

---

 ### Stage 2: Make Charts
 #### Longhorn
 1. Run `PACKAGE=longhorn/longhorn-1.4 make charts`, which will generate the chart files under `/charts/longhorn/101.2.0+up1.4.0` and packaging files under `/assets/longhorn/101.2.0+up1.4.0.tgz` that will be officially released and end user installed.
 2. Add released version in `release.yaml`
    ```yaml
    longhorn:
      - 101.2.0+up1.4.0
    ```
 #### Longhorn-crd
 3. Run `PACKAGE=longhorn-crd/longhorn-1.4 make charts`, which will generate the chart files under `/charts/longhorn-crd/101.2.0+up1.4.0` and packaging files under `/assets/longhorn-crd/101.2.0+up1.4.0.tgz` that will be officially released and end user installed.
 4. Add released version in `release.yaml`
    ```yaml
    longhorn-crd:
      - 101.2.0+up1.4.0
    ```
 
 5. Commit and push with comment like `make charts: release longhorn v1.4.0 into Rancher 2.7`
