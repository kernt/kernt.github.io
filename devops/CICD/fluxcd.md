---
tags:
  - devops
  - flux
  - cicd
---

# Flux bootstrap Anpassungen

## Vorbereitungen für GitLAB

```sh
export GITLAB_TOKEN=<gl-token>
```
## GitLab Personal Account

```sh
flux bootstrap gitlab \
  --deploy-token-auth \
  --owner=my-gitlab-username \
  --repository=my-project \
  --branch=master \
  --path=clusters/my-cluster \
  --personal
```
## GitLab Groups

```sh
flux bootstrap gitlab \
  --deploy-token-auth \
  --owner=my-gitlab-group/my-gitlab-subgroup \
  --repository=my-project \
  --branch=master \
  --path=clusters/my-cluster
```
## [GitLab Deploy Keys](https://fluxcd.io/flux/installation/bootstrap/gitlab/#gitlab-deploy-keys)

If you want to bootstrap Flux using SSH instead of HTTP/S, you can set `--token-auth=false` and the Flux CLI will use the GitLab PAT to set a deploy key for your project.

When using SSH, the bootstrap command will generate a SSH private key. The private key is stored in the cluster as a Kubernetes secret named `flux-system` inside the `flux-system` namespace.

The generated SSH key defaults to `ECDSA P-384`, to change the format use `--ssh-key-algorithm` and `--ssh-ecdsa-curve`.

The SSH public key, is used to create a GitLab deploy key. By default, the GitLab deploy key is set to read-only access. If you’re using Flux image automation, you must give it write access with `--read-write-key=true`.

#### Deploy Key rotation

To regenerate the deploy key, delete the `flux-system` secret from the cluster and re-run the bootstrap command using a valid GitLab PAT
## [Bootstrap without a GitLab PAT](https://fluxcd.io/flux/installation/bootstrap/gitlab/#bootstrap-without-a-gitlab-pat)

For existing GitLab repositories, you can bootstrap Flux over SSH without using a GitLab PAT.

To use an SSH key instead of a GitLab PAT, the command changes to `flux bootstrap git`:

```shell
flux bootstrap git \
  --url=ssh://git@gitlab.com/<group>/<project> \
  --branch=<my-branch> \
  --private-key-file=<path/to/ssh/private.key> \
  --password=<key-passphrase> \
  --path=clusters/my-cluster
```

**Note** that you must generate an SSH private key and set the public key as a deploy key on GitLab in advance.

