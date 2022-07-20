---
layout: single
title:  "Building a Kubernetes Operator"
date:   2022-07-07 03:55:04 +0530
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

In this post, let us create a simple Hello world Kubernetes Operator using Golang, as discussed here https://developers.redhat.com/articles/2021/09/07/build-kubernetes-operator-six-steps#.

## Verify Golang
```go
(base) pradeep:~$go version
go version go1.18.1 darwin/amd64
(base) pradeep:~$
```

## Verify Minikube

```sh
(base) pradeep:~$minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured

(base) pradeep:~$
```

## Verify Operator SDK

```go
(base) pradeep:~$operator-sdk version
operator-sdk version: "v1.22.0", commit: "9e95050a94577d1f4ecbaeb6c2755a9d2c231289", kubernetes version: "v1.24.1", go version: "go1.18.3", GOOS: "darwin", GOARCH: "amd64"
(base) pradeep:~$

```



## Step 1: Generate boilerplate code

```sh
(base) pradeep:~$mkdir -p $GOPATH/src/operators && cd $GOPATH/src/operators
```

Start Minikube

```sh

(base) pradeep:~$minikube start init
😄  minikube v1.25.2 on Darwin 12.4
✨  Using the hyperkit driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🏃  Updating the running hyperkit "minikube" VM ...
🎉  minikube 1.26.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.26.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🐳  Preparing Kubernetes v1.23.3 on Docker 20.10.12 ...
    ▪ kubelet.housekeeping-interval=5m
🔎  Verifying Kubernetes components...
    ▪ Using image gcr.io/k8s-minikube/storage-provisioner:v5
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  Enabled addons: storage-provisioner, default-storageclass, ingress
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
(base) pradeep:~$

```
Then run `operator-sdk init` to generate the boilerplate code for our example application:

```go
(base) pradeep:~$operator-sdk init
Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.12.1
go: downloading k8s.io/apimachinery v0.24.0
go: downloading k8s.io/client-go v0.24.0
go: downloading github.com/go-logr/logr v1.2.0
go: downloading k8s.io/component-base v0.24.0
go: downloading github.com/google/gofuzz v1.1.0
go: downloading k8s.io/api v0.24.0
go: downloading k8s.io/apiextensions-apiserver v0.24.0
go: downloading golang.org/x/net v0.0.0-20220127200216-cd36cc0744dd
go: downloading github.com/google/uuid v1.1.2
go: downloading golang.org/x/sys v0.0.0-20220209214540-3681064d5158
go: downloading google.golang.org/protobuf v1.27.1
go: downloading github.com/google/go-cmp v0.5.5
go: downloading google.golang.org/appengine v1.6.7
Update dependencies:
$ go mod tidy
go: downloading github.com/onsi/ginkgo v1.16.5
go: downloading github.com/stretchr/testify v1.7.0
go: downloading github.com/Azure/go-autorest/autorest v0.11.18
go: downloading github.com/Azure/go-autorest/autorest/adal v0.9.13
go: downloading go.uber.org/goleak v1.1.12
go: downloading github.com/benbjohnson/clock v1.1.0
go: downloading cloud.google.com/go v0.81.0
go: downloading gopkg.in/check.v1 v1.0.0-20200227125254-8fa46927fb4f
go: downloading golang.org/x/xerrors v0.0.0-20200804184101-5ec99f83aff1
go: downloading github.com/nxadm/tail v1.4.8
go: downloading github.com/Azure/go-autorest/autorest/mocks v0.4.1
go: downloading github.com/form3tech-oss/jwt-go v3.2.3+incompatible
go: downloading golang.org/x/crypto v0.0.0-20220214200702-86341886e292
go: downloading gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7
go: downloading github.com/niemeyer/pretty v0.0.0-20200227124842-a10e7caefd8e
Next: define a resource with:
$ operator-sdk create api
(base) pradeep:~$
```

### Step 2: Create APIs and a custom resource

The following command creates an API and labels it `Traveller` through the `--kind` option. In the YAML configuration files created by the command, you can find a field labeled `kind` with the value `Traveller`. This field indicates that `Traveller` is used throughout the development process to refer to our APIs:

