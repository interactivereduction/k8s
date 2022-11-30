# Kubernetes deployment for IR

Requirements:
-------------

- [Install k0sctl](https://github.com/k0sproject/k0sctl#installation)
- Access to the STFC cloud via an openstack account, with [setup environment variables](https://stfc-cloud-docs.readthedocs.io/en/latest/howto/CreateVMFromCommandLine.html#setting-up-the-environment-to-select-project) on the terminal of choice.
- Install conda (recommend mambaforge) for managing the k8s repo or install python-kubernetes, ansible, and all of the kubernetes management software (kubernetes-client, kuberentes-server, etc) into your system/distro.
- [Install Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli)
- [Install hubble Client](https://docs.cilium.io/en/v1.10/gettingstarted/hubble_setup/#install-the-hubble-client)

Conda env setup
---------------

To create a conda environment that can sustain development of this repository (excluding k0sctl and setup of k0s itself) you can run the following command, whilst in the repository:

```shell
conda env create -f k8s-conda-env.yml
```

Recommended installs:
---------------------

After installing the conda env, activate it and install this plugin to helm.

```shell
helm plugin install https://github.com/databus23/helm-diff
```

Cloud setup and deploy k0s via k0sctl in terraform:
---------------------------------------------------

You can achieve this by using terraform (included in the conda environment) from inside the terraform directory. `terraform apply` can fail due to cloud instability, if it does, just run it again. You need to have setup the environment variables for openstack listed in the requirements at the top of this file.

Ensure that the `terraform/main.tf` file uses your FedId for the SSH Key name, and the key is up to date with yours.

```shell
terraform init
terraform apply
```

Setup an ssh-agent for connecting to the cluster with k0sctl. Example:

```shell
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

Use terraform to output the data and then apply that to construct the k0s cluster.

```shell
terraform output -raw k0s_cluster | k0sctl apply --no-wait --config -
```

Using terraform output to help setup dev env
--------------------------------------------

- Export the kubeconfig to the top of the repository, whilst in the terraform directory

```shell
terraform output -raw k0s_cluster | k0sctl kubeconfig --config - > ../kubeconfig
```

Use terraform to output the ansible inventory into your ansible directory

```shell
terraform output -raw ansible_inventory > ../ansible/inventory.ini
```

- Export KUBECONFIG

```shell
export KUBECONFIG=/path/to/repository/kubeconfig
```

Add the k8s componenets needed for IR
-------------------------------------

Navigate to the ansible playbook directory and run the playbook for deploying K8s tools such as Traefik, Cilium, Longhorn, Prometheus etc.

```shell
ansible-playbook deploy-k8s-services.yml --ask-vault-password
```

Gotchas
-------

- `terraform apply` struggles with creating all the openstack VMs, this happens when doing it manually and is not related to terraform, it is due to cloud instability.
- `terraform destroy` struggles with the load balancer, you can help it along by deleting the load balancer manually in the GUI whilst running it.