For more information on how to use the `flux bootstrap git` command, please see the generic Git server [documentation](https://fluxcd.io/flux/installation/bootstrap/generic-git-server/).
## Vorbereitungen für  Github

## [GitHub PAT](https://fluxcd.io/flux/installation/bootstrap/github/#github-pat)

For accessing the GitHub API, the bootstrap command requires a GitHub personal access token (PAT) with administration permissions.

The GitHub PAT can be exported as an environment variable:

```sh
export GITHUB_TOKEN=<gh-token>
```

If the `GITHUB_TOKEN` env var is not set, the bootstrap command will prompt you to type it the token.

You can also supply the token using a pipe e.g. `echo "<gh-token>" | flux bootstrap github`.

* [gitops](https://docs.gitlab.com/ee/user/clusters/agent/gitops/flux_tutorial.html)

## [GitHub Personal Account](https://fluxcd.io/flux/installation/bootstrap/github/#github-personal-account)

If you want to bootstrap Flux for a repository owned by a personal account, you can generate a [GitHub PAT](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) that can create repositories by checking all permissions under `repo`.

If you want to use an existing repository, the PAT’s user must have `admin` [permissions](https://docs.github.com/en/organizations/managing-access-to-your-organizations-repositories/repository-roles-for-an-organization#permissions-for-each-role).

Run the bootstrap for a repository on your personal GitHub account:

```sh
flux bootstrap github \
  --token-auth \
  --owner=my-github-username \
  --repository=my-repository-name \
  --branch=main \
  --path=clusters/my-cluster \
  --personal
```

If the specified repository does not exist, Flux will create it for you as private. If you wish to create a public repository, set `--private=false`.

When using `--token-auth`, the CLI and the Flux controllers running on the cluster will use the GitHub PAT to access the Git repository over HTTPS.

#### PAT secret

Note that the GitHub PAT is stored in the cluster as a **Kubernetes Secret** named `flux-system` inside the `flux-system` namespace. If you want to avoid storing your PAT in the cluster, please see how to configure [GitHub Deploy Keys](https://fluxcd.io/flux/installation/bootstrap/github/#github-deploy-keys).

## [GitHub Organization](https://fluxcd.io/flux/installation/bootstrap/github/#github-organization)

If you want to bootstrap Flux for a repository owned by an GitHub organization, it is recommended to create a dedicated user for Flux under your organization.

Generate a GitHub PAT for the Flux user that can create repositories by checking all permissions under `repo`.

If you want to use an existing repository, the Flux user must have `admin` permissions for that repository.

#### GitHub fine-grained PAT

Bootstrap can be run with a GitHub [fine-grained personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#fine-grained-personal-access-tokens), for repositories that are created ahead of time by an organization admin.

The fine-grained PAT must be generated with the following permissions:

- `Administration` -> `Access: Read-only`
- `Contents` -> `Access: Read and write`
- `Metadata` -> `Access: Read-only`

Note that `Administration` should be set to `Access: Read and write` when using `bootstrap github --token-auth=false`.

Run the bootstrap for a repository owned by a GitHub organization:

```sh
flux bootstrap github \
  --token-auth \
  --owner=my-github-organization \
  --repository=my-repository \
  --branch=main \
  --path=clusters/my-cluster
```

When creating a new repository, you can specify a list of GitHub teams with `--team=team1-slug,team2-slug`, those teams will be granted maintainer access to the repository.

## [GitHub Enterprise](https://fluxcd.io/flux/installation/bootstrap/github/#github-enterprise)

To run the bootstrap for a repository hosted on GitHub Enterprise, you have to specify your GitHub hostname:

```sh
flux bootstrap github \
  --token-auth \
  --hostname=my-github-enterprise.com \
  --owner=my-github-organization \
  --repository=my-repository \
  --branch=main \
  --path=clusters/my-cluster
```

If you want use SSH and [GitHub deploy keys](https://fluxcd.io/flux/installation/bootstrap/github/#github-deploy-keys), set `--token-auth=false` and provide the SSH hostname with `--ssh-hostname=my-github-enterprise.com`.

## [GitHub Deploy Keys](https://fluxcd.io/flux/installation/bootstrap/github/#github-deploy-keys)

If you want to bootstrap Flux using SSH instead of HTTP/S, you can set `--token-auth=false` and the Flux CLI will use the GitHub PAT to set a deploy key for your repository.

When using SSH, the bootstrap command will generate a SSH private key. The private key is stored in the cluster as a Kubernetes secret named `flux-system` inside the `flux-system` namespace.

The generated SSH key defaults to `ECDSA P-384`, to change the format use `--ssh-key-algorithm` and `--ssh-ecdsa-curve`.

The SSH public key, is used to create a GitHub deploy key. The deploy key is linked to the personal access token used to authenticate.

By default, the GitHub deploy key is set to read-only access. If you’re using Flux image automation, you must give it write access with `--read-write-key=true`.

#### Deploy Key rotation

Note that when the PAT is removed or when it expires, the GitHub deploy key will stop working. To regenerate the deploy key, delete the `flux-system` secret from the cluster and re-run the bootstrap command using a valid GitHub PAT.

## [Bootstrap without a GitHub PAT](https://fluxcd.io/flux/installation/bootstrap/github/#bootstrap-without-a-github-pat)

For existing GitHub repositories, you can bootstrap Flux over SSH without using a GitHub PAT.

To use a SSH key instead of a GitHub PAT, the command changes to `flux bootstrap git`:

```shell
flux bootstrap git \
  --url=ssh://git@github.com/<org>/<repository> \
  --branch=<my-branch> \
  --private-key-file=<path/to/ssh/private.key> \
  --password=<key-passphrase> \
  --path=clusters/my-cluster
```

# flux quellen hinzufügen

- `flux create source helm cert-manager --url https://charts.jetstack.io`
- `flux create source helm  traefik --url https://traefik.github.io/charts`
- `flux create source helm smueller18 --url https://smueller18.gitlab.io/helm-charts`
- `flux create source helm jenkins --url https://charts.jenkins.io`
- `flux create source helm nfs-subdir-external-provisioner --url https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/`
- `flux create source helm gitlab --url https://charts.gitlab.io`
- `flux create source helm minio-operator --url https://operator.min.io`
- `flux create source helm metallb --url https://metallb.github.io/metallb`
- `flux create source helm icinga --url https://icinga.github.io/helm-charts`
- `flux create source helm hashicorp --url https://helm.releases.hashicorp.com`
- `flux create source helm kubevpn --url https://raw.githubusercontent.com/kubenetworks/kubevpn/master/charts`
- `flux create source helm ingress-nginx --url https://kubernetes.github.io/ingress-nginx`
- `flux create source helm awx-operator --url https://ansible-community.github.io/awx-operator-helm/`
- `flux create source helm longhorn --url https://charts.longhorn.io`
- `flux create source helm fleet --url https://rancher.github.io/fleet-helm-charts/`
- `flux create source helm rancher-latest --url https://releases.rancher.com/server-charts/latest`
- `flux create source helm wg-easy --url https://raw.githubusercontent.com/hansehe/wg-easy-helm/master/helm/charts`
- `flux create source helm external-dns --url https://kubernetes-sigs.github.io/external-dns/`
- `flux create source helm bitnami --url https://charts.bitnami.com/bitnami`
- `flux create source helm jaegertracing --url https://jaegertracing.github.io/helm-charts`
- `flux create source helm grafana --url https://grafana.github.io/helm-charts`
- `flux create source helm grafana --urlhelm repo add keel https://keel-hq.github.io/keel/`
# flux Konfiguration

https://github.com/fluxcd/terraform-provider-flux/tree/main/examples/github-via-ssh

https://blog.andi95.de/2024/05/gitops-mit-fluxcd-fuer-meinen-heim-kubernetes-cluster/