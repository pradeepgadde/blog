---
layout: single
title:  "Installing Kubernetes Operator SDK"
date:   2022-07-07 02:55:04 +0530
categories: Kubernetes
tags: minikube 
classes: wide
show_date: true
header:
  overlay_image: /assets/images/kubernetes.png
  og_image: /assets/images/kubernetes.png
  teaser: /assets/images/kubernetes.png
  caption: "Photo credit: [**Kubernetes**](https://kubernetes.io)"
  actions:
    - label: "Learn more"
      url: "https://kubernetes.io"

author:
  name     : "Kubernetes"
  avatar   : "/assets/images/kubernetes.png"
 
sidebar:
  - title: "Blog"
 
    text: "Checkout other topics"
    nav: my-sidebar
---

In this post, let us install Kubernetes Operator SDK CLI tool using the `brew install operator-sdk` command.

```sh
(base) pradeep:~$brew install operator-sdk
Running `brew update --preinstall`...
==> Auto-updated Homebrew!
Updated 4 taps (hashicorp/tap, azure/functions, homebrew/core and homebrew/cask).
==> New Formulae
adamstark-audiofile      git-workspace            libapplewm               nb                       stanc3
astro                    glibc@2.13               libnetfilter_conntrack   neovide                  stencil
aws-nuke                 glider                   libnftnl                 nftables                 swtpm
aztfy                    gnustep-base             libnl                    oak                      synergy-core
bore-cli                 goctl                    libobjc2                 octosql                  teller
cargo-bundle             gokart                   libpython-tabulate       ohdear-cli               terramate
cfonts                   gold                     libxcvt                  opencl-headers           tinysearch
cxgo                     groestlcoin              livekit                  opentelemetry-cpp        toxcore
czg                      hashicorp/tap/hcdiag     livekit-cli              pacmc                    tradcpp
dart-sdk                 hatch                    llvm@13                  pax                      tremor-runtime
dbml-cli                 helmify                  lndir                    pg_cron                  trzsz-go
doggo                    hwatch                   lunar-date               pg_partman               ttmath
dotdrop                  ijq                      mabel                    pget                     tuc
dtrx                     install-peerdeps         maclaunch                phrase-cli               unisonlang
dump1090-mutability      iptables                 mariadb@10.7             pixie                    uthash
dumpling                 jackett                  markdown-toc             poac                     vectorscan
editorconfig-checker     jaq                      mbt                      podman-compose           verapdf
eget                     kics                     mbw                      primecount               webkitgtk
erlang@24                kt-connect               mcap                     qbe                      xcode-kotlin
evernote-backup          leapp-cli                mle                      redis@6.2                xkbcomp
fastnetmon               levant                   mprocs                   release-it               xpipe
flix                     lexicon                  mypaint-brushes          req                      yorkie
flock                    lgeneral                 nali                     sdl2_sound               zx
gcc@12                   libabw                   naml                     sftpgo
==> New Casks
aethersx2                                  gama-jdk                                   podman-desktop
amazon-luna                                gamma-control                              protokol
app-fair                                   groestlcoin-core                           psst
archy                                      gyroflow                                   roam-research
audiostellar                               hdfview                                    rockboxutility
avifquicklook                              input-source-pro                           rustdesk
betterandbetter                            jpc-qlcolorcode                            sol
betterdisplay                              jquake                                     sonixd
bike                                       ktalk                                      squash
bili-downloader                            localxpose                                 swiftcord
bilibili-official                          manila                                     tailscale
bing-wallpaper                             mbcord                                     tdr-kotelnikov
cardpresso                                 medis                                      tdr-nova
cleaneronepro                              mega                                       tdr-vos-slickeq
cloud189                                   metadatics                                 ti-smartview-ce-for-the-ti-84-plus-family
contour                                    miaoyan                                    tmpdisk
dcp-o-matic-combiner                       miln-movie-splitter                        tomatobar
dcp-o-matic-disk-writer                    miniwol                                    tqsl
dcp-o-matic-playlist-editor                mp3tag                                     trivial
detail                                     mx-power-gadget                            twitch-studio
dixa                                       nitro-pdf-pro                              ui
dmg-canvas                                 orangedrangon-android-messages             universal-android-debloater
ecamm-live                                 orion                                      vieb
electrum-grs                               oso-cloud                                  weektodo
elephicon                                  oxwu                                       wirecast
ferdium                                    phpwebstudy                                wow
fertigt-slate                              plex-htpc                                  yandex-music-unofficial
fly-key                                    plus42-binary                              yattee
gama                                       plus42-decimal                             yousician

You have 16 outdated formulae and 1 outdated cask installed.
You can upgrade them with brew upgrade
or list them with brew outdated.

==> Downloading https://ghcr.io/v2/homebrew/core/go/manifests/1.18.3
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/go/blobs/sha256:d99358b9ca6dadc9298ef1164d69fadd31ccbadd6fa21f2ddd710181648133
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:d99358b9ca6dadc9298ef1164d69fadd31ccbadd6f
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/operator-sdk/manifests/1.22.0
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/operator-sdk/blobs/sha256:713c7d54ab95a8f4ef7184f0904556592a22f833c69aea62a4cb
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:713c7d54ab95a8f4ef7184f0904556592a22f833c6
######################################################################## 100.0%
==> Installing dependencies for operator-sdk: go
==> Installing operator-sdk dependency: go
==> Pouring go--1.18.3.monterey.bottle.tar.gz
🍺  /usr/local/Cellar/go/1.18.3: 11,981 files, 592.9MB
==> Installing operator-sdk
==> Pouring operator-sdk--1.22.0.monterey.bottle.tar.gz
==> Caveats
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/operator-sdk/1.22.0: 11 files, 198.7MB
==> `brew cleanup` has not been run in the last 30 days, running now...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
Removing: /Users/pradeep/Library/Caches/Homebrew/asciinema--2.1.0... (2.3MB)
Removing: /Users/pradeep/Library/Caches/Homebrew/gdbm--1.23... (270.6KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/gettext--0.21... (8.5MB)
Removing: /Users/pradeep/Library/Caches/Homebrew/readline--8.1.2... (534.7KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/tcl-tk--8.6.12_1... (8.3MB)
Removing: /Users/pradeep/Library/Caches/Homebrew/terraform--1.1.9.zip... (19.7MB)
Removing: /Users/pradeep/Library/Caches/Homebrew/python@3.10_bottle_manifest--3.10.2... (18.7KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/gdbm_bottle_manifest--1.23... (6.1KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/hugo_bottle_manifest--0.92.0... (6.4KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/ca-certificates_bottle_manifest--2022-02-01... (1.8KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/openssl@1.1_bottle_manifest--1.1.1m... (7.6KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/sqlite_bottle_manifest--3.38.0... (6.9KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/asciinema_bottle_manifest--2.1.0... (15.5KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/git_bottle_manifest--2.33.1... (13.7KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/python@3.9_bottle_manifest--3.9.10... (18.1KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/gettext_bottle_manifest--0.21... (10.5KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/readline_bottle_manifest--8.1.2... (6.6KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/tcl-tk_bottle_manifest--8.6.12_1... (9.6KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/pcre2_bottle_manifest--10.38_1... (8.5KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/azure-cli_bottle_manifest--2.34.1... (18.3KB)
Removing: /Users/pradeep/Library/Caches/Homebrew/hugo_bottle_manifest--0.93.1... (6.4KB)
Removing: /Users/pradeep/Library/Logs/Homebrew/terraform... (119B)
==> Caveats
==> operator-sdk
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
(base) pradeep:~$
```

