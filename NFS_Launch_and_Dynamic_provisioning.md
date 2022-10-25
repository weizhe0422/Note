 # NFS Server: Installation and Dynamic Provisioning
 ## Steps
 1. Launch a VM with cloud-init file: `multipass launch -c 5 -m 5G -d 50G -n nfs-server --cloud-init cloud-init.yaml `
   - cloud-init.yaml
        ```yaml
        ssh_authorized_keys:
        - <generate-by-yourself> # ssh-keygen -t rsa -b 2048 -> /home/<user-name>/.ssh/id_rsa.pub
        packages:
        - open-iscsi
        - nfs-common
        ```
 2. Set up nfs-server environment
     - Enter into server: `multipass ssh nfs-server`
     - Install `nfs-kernel-server` package: `sudo apt install nfs-kernel-server`
     - Create folders for export
         - `sudo mkdir /srv/kubernetes/`
         - `sudo mkdir /srv/kubernetes/volumes/`
         - `sudo mkdir /srv/kubernetes/backup/`
     - Modify the folders' permission and owner
         - `sudo chown -R nobody:nogroup /srv/kubernetes/`
         - `sudo chmod -R 777 /srv/kubernetes/`
     - Edit `/etc/exports`: add `/srv/kubernetes/   10.121.98.0/24(rw,no_root_squash)`
         - IP should be same as the `nfs-server` from `ip a`
     - Restart nfs service: `sudo systemctl restart nfs-server`
 3. Mount the nfs folder in master node
     - Enter into master node: `multipass shell c1-control`
     - Install `nfs-common` package: `sudo apt-get install nfs-common -y`
     - Mount the NFS folder: `sudo mount -t nfs 10.121.98.59:/srv/kubernetes/ /mnt`
 4. Build [NFS Subdirectory External Provisioner](https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/)
     - Add Helm Repo: `helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/`
     - Update Helm repo.: `helm repo update`
     - Go to [How to deploy NFS Subdir External Provisioner to your cluster](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner#how-to-deploy-nfs-subdir-external-provisioner-to-your-cluster)
    - Install provisioner: `helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=10.121.98.59 --set nfs.path=/srv/kubernetes/ -n nfs-system --create-namespace`
    - Check the provisioner and storage class ready
       - `kubectl get pods -n nfs-system`
       - `kubectl get sc -n nfs-system`
 5. Go to Longhorn UI -> Setting -> Backup Target, and set it as `nfs://10.121.98.59:/srv/kubernetes/backup/`
