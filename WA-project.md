# Getting Started with Kubernetes

1. We need to ask Javier for access to our namespace at UCSD: `cms-ml`. Once you've been added, log into the [NRP Nautilus](https://portal.nrp-nautilus.io/) website with your student login info (after finding UCSD).

2. Follow the getting started instructions. Basically,
  * Follow `kubectl` command line installation.
  * Add the `config` file in a local `~/.kube/` folder.
  * Check that no erros are raised from the command: `kubectl config get-contexts`.

3. If no errors above, you can check the current running pods in our namespace `cms-ml` with the command: `kubectl get pods -n cms-ml`. It should output the list of pods with running jobs. 
