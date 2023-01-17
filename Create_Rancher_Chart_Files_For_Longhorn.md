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
        - rancher/mirrored-longhornio-support-bundle-kit (**New add since 1.4.0**)
    - CSI
        - rancher/mirrored-longhornio-csi-attacher
        - rancher/mirrored-longhornio-csi-node-driver-registrar
        - rancher/mirrored-longhornio-csi-provisioner
        - rancher/mirrored-longhornio-csi-resizer
        - rancher/mirrored-longhornio-csi-snapshotter
        - rancher/mirrored-longhornio-livenessprobe (**New add since 1.4.0**)
### Generate Rancher Chart files
 > Example [dev-2.6](https://github.com/rancher/charts/pull/2304) [dev-2.7]
 1. According to target Rancher version to fork corresponing branch from 