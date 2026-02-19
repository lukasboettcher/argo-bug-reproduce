# create kind cluster
kind create cluster

# install argo-cd
kubectl create namespace argocd
kubectl apply -n argocd --server-side --force-conflicts -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# install kyverno and policy to convert vcluster secret into argo-cd cluster secret
helm --namespace kyverno upgrade --install kyverno kyverno --repo https://kyverno.github.io/kyverno/ --create-namespace --wait
kubectl apply -f kyverno-rbac.yaml
kubectl apply -f policy.yaml

# create namespace with limitrange for vcluster
kubectl apply -f namespace.yaml
helm --namespace test upgrade --install vcluster vcluster --repo https://charts.loft.sh --set exportKubeConfig.server="https://vcluster.test:443"

# deploy test-appl
kubectl apply -f app.yaml

# observe panic in argocd-application-controller

"""
panic: assignment to entry in nil map

goroutine 48 [running]:
k8s.io/kubectl/pkg/util/resource.maxResourceList(0x0, 0xc002831b78?)
	/go/pkg/mod/k8s.io/kubectl@v0.34.0/pkg/util/resource/resource.go:179 +0x27a
k8s.io/kubectl/pkg/util/resource.max(0x5b?, {0xc002831be8, 0x2, 0x0?})
	/go/pkg/mod/k8s.io/kubectl@v0.34.0/pkg/util/resource/resource.go:157 +0x5e
k8s.io/kubectl/pkg/util/resource.determineContainerReqs(0x48fb460?, 0xc0028327d0?, 0xc002689e0c?)
	/go/pkg/mod/k8s.io/kubectl@v0.34.0/pkg/util/resource/resource.go:149 +0x175
k8s.io/kubectl/pkg/util/resource.podRequests(0xc0026c2d88)
	/go/pkg/mod/k8s.io/kubectl@v0.34.0/pkg/util/resource/resource.go:57 +0x33a
k8s.io/kubectl/pkg/util/resource.PodRequestsAndLimits(0xc0026c2d88)
	/go/pkg/mod/k8s.io/kubectl@v0.34.0/pkg/util/resource/resource.go:36 +0x18
github.com/argoproj/argo-cd/v3/controller/cache.populatePodInfo(0xc002685300, 0xc0026900e0)
	/go/src/github.com/argoproj/argo-cd/controller/cache/info.go:462 +0xca8
github.com/argoproj/argo-cd/v3/controller/cache.populateNodeInfo(0xc002685300, 0xc0026900e0, {0x9b6e2c0, 0x0, 0x6768118?})
	/go/src/github.com/argoproj/argo-cd/controller/cache/info.go:54 +0x4c9
github.com/argoproj/argo-cd/v3/controller/cache.(*liveStateCache).getCluster.func1(0xc002685300, 0x0)
	/go/src/github.com/argoproj/argo-cd/controller/cache/cache.go:558 +0x85
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).newResource(0xc00171e000, 0xc002685300)
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:453 +0x82
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).sync.func1.1.1.1({0x6766ef0?, 0xc002685300?})
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:1067 +0x65
k8s.io/apimachinery/pkg/apis/meta/v1/unstructured.(*UnstructuredList).EachListItem(0xc000d13700, 0xc001f94288)
	/go/pkg/mod/k8s.io/apimachinery@v0.34.0/pkg/apis/meta/v1/unstructured/unstructured_list.go:48 +0x6d
k8s.io/apimachinery/pkg/api/meta.eachListItem({0x6768118, 0xc000d13700}, 0xc001f94288, 0x0)
	/go/pkg/mod/k8s.io/apimachinery@v0.34.0/pkg/api/meta/help.go:136 +0x8b
k8s.io/apimachinery/pkg/api/meta.EachListItem({0x6768118?, 0xc000d13700?}, 0x0?)
	/go/pkg/mod/k8s.io/apimachinery@v0.34.0/pkg/api/meta/help.go:119 +0x1f
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).sync.func1.1.1.(*ListPager).EachListItem.2({0x6768118?, 0xc000d13700?})
	/go/pkg/mod/k8s.io/client-go@v0.34.0/tools/pager/pager.go:185 +0x25
k8s.io/client-go/tools/pager.(*ListPager).eachListChunkBuffered(0xc001f94270, {0x679cde0, 0x9b6e2c0}, {{{0x0, 0x0}, {0x0, 0x0}}, {0x0, 0x0}, {0x0, ...}, ...}, ...)
	/go/pkg/mod/k8s.io/client-go@v0.34.0/tools/pager/pager.go:245 +0x2a2
k8s.io/client-go/tools/pager.(*ListPager).EachListItem(...)
	/go/pkg/mod/k8s.io/client-go@v0.34.0/tools/pager/pager.go:184
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).sync.func1.1.1(0xc001f94270)
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:1063 +0x116
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).listResources(0xc00171e000, {0x679ce90?, 0xc001f8af00?}, {0x67c3b30, 0xc001f8af50}, 0xc000ddbbf0)
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:747 +0x24d
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).sync.func1.1({0x67c3b30, 0xc001f8af50}, {0x0, 0x0})
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:1062 +0xbd
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).processApi(_, {_, _}, {{{0x0, 0x0}, {0xc001e52954, 0x3}}, {{0xc001e52950, 0x4}, {0xc001e52957, ...}, ...}, ...}, ...)
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:918 +0x15c
github.com/argoproj/gitops-engine/pkg/cache.(*clusterCache).sync.func1(0x67fa3d0?)
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/cache/cluster.go:1061 +0x273
github.com/argoproj/gitops-engine/pkg/utils/kube.RunAllAsync.func1()
	/go/src/github.com/argoproj/argo-cd/gitops-engine/pkg/utils/kube/ctl.go:370 +0x1b
golang.org/x/sync/errgroup.(*Group).Go.func1()
	/go/pkg/mod/golang.org/x/sync@v0.19.0/errgroup/errgroup.go:93 +0x50
created by golang.org/x/sync/errgroup.(*Group).Go in goroutine 263
	/go/pkg/mod/golang.org/x/sync@v0.19.0/errgroup/errgroup.go:78 +0x95
"""