```go
(base) pradeep:~$operator-sdk create api --version=v1alpha1 --kind=Traveller
Create Resource [y/n]
y
Create Controller [y/n]
y
Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
api/v1alpha1/traveller_types.go
controllers/traveller_controller.go
Update dependencies:
$ go mod tidy
go: downloading github.com/onsi/ginkgo/v2 v2.0.0
Running make:
$ make generate
mkdir -p /Users/pradeep/go/src/operators/bin
GOBIN=/Users/pradeep/go/src/operators/bin go install sigs.k8s.io/controller-tools/cmd/controller-gen@v0.9.0
go: downloading github.com/fatih/color v1.12.0
go: downloading golang.org/x/tools v0.1.10-0.20220218145154-897bd77cd717
go: downloading github.com/mattn/go-isatty v0.0.12
go: downloading github.com/mattn/go-colorable v0.1.8
go: downloading golang.org/x/mod v0.6.0-dev.0.20220106191415-9b9b3d81d5e3
/Users/pradeep/go/src/operators/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
Next: implement your new API and generate the manifests (e.g. CRDs,CRs) with:
$ make manifests
(base) pradeep:~$

```



### Step 3: Download the dependencies

Our application uses the `tidy` module to remove dependencies we don't need, and the `vendor` module [to consolidate packages](https://golang.org/ref/mod#vendoring). Install these modules as follows:

```go
(base) pradeep:~$go mod tidy
(base) pradeep:~$go mod vendor
(base) pradeep:~$
```

### Step 4: Create a deployment

Now we will create, under our Kubernetes Operator umbrella, the standard resources that make up a containerized application. Because a Kubernetes Operator runs iteratively to reconcile the state of your application, it's very important to write the controller to be idempotent: In other words, the controller can run the code multiple times without creating multiple instances of a resource.

The following repo includes a controller for a deployment resource in the file `controllers/deployment.go`.

