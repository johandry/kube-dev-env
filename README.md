# Kubernetes Development Environment

A Kubernetes development environment (**KuDE**) isolated to install, develop and tests Kubernetes from the [Github source code](https://github.com/kubernetes/kubernetes), using a Vagrant Box. This Vagrant Box or Environment is for developers contributing the Kubernetes project.

Using a Kubernetes development environment using a Vagrant Box allows to:

1. Automate the provisioning process, ensuring to have the same environment every time it is re-created
2. Destroy the environment when it's not in use, to release resources. The environment will be exactly the same the next time it's created
3. Use the same operative system (Ubuntu) and configuration by all the developers
4. Use the IDE of our preference on your own computer
5. Use the environment only to build and test your changes. You can turn it up or down whenever you want.

- [Kubernetes Development Environment](#kubernetes-development-environment)
  - [Forking](#forking)
  - [Requirements](#requirements)
  - [Quick Install](#quick-install)
  - [Build your own environment](#build-your-own-environment)
  - [Git workflow](#git-workflow)
  - [Build Kubernetes](#build-kubernetes)
  - [Testing Kubernetes](#testing-kubernetes)
  - [Alternatives](#alternatives)
  - [Credits](#credits)

## Forking

The first step to contribute to Kubernetes is to fork the Kubernetes projects on your Github account. Go to each of te following projects and click on the Fork button in the top-right corner.

- **Kubernetes code**: https://github.com/kubernetes/kubernetes
- **Kubernetes Website and Documentation**: https://github.com/kubernetes/website
- **Kubernetes Testing Infrastructure**: https://github.com/kubernetes/test-infra

To save time provisioning the environment, it's recommended to clone the forked Kubernetes projects mentioned above into the directory `./shared_folder/k8s.io`. Like this:

```bash
mkdir -p shared_folder/k8s.io
cd shared_folder/k8s.io

git clone https://github.com/<github_username>/kubernetes
git clone https://github.com/<github_username>/website
git clone https://github.com/<github_username>/test-infra
```

## Requirements

The requirements on any operative system are:

- **[Virtual Box](https://www.virtualbox.org/wiki/Downloads)** and the **Extension Pack** version 6 or higher
- **[Vagrant](https://www.vagrantup.com/downloads)** version 2.2 or higher

Specifically on **macOS X**, you can install everything using `brew` executing this:

```bash
brew install virtualbox virtualbox-extension-pack
brew install vagrant
```

On **Ubuntu**, you need nothing if you don't want an isolated environment.

## Quick Install

Having forked the Kubernetes projects and installed all the requirements for an isolated environment you can have the Kubernetes Development Environment following these instructions:

1. Create a directory to be use as workspace. Example: `./kude`
2. Export the environment variable `GITHUB_USER` with your Github username or organization name.
3. To save time provisioning the environment, it's recommended to clone the forked Kubernetes projects mentioned above into the directory `./kude/shared_folder/k8s.io`.
4. Download the `Vagrantfile` either from [here](https://raw.githubusercontent.com/johandry/kube-dev-env/main/Vagrantfile) or using `curl`. You can also clone the repo but the `Vagrantfile` is all you need.
5. Execute `vagrant up` to provision the environment
6. Use your favorite IDE to open the Kubernetes projects, and to follow the Git workflow explained below.
7. Execute `vagrant ssh` to login to the Vagrant box
8. If you don't need the environment and want to release resources, execute `vagrant destroy`

Or, executing on a terminal the following commands:

```bash
mkdir -p ./kude/shared_folder/k8s.io

export GITHUB_USER=johandry

cd ./kude
curl -fsL https://raw.githubusercontent.com/johandry/kube-dev-env/main/Vagrantfile

cd ./shared_folder/k8s.io
git clone https://github.com/${GITHUB_USER}/kubernetes
git clone https://github.com/${GITHUB_USER}/website
git clone https://github.com/${GITHUB_USER}/test-infra

vagrant up

vagrant ssh
```

<!--
On **Ubuntu**, for a non-isolated environment just need to execute `curl -fsL http://www.johandry.com/kude/setup.sh | bash`. This will setup the environment in your system.
-->

## Build your own environment

1. Clone this repository or download just the [Vagranfile](http://www.johandry.com/kude/Vagrantfile), then modify the Vagrantfile. The main sections in the file are:

   - **Parameters**: They are self-descriptive variables located at the top of the file.
   - **Setup/Provisioning Script**: It is located after the parameters in the variable `PROVISION_NODE`. These are all the shell instructions to execute to setup or provisioning the vagrant box instance or node when it boot or when you execute `vagrant up`.
     - Try to make it idempotent.
     - Exit with non-zero when a validation fail
     - All these instructions are executed as `root`.
   - **Network Settings**: It is the last section inside teh block `Vagrant.configure` ... `end`. This block contain Vagrant instructions to setup the node. You can find information on the [Vagrant Documentation](https://www.vagrantup.com/docs/vagrantfile/machine_settings).

2. Export the following environment variables:

   - **GITHUB_USER**: Your Github username or organization where you forked the Kubernetes projects. The Github URL should be like: `github.com/{GITHUB_USERNAME}`
   - `FULLNAME`: (Optional) Your full name like it is assigned to your Github user. It's used to configure `git` and it's used for your commits. If not provided, the script will get it using the `git config` command.
   - `EMAIL`: (Optional) Your email address like it is assigned to your Github user. It's used to configure `git` and it's used for your commits. If not provided, the script will get it using the `git config` command.

3. Execute the following instructions, it includes the download of the `Vagrantfile`, and the `export` of the above variables:

   ```bash
   mkdir -p kude/shared_folder
   cd kude
   curl -fsL https://raw.githubusercontent.com/johandry/kube-dev-env/main/Vagrantfile

   export GITHUB_USER=johandry

   vagrant up

   cd shared_folder
   ```

To test the environment, login into the Vagrant box and verify you have all the important requirements, or execute `test.sh`, like this:

```bash
vagrant ssh

./test.sh
```

If something fail in your build, you may want to cleanup everything and start over.

```bash
vagrant destroy
```

## Git workflow

You must sign the [Contributor License Agreement](https://git.k8s.io/community/contributors/guide/README.md#sign-the-cla) in order to contribute.

1. Go to https://github.com/kubernetes/kubernetes/issues to find issues to work on.
2. Take ownership of the issue. Make a comment on the issue you found using the tag `#dibs` or `I would like to work on this`.
3. Sync your fork's master.
4. Create a feature branch and begin work.
5. Sync as you work and iterate.
6. Pull request back to `github.com/kubernetes/kubernetes`
7. Go to step 5 until it's finished and accepted.
8. Merge.

Some important comments:

- Don't by shy, ask if you have questions
- If you are not ready to fix the issue, tag it as `#investigating`. You will not be assigned but will let others know you might call it as `#dibs`

You can find more information about this process in the [CONTRIBUTING](https://github.com/kubernetes/kubernetes/blob/master/CONTRIBUTING.md) document of each Kubernetes project or SIG.

You can work from your computer, on whatever operative system you are, on the directory `kude/shared_folder/k8s.io/`. This directory is shared in your Vagrant box or development environment on the directory `~/shared_folder/k8s.io`

## Build Kubernetes

To build Kubernetes login to the Vagrant box and execute `make`, like this:

```bash
vagrant ssh
sudo make all
```

When you use `vagrant ssh` you login as the user `vagrant`, so it's required to use `sudo` when executing `make`.

You can also use the parameter `WHAT` to build just one package or executable, like this:

```bash
make WHAT=cmd/kubelet
```

Other useful Makefile rules are:

- `clean`: remove built binaries
- `release`: generate a release
- `release-skip-tests`: generate a release without running tests
- `help`: print all the rules

Example:

```bash
sudo make clean
sudo make release-skip-tests WHAT=cmd/kubelet
```

## Testing Kubernetes

It's required to run the tests before committing any code change, depending of the change is the test you'd like to execute.

The tests are also executed using `make` using the following rules (don't forget to use `sudo`):

- `verify`: Verification tests. Required before pushing a PR
- `test`: Unit tests. You can use the parameter `WHAT` to test a package. Example: `WHAT=./pkg/api/pod`
- `test-integration`: Integration tests. You can use the parameter `WHAT` to test an existing integration tests. Example: `WHAT=./test/integration/kubelet`

You can also use the parameter `GOFLAGS="-v"` to tests in verbose mode. For example:

```bash
sudo make test-integration WHAT=./test/integration/pods GOFLAGS="-v"
```

## Alternatives

The alternatives to this project would be:

- **KinD** which uses Docker instead of Virtual Box. KinD may consume less resources than KuDE and may provision the environment quicker (Containers vs VMs).
- **VM on a Cloud** can be setup using the same provisioning script but requires access to internet and will cost you. However, the performance may be better as you are not consuming your computer resources.

## Credits

Thanks to [Mike Brown](https://twitter.com/mikebrow) for the mentoring and instructions to setup a VM to contribute to Kubernetes projects. The process explained here are from his repo and the Kubernetes project repos.