Verify the installation
```sh

(base) pradeep:~$operator-sdk version
operator-sdk version: "v1.22.0", commit: "9e95050a94577d1f4ecbaeb6c2755a9d2c231289", kubernetes version: "v1.24.1", go version: "go1.18.3", GOOS: "darwin", GOARCH: "amd64"
(base) pradeep:~$

```

Other option is to compile and install from the master

```sh
(base) pradeep:~$git clone https://github.com/operator-framework/operator-sdk
Cloning into 'operator-sdk'...
remote: Enumerating objects: 47412, done.
remote: Counting objects: 100% (288/288), done.
remote: Compressing objects: 100% (179/179), done.
remote: Total 47412 (delta 121), reused 218 (delta 86), pack-reused 47124
Receiving objects: 100% (47412/47412), 65.89 MiB | 9.99 MiB/s, done.
Resolving deltas: 100% (27740/27740), done.
```

```sh
(base) pradeep:~$pwd   
/Users/pradeep
(base) pradeep:~$cd operator-sdk 
(base) pradeep:~$git branch  
* master
```
```sh
(base) pradeep:~$git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/create-pull-request/patch-2123037
  remotes/origin/create-pull-request/patch-246b402
  remotes/origin/create-pull-request/patch-40ac14f
  remotes/origin/create-pull-request/patch-69ba8b0
  remotes/origin/create-pull-request/patch-6a2ca08
  remotes/origin/create-pull-request/patch-746ad39
  remotes/origin/create-pull-request/patch-7685cb1
  remotes/origin/create-pull-request/patch-81d0bfd
  remotes/origin/create-pull-request/patch-a1a1f9c
  remotes/origin/create-pull-request/patch-a9bb72f
  remotes/origin/create-pull-request/patch-ada7bf4
  remotes/origin/create-pull-request/patch-b26427b
  remotes/origin/create-pull-request/patch-b4cfdd2
  remotes/origin/create-pull-request/patch-bc8aedd
  remotes/origin/create-pull-request/patch-bf87d0a
  remotes/origin/create-pull-request/patch-d162b0b
  remotes/origin/create-pull-request/patch-d8c34a8
  remotes/origin/create-pull-request/patch-d942ee6
  remotes/origin/create-pull-request/patch-ddeecc9
  remotes/origin/create-pull-request/patch-e588158
  remotes/origin/create-pull-request/patch-f34c12d
  remotes/origin/create-pull-request/patch-fddeb05
  remotes/origin/dependabot/docker/images/ansible-operator/ubi8/ubi-8.6
  remotes/origin/dependabot/docker/images/scorecard-test-kuttl/kudobuilder/kuttl-v0.12.1
  remotes/origin/dependabot/github_actions/actions/checkout-3
  remotes/origin/dependabot/github_actions/actions/setup-go-3
  remotes/origin/dependabot/github_actions/docker/login-action-2
  remotes/origin/dependabot/github_actions/docker/setup-qemu-action-2
  remotes/origin/dependabot/github_actions/gaurav-nelson/github-action-markdown-link-check-1.0.14
  remotes/origin/dependabot/pip/images/ansible-operator/ansible-4.2.0
  remotes/origin/feature/external-validators
  remotes/origin/feature/run-bundle
  remotes/origin/feature/run-bundle(-upgrade)
  remotes/origin/fix_release
  remotes/origin/glossary
  remotes/origin/latest
  remotes/origin/master
  remotes/origin/release-4.2
  remotes/origin/release-4.3
  remotes/origin/release-4.4
  remotes/origin/release-v1.6.0
  remotes/origin/release-v1.6.1
  remotes/origin/release-v1.6.2
  remotes/origin/run-bundle-prototype
  remotes/origin/v0.0.7
  remotes/origin/v0.1.x
  remotes/origin/v0.10.x
  remotes/origin/v0.11.x
  remotes/origin/v0.12.x
  remotes/origin/v0.13.x
  remotes/origin/v0.14.x
  remotes/origin/v0.15.x
  remotes/origin/v0.16.x
  remotes/origin/v0.17.x
  remotes/origin/v0.18.x
  remotes/origin/v0.19.x
  remotes/origin/v0.2.x
  remotes/origin/v0.3.x
  remotes/origin/v0.4.x
  remotes/origin/v0.5.x
  remotes/origin/v0.6.x
  remotes/origin/v0.7.x
  remotes/origin/v0.8.x
  remotes/origin/v0.9.x
  remotes/origin/v1.0.x
  remotes/origin/v1.1.x
  remotes/origin/v1.10.x
  remotes/origin/v1.11.x
  remotes/origin/v1.12.x
  remotes/origin/v1.13.x
  remotes/origin/v1.14.x
  remotes/origin/v1.15.x
  remotes/origin/v1.16.x
  remotes/origin/v1.17.x
  remotes/origin/v1.18.x
  remotes/origin/v1.19.x
  remotes/origin/v1.2.x
  remotes/origin/v1.20.x
  remotes/origin/v1.21.x
  remotes/origin/v1.22.x
  remotes/origin/v1.3.x
  remotes/origin/v1.4.x
  remotes/origin/v1.5.x
  remotes/origin/v1.6.x
  remotes/origin/v1.7.x
  remotes/origin/v1.8.x
  remotes/origin/v1.9.x
(base) pradeep:~$git checkout master
Already on 'master'
Your branch is up to date with 'origin/master'.
```
```sh
(base) pradeep:~$make install
go install -gcflags "all=-trimpath=/Users/pradeep" -asmflags "all=-trimpath=/Users/pradeep" -ldflags " -X 'github.com/operator-framework/operator-sdk/internal/version.Version=v1.22.0+git' -X 'github.com/operator-framework/operator-sdk/internal/version.GitVersion=v1.22.0-20-g4070e40e' -X 'github.com/operator-framework/operator-sdk/internal/version.GitCommit=4070e40ec79cc78d538d20a139bad5e909ce695e' -X 'github.com/operator-framework/operator-sdk/internal/version.KubernetesVersion=v1.24.1' -X 'github.com/operator-framework/operator-sdk/internal/version.ImageVersion=v1.22.0' "  ./cmd/{operator-sdk,ansible-operator,helm-operator}
go: downloading k8s.io/client-go v0.24.1
go: downloading k8s.io/apimachinery v0.24.1
go: downloading github.com/sirupsen/logrus v1.8.1
go: downloading sigs.k8s.io/controller-runtime v0.12.1
go: downloading github.com/operator-framework/helm-operator-plugins v0.0.12-0.20220616200420-1a695cb9f6a1
go: downloading github.com/operator-framework/java-operator-plugins v0.5.1
go: downloading sigs.k8s.io/kubebuilder/v3 v3.5.0
go: downloading github.com/spf13/viper v1.10.0
go: downloading github.com/prometheus/client_golang v1.12.1
go: downloading github.com/operator-framework/operator-lib v0.11.0
go: downloading helm.sh/helm/v3 v3.9.0
go: downloading github.com/go-task/slim-sprig v0.0.0-20210107165309-348f09dbbbc0
go: downloading gomodules.xyz/jsonpatch/v3 v3.0.1
go: downloading k8s.io/apiextensions-apiserver v0.24.1
go: downloading k8s.io/cli-runtime v0.24.1
go: downloading github.com/operator-framework/operator-registry v1.23.0
go: downloading golang.org/x/text v0.3.7
go: downloading golang.org/x/sys v0.0.0-20220614162138-6c1b26c55098
go: downloading k8s.io/api v0.24.1
go: downloading github.com/go-logr/logr v1.2.2
go: downloading github.com/go-logr/zapr v1.2.0
go: downloading go.uber.org/zap v1.19.1
go: downloading k8s.io/utils v0.0.0-20220210201930-3a6ce19ff2f9
go: downloading github.com/evanphx/json-patch v4.12.0+incompatible
go: downloading k8s.io/kubectl v0.24.1
go: downloading golang.org/x/mod v0.6.0-dev.0.20220419223038-86c51ed26bb4
go: downloading github.com/operator-framework/api v0.15.1-0.20220624132056-decf74800a17
go: downloading github.com/spf13/afero v1.6.0
go: downloading github.com/iancoleman/strcase v0.2.0
go: downloading github.com/gogo/protobuf v1.3.2
go: downloading github.com/google/gofuzz v1.2.0
go: downloading golang.org/x/net v0.0.0-20220407224826-aac1ed45d8e3
go: downloading k8s.io/klog/v2 v2.60.1
go: downloading github.com/fsnotify/fsnotify v1.5.1
go: downloading github.com/magiconair/properties v1.8.5
go: downloading github.com/mitchellh/mapstructure v1.4.3
go: downloading github.com/spf13/cast v1.4.1
go: downloading github.com/spf13/jwalterweatherman v1.1.0
go: downloading github.com/subosito/gotenv v1.2.0
go: downloading gopkg.in/ini.v1 v1.66.2
go: downloading golang.org/x/tools v0.1.11
go: downloading github.com/sergi/go-diff v1.2.0
go: downloading sigs.k8s.io/structured-merge-diff/v4 v4.2.1
go: downloading github.com/golang/groupcache v0.0.0-20210331224755-41bb18bfe9da
go: downloading github.com/beorn7/perks v1.0.1
go: downloading github.com/cespare/xxhash/v2 v2.1.2
go: downloading github.com/golang/protobuf v1.5.2
go: downloading github.com/prometheus/client_model v0.2.0
go: downloading github.com/prometheus/common v0.32.1
go: downloading github.com/prometheus/procfs v0.7.3
go: downloading google.golang.org/protobuf v1.28.0
go: downloading gomodules.xyz/orderedmap v0.1.0
go: downloading github.com/jmoiron/sqlx v1.3.4
go: downloading github.com/Masterminds/squirrel v1.5.2
go: downloading github.com/lib/pq v1.10.4
go: downloading github.com/rubenv/sql-migrate v1.1.1
go: downloading github.com/Masterminds/semver/v3 v3.1.1
go: downloading k8s.io/kube-openapi v0.0.0-20220328201542-3ee0da9b0b42
go: downloading github.com/cyphar/filepath-securejoin v0.2.3
go: downloading github.com/mitchellh/copystructure v1.2.0
go: downloading github.com/xeipuuv/gojsonschema v1.2.0
go: downloading github.com/google/gnostic v0.5.7-v3refs
go: downloading github.com/Masterminds/sprig/v3 v3.2.2
go: downloading github.com/gosuri/uitable v0.0.4
go: downloading golang.org/x/term v0.0.0-20210927222741-03fcf44c2211
go: downloading github.com/imdario/mergo v0.3.12
go: downloading golang.org/x/time v0.0.0-20220210224613-90d013bbcef8
go: downloading sigs.k8s.io/kustomize/api v0.11.4
go: downloading sigs.k8s.io/kustomize/kyaml v0.13.6
go: downloading go.uber.org/atomic v1.7.0
go: downloading go.uber.org/multierr v1.6.0
go: downloading k8s.io/component-base v0.24.1
go: downloading gomodules.xyz/jsonpatch/v2 v2.2.0
go: downloading sigs.k8s.io/json v0.0.0-20211208200746-9f7c6b3444d2
go: downloading golang.org/x/oauth2 v0.0.0-20211104180415-d3ed0bb246c8
go: downloading github.com/mxk/go-flowrate v0.0.0-20140419014527-cca7078d478f
go: downloading github.com/gobuffalo/flect v0.2.5
go: downloading github.com/operator-framework/operator-manifest-tools v0.2.1
go: downloading github.com/blang/semver/v4 v4.0.0
go: downloading github.com/thoas/go-funk v0.8.0
go: downloading github.com/blang/semver v3.5.1+incompatible
go: downloading gopkg.in/inf.v0 v0.9.1
go: downloading github.com/Azure/go-autorest/autorest v0.11.20
go: downloading github.com/Azure/go-autorest/autorest/adal v0.9.15
go: downloading github.com/davecgh/go-spew v1.1.1
go: downloading github.com/hashicorp/hcl v1.0.0
go: downloading github.com/cloudflare/cfssl v1.5.0
go: downloading github.com/Azure/go-autorest v14.2.0+incompatible
go: downloading sigs.k8s.io/controller-tools v0.9.0
go: downloading github.com/onsi/gomega v1.18.1
go: downloading github.com/google/uuid v1.2.0
go: downloading github.com/json-iterator/go v1.1.12
go: downloading github.com/matttproud/golang_protobuf_extensions v1.0.2-0.20181231171920-c182affec369
go: downloading github.com/exponent-io/jsonpath v0.0.0-20151013193312-d6023ce2651d
go: downloading github.com/lann/builder v0.0.0-20180802200727-47ae307949d0
go: downloading github.com/mitchellh/reflectwalk v1.0.2
go: downloading github.com/go-gorp/gorp/v3 v3.0.2
go: downloading github.com/gobwas/glob v0.2.3
go: downloading golang.org/x/crypto v0.0.0-20220408190544-5352b0902921
go: downloading github.com/containerd/containerd v1.4.11
go: downloading github.com/opencontainers/image-spec v1.0.3-0.20211202183452-c5a74bcca799
go: downloading oras.land/oras-go v1.1.0
go: downloading github.com/liggitt/tabwriter v0.0.0-20181228230101-89fcab3d43de
go: downloading github.com/xeipuuv/gojsonreference v0.0.0-20180127040603-bd5ef7bd5415
go: downloading github.com/fatih/color v1.13.0
go: downloading github.com/Masterminds/goutils v1.1.1
go: downloading github.com/huandu/xstrings v1.3.2
go: downloading github.com/shopspring/decimal v1.2.0
go: downloading github.com/google/go-cmp v0.5.6
go: downloading github.com/adrg/xdg v0.4.0
go: downloading github.com/docker/cli v20.10.12+incompatible
go: downloading github.com/docker/docker v20.10.14+incompatible
go: downloading go.etcd.io/bbolt v1.3.6
go: downloading github.com/ghodss/yaml v1.0.0
go: downloading github.com/otiai10/copy v1.2.0
go: downloading github.com/kr/text v0.2.0
go: downloading cloud.google.com/go v0.99.0
go: downloading github.com/google/go-containerregistry v0.8.0
go: downloading github.com/Azure/go-autorest/logger v0.2.1
go: downloading github.com/Azure/go-autorest/tracing v0.6.0
go: downloading github.com/Azure/go-autorest/autorest/date v0.3.0
go: downloading github.com/golang-jwt/jwt/v4 v4.0.0
go: downloading github.com/go-errors/errors v1.0.1
go: downloading github.com/monochromegane/go-gitignore v0.0.0-20200626010858-205db1a8cc00
go: downloading github.com/stretchr/testify v1.7.1
go: downloading github.com/xlab/treeprint v0.0.0-20181112141820-a009c3971eca
go: downloading github.com/google/cel-go v0.10.1
go: downloading google.golang.org/genproto v0.0.0-20220407144326-9054f6ed7bac
go: downloading github.com/h2non/filetype v1.1.1
go: downloading github.com/h2non/go-is-svg v0.0.0-20160927212452-35e8c4b0612c
go: downloading google.golang.org/grpc v1.45.0
go: downloading github.com/gregjones/httpcache v0.0.0-20180305231024-9cad4c3443a7
go: downloading github.com/peterbourgon/diskv v2.0.1+incompatible
go: downloading github.com/modern-go/concurrent v0.0.0-20180306012644-bacd9c7ef1dd
go: downloading github.com/modern-go/reflect2 v1.0.2
go: downloading github.com/chai2010/gettext-go v0.0.0-20160711120539-c6fed771bfd5
go: downloading github.com/MakeNowJust/heredoc v0.0.0-20170808103936-bb23615498cd
go: downloading github.com/mitchellh/go-wordwrap v1.0.0
go: downloading github.com/russross/blackfriday v1.5.2
go: downloading github.com/munnerz/goautoneg v0.0.0-20191010083416-a7dc8b61c822
go: downloading github.com/zmap/zlint/v2 v2.2.1
go: downloading github.com/google/certificate-transparency-go v1.0.21
go: downloading github.com/lann/ps v0.0.0-20150810152359-62de8c46ede0
go: downloading github.com/asaskevich/govalidator v0.0.0-20200428143746-21a406dcc535
go: downloading k8s.io/apiserver v0.24.1
go: downloading github.com/opencontainers/go-digest v1.0.0
go: downloading golang.org/x/sync v0.0.0-20210220032951-036812b2e83c
go: downloading github.com/mattn/go-runewidth v0.0.9
go: downloading github.com/xeipuuv/gojsonpointer v0.0.0-20180127040702-4e3ac2762d5f
go: downloading github.com/mattn/go-colorable v0.1.12
go: downloading github.com/containerd/ttrpc v1.1.0
go: downloading github.com/containerd/continuity v0.0.0-20201208142359-180525291bb7
go: downloading github.com/fatih/structtag v1.1.0
go: downloading github.com/markbates/inflect v1.0.4
go: downloading github.com/joelanford/ignore v0.0.0-20210607151042-0d25dc18b62d
go: downloading github.com/mitchellh/hashstructure/v2 v2.0.2
go: downloading github.com/docker/docker-credential-helpers v0.6.4
go: downloading github.com/go-openapi/jsonreference v0.19.5
go: downloading github.com/go-openapi/swag v0.19.14
go: downloading github.com/mitchellh/go-homedir v1.1.0
go: downloading github.com/containerd/stargz-snapshotter/estargz v0.10.1
go: downloading github.com/stoewer/go-strcase v1.2.0
go: downloading github.com/docker/distribution v0.0.0-20191216044856-a8371794149d
go: downloading github.com/docker/go-connections v0.4.0
go: downloading github.com/google/btree v1.0.1
go: downloading github.com/moby/term v0.0.0-20210619224110-3f7ff695adc6
go: downloading github.com/emicklei/go-restful v2.9.5+incompatible
go: downloading github.com/docker/go-units v0.4.0
go: downloading github.com/google/shlex v0.0.0-20191202100458-e7afc7fbc510
go: downloading github.com/golang-migrate/migrate/v4 v4.6.2
go: downloading github.com/mattn/go-sqlite3 v1.10.0
go: downloading github.com/moby/spdystream v0.2.0
go: downloading github.com/gobuffalo/envy v1.6.5
go: downloading github.com/go-git/go-git/v5 v5.3.0
go: downloading github.com/pmezard/go-difflib v1.0.0
go: downloading github.com/antlr/antlr4/runtime/Go/antlr v0.0.0-20210826220005-b48c857c3a0e
go: downloading github.com/PuerkitoBio/purell v1.1.1
go: downloading github.com/go-openapi/jsonpointer v0.19.5
go: downloading github.com/zmap/zcrypto v0.0.0-20200911161511-43ff0ea04f21
go: downloading github.com/mailru/easyjson v0.7.6
go: downloading github.com/morikuni/aec v1.0.0
go: downloading github.com/klauspost/compress v1.14.1
go: downloading github.com/vbatts/tar-split v0.11.2
go: downloading go.opentelemetry.io/otel/trace v0.20.0
go: downloading github.com/joho/godotenv v1.3.0
go: downloading github.com/go-git/go-billy/v5 v5.1.0
go: downloading go.opentelemetry.io/otel v0.20.0
go: downloading github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578
go: downloading github.com/weppos/publicsuffix-go v0.13.0
go: downloading github.com/josharian/intern v1.0.0
go: downloading sigs.k8s.io/apiserver-network-proxy/konnectivity-client v0.0.30
go: downloading go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp v0.20.0
go: downloading go.opentelemetry.io/otel/exporters/otlp v0.20.0
go: downloading go.opentelemetry.io/otel/sdk v0.20.0
go: downloading go.opentelemetry.io/contrib v0.20.0
go: downloading go.starlark.net v0.0.0-20200306205701-8dd3e2ee1dd5
go: downloading github.com/go-git/gcfg v1.5.0
go: downloading github.com/jbenet/go-context v0.0.0-20150711004518-d14ea06fba99
go: downloading github.com/gorilla/mux v1.8.0
go: downloading go.opentelemetry.io/otel/sdk/export/metric v0.20.0
go: downloading go.opentelemetry.io/proto/otlp v0.7.0
go: downloading go.opentelemetry.io/otel/metric v0.20.0
go: downloading go.opentelemetry.io/otel/sdk/metric v0.20.0
go: downloading gopkg.in/warnings.v0 v0.1.2
go: downloading github.com/docker/go-metrics v0.0.1
go: downloading github.com/felixge/httpsnoop v1.0.1
go: downloading github.com/grpc-ecosystem/grpc-gateway v1.16.0

(base) pradeep:~$
```

```sh
(base) pradeep:~$which operator-sdk
/usr/local/bin/operator-sdk
(base) pradeep:~$
```