The [code for this step](https://github.com/deepak1725/hello-operator2/commit/c87544405b8359da2007317112bb64e434330f5f) is available in the Hello Operator GitHub repository.

```go
(base) pradeep:~$cat controllers/deployment.go 
package controllers

import (
	"context"

	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/types"
	"sigs.k8s.io/controller-runtime/pkg/controller/controllerutil"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"

	mydomainv1alpha1 "hello-operator2/api/v1alpha1"
)

func labels(v *mydomainv1alpha1.Traveller, tier string) map[string]string {
	// Fetches and sets labels

	return map[string]string{
		"app":             "visitors",
		"visitorssite_cr": v.Name,
		"tier":            tier,
	}
}

// ensureDeployment ensures Deployment resource presence in given namespace.
func (r *TravellerReconciler) ensureDeployment(request reconcile.Request,
	instance *mydomainv1alpha1.Traveller,
	dep *appsv1.Deployment,
) (*reconcile.Result, error) {

	// See if deployment already exists and create if it doesn't
	found := &appsv1.Deployment{}
	err := r.Get(context.TODO(), types.NamespacedName{
		Name:      dep.Name,
		Namespace: instance.Namespace,
	}, found)
	if err != nil && errors.IsNotFound(err) {

		// Create the deployment
		err = r.Create(context.TODO(), dep)

		if err != nil {
			// Deployment failed
			return &reconcile.Result{}, err
		} else {
			// Deployment was successful
			return nil, nil
		}
	} else if err != nil {
		// Error that isn't due to the deployment not existing
		return &reconcile.Result{}, err
	}

	return nil, nil
}

// backendDeployment is a code for Creating Deployment
func (r *TravellerReconciler) backendDeployment(v *mydomainv1alpha1.Traveller) *appsv1.Deployment {

	labels := labels(v, "backend")
	size := int32(1)
	dep := &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      "hello-pod",
			Namespace: v.Namespace,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &size,
			Selector: &metav1.LabelSelector{
				MatchLabels: labels,
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: labels,
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{{
						Image:           "paulbouwer/hello-kubernetes:1.10",
						ImagePullPolicy: corev1.PullAlways,
						Name:            "hello-pod",
						Ports: []corev1.ContainerPort{{
							ContainerPort: 8080,
							Name:          "hello",
						}},
					}},
				},
			},
		},
	}

	controllerutil.SetControllerReference(v, dep, r.Scheme)
	return dep
}
(base) pradeep:~$

```

### Step 5: Create a service

Because we want the pods created by our deployment to be accessible outside our system, we attach a service to the deployment we just created. The code is in the file `controllers/service.go`.

The [code for this step](https://github.com/deepak1725/hello-operator2/commit/b17c24f7abd75328aa1549b0afb6c6bcb1ca8f61) is available in the Hello Operator GitHub repository.

```go
(base) pradeep:~$cat controllers/service.go 
package controllers

import (
	"context"
	mydomainv1alpha1 "hello-operator2/api/v1alpha1"

	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/types"
	"k8s.io/apimachinery/pkg/util/intstr"
	"sigs.k8s.io/controller-runtime/pkg/controller/controllerutil"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"
)

// ensureService ensures Service is Running in a namespace.
func (r *TravellerReconciler) ensureService(request reconcile.Request,
	instance *mydomainv1alpha1.Traveller,
	service *corev1.Service,
) (*reconcile.Result, error) {

	// See if service already exists and create if it doesn't
	found := &appsv1.Deployment{}
	err := r.Get(context.TODO(), types.NamespacedName{
		Name:      service.Name,
		Namespace: instance.Namespace,
	}, found)
	if err != nil && errors.IsNotFound(err) {

		// Create the service
		err = r.Create(context.TODO(), service)

		if err != nil {
			// Service creation failed
			return &reconcile.Result{}, err
		} else {
			// Service creation was successful
			return nil, nil
		}
	} else if err != nil {
		// Error that isn't due to the service not existing
		return &reconcile.Result{}, err
	}

	return nil, nil
}

// backendService is a code for creating a Service
func (r *TravellerReconciler) backendService(v *mydomainv1alpha1.Traveller) *corev1.Service {
	labels := labels(v, "backend")

	service := &corev1.Service{
		ObjectMeta: metav1.ObjectMeta{
			Name:      "backend-service",
			Namespace: v.Namespace,
		},
		Spec: corev1.ServiceSpec{
			Selector: labels,
			Ports: []corev1.ServicePort{{
				Protocol:   corev1.ProtocolTCP,
				Port:       80,
				TargetPort: intstr.FromInt(8080),
				NodePort:   30685,
			}},
			Type: corev1.ServiceTypeNodePort,
		},
	}

	controllerutil.SetControllerReference(v, service, r.Scheme)
	return service
}
(base) pradeep:~$

```



### Step 6: Add a reference in the controller

This step lets our controller know the existence of the deployment and service. It does this through edits to the reconciliation loop function of the `traveller_controller.go` file.



Original template

```go
(base) pradeep:~$cat controllers/traveller_controller.go 
/*
Copyright 2022.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package controllers

import (
	"context"

	"k8s.io/apimachinery/pkg/runtime"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"

	mydomainv1alpha1 "operators/api/v1alpha1"
)

// TravellerReconciler reconciles a Traveller object
type TravellerReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=my.domain,resources=travellers,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=my.domain,resources=travellers/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=my.domain,resources=travellers/finalizers,verbs=update

// Reconcile is part of the main kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// TODO(user): Modify the Reconcile function to compare the state specified by
// the Traveller object against the actual cluster state, and then
// perform operations to make the cluster state reflect the state specified by
// the user.
//
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.12.1/pkg/reconcile
func (r *TravellerReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	_ = log.FromContext(ctx)

	// TODO(user): your logic here

	return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
func (r *TravellerReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&mydomainv1alpha1.Traveller{}).
		Complete(r)
}
(base) pradeep:~$

```

Sample 

```go
(base) pradeep:~$cat controllers/traveller_controller.go
/*
Copyright 2021.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/

package controllers

import (
	"context"

	appsv1 "k8s.io/api/apps/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/types"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"
	"sigs.k8s.io/controller-runtime/pkg/reconcile"

	mydomainv1alpha1 "hello-operator2/api/v1alpha1"
)

// TravellerReconciler reconciles a Traveller object
type TravellerReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=my.domain,resources=travellers,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=my.domain,resources=travellers/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=my.domain,resources=travellers/finalizers,verbs=update

// Reconcile is part of the main kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// TODO(user): Modify the Reconcile function to compare the state specified by
// the Traveller object against the actual cluster state, and then
// perform operations to make the cluster state reflect the state specified by
// the user.
//
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.9.2/pkg/reconcile
func (r *TravellerReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	log := log.FromContext(ctx).WithValues("Traveller", req.NamespacedName)

	// Fetch the Traveller instance
	instance := &mydomainv1alpha1.Traveller{}
	err := r.Get(context.TODO(), req.NamespacedName, instance)
	if err != nil {
		if errors.IsNotFound(err) {
			// Request object not found, could have been deleted after reconcile request.
			// Owned objects are automatically garbage collected. For additional cleanup logic use finalizers.
			// Return and don't requeue
			return reconcile.Result{}, nil
		}
		// Error reading the object - requeue the request.
		return reconcile.Result{}, err
	}

	// Check if this Deployment already exists
	found := &appsv1.Deployment{}
	err = r.Get(context.TODO(), types.NamespacedName{Name: instance.Name, Namespace: instance.Namespace}, found)
	var result *reconcile.Result
	result, err = r.ensureDeployment(req, instance, r.backendDeployment(instance))
	if result != nil {
		log.Error(err, "Deployment Not ready")
		return *result, err
	}

	// Check if this Service already exists
	result, err = r.ensureService(req, instance, r.backendService(instance))
	if result != nil {
		log.Error(err, "Service Not ready")
		return *result, err
	}

	// Deployment and Service already exists - don't requeue
	log.Info("Skip reconcile: Deployment and service already exists",
		"Deployment.Namespace", found.Namespace, "Deployment.Name", found.Name)

	return ctrl.Result{}, nil
}

// SetupWithManager sets up the controller with the Manager.
func (r *TravellerReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&mydomainv1alpha1.Traveller{}).
		Complete(r)
}
(base) pradeep:~$

```

### Installing the CRD

All we have to do to deploy our hard work locally is to run a build:

```sh
(base) pradeep:~$make install
/Users/pradeep/go/src/operators/bin/controller-gen rbac:roleName=manager-role crd webhook paths="./..." output:crd:artifacts:config=config/crd/bases
hello-operator2/api/v1alpha1:-: CRD for Traveller.my.domain has no storage version
Error: not all generators ran successfully
run `controller-gen rbac:roleName=manager-role crd webhook paths=./... output:crd:artifacts:config=config/crd/bases -w` to see all available markers, or `controller-gen rbac:roleName=manager-role crd webhook paths=./... output:crd:artifacts:config=config/crd/bases -h` for usage
make: *** [manifests] Error 1
(base) pradeep:~$kustomize build config/samples | kubectl apply -f -
zsh: command not found: kustomize
error: no objects passed to apply
(base) pradeep:~$pwd
/Users/pradeep/go/src/operators
(base) pradeep:~$ls
Dockerfile	PROJECT		api		config		go.mod		hack		vendor
Makefile	README.md	bin		controllers	go.sum		main.go
(base) pradeep:~$ls config 
crd		manager		prometheus	samples
default		manifests	rbac		scorecard
(base) pradeep:~$ls config/samples 
_v1alpha1_traveller.yaml	kustomization.yaml
(base) pradeep:~$
(base) pradeep:~$

```

```sh
(base) pradeep:~$brew install kustomize
==> Downloading https://ghcr.io/v2/homebrew/core/kustomize/manifests/4.5.5
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/kustomize/blobs/sha256:d8cba8c955b392279f9a95be294d8c005ccfb3dc98b
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sha256:d8cba8c955b392279f9a95be294d8c
######################################################################## 100.0%
==> Pouring kustomize--4.5.5.monterey.bottle.tar.gz
==> Caveats
zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/kustomize/4.5.5: 8 files, 17.6MB
==> Running `brew cleanup kustomize`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
(base) pradeep:~$

```

```sh
(base) pradeep:~$kustomize build config/samples | kubectl apply -f -
error: unable to recognize "STDIN": no matches for kind "Traveller" in version "my.domain/v1alpha1"
(base) pradeep:~$
```

