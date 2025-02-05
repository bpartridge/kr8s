# Alternatives

There are many [third-party client libraries for Kubernetes](https://kubernetes.io/docs/reference/using-api/client-libraries/#community-maintained-client-libraries) out there, that's the wonderful nature of open source software.

The core goals of `kr8s` are to be readable, beginner-friendly, batteries-included and maintainable above all else. However, other libraries in the ecosystem have different strengths and goals which may better suit the needs of your project.

This page is intended to highlight the differences between `kr8s` and other libraries to help you make good choices about which library is best for your project.

```{warning}
These comparions are subjective and certainly skewed in favour of `kr8s`. Take this information with a pinch of salt and be sure to do your own research.

If you spot any information on this page that you beleive to be incorrect or incomplete please don't hesitate to [open a Pull Request](https://github.com/kr8s-org/kr8s/edit/main/docs/alternatives.md).
```

## Comparison Table

| Name                  | Sync | Asyncio | Repo Stars  | PyPI Downloads |
| --------------------- | ---- | ------- | ----------- | -------------- |
| [`kr8s`](https://github.com/kr8s-org/kr8s) | ✅ | ✅ | ![GitHub Repo stars](https://img.shields.io/github/stars/kr8s-org/kr8s) | ![PyPI - Downloads](https://img.shields.io/pypi/dm/kr8s) |
| [`kubernetes`](https://github.com/kubernetes-client/python) | ✅ | ❌ | ![GitHub Repo stars](https://img.shields.io/github/stars/kubernetes-client/python) | ![PyPI - Downloads](https://img.shields.io/pypi/dm/kubernetes) |
| [`kubernetes-asyncio`](https://github.com/tomplus/kubernetes_asyncio) | ❌ | ✅ |  ![GitHub Repo stars](https://img.shields.io/github/stars/tomplus/kubernetes_asyncio) | ![PyPI - Downloads](https://img.shields.io/pypi/dm/kubernetes-asyncio) |
| [`pykube-ng`](https://pykube.readthedocs.io/en/latest/) | ✅ | ❌ | ![Gitea Repo Stars](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fcodeberg.org%2Fapi%2Fv1%2Frepos%2Fhjacobs%2Fpykube-ng%2Fstargazers&query=%24.length&label=stars) | ![PyPI - Downloads](https://img.shields.io/pypi/dm/pykube-ng) |
| [`lightkube`](https://lightkube.readthedocs.io/en/stable/) | ✅ | ✅ |  ![GitHub Repo stars](https://img.shields.io/github/stars/gtsystem/lightkube) | ![PyPI - Downloads](https://img.shields.io/pypi/dm/lightkube) |

## Direct Comparisons

### `kr8s` vs `kubernetes`

The official `kubernetes` library maps exactly onto the Kubernetes API due to auto generation. However, this generation results in very verbose code and hard to understand documentation.

In contrast `kr8s` may not have 100% API coverage with the Kubernetes API but the library is written to be very clear and readable which generally improves code quality. You can also use the [low-level API](client) to fill in any missing API gaps that you may need.

Here's an example comparing listing all Pods using a label selector:

`````{tab-set}

````{tab-item} kr8s
```python
import kr8s

selector = {'component': 'kube-scheduler'}

for pod in kr8s.get("pods", namespace=kr8s.ALL, label_selector=selector):
    print(pod.namespace, pod.name)
```
````

````{tab-item} kubernetes
```python
from kubernetes import client, config

selector = {'component': 'kube-scheduler'}
selector_str = ",".join([f"{key}={value}" for key, value in selector.items()])

config.load_kube_config()

v1 = client.CoreV1Api()
for pods in v1.list_pod_for_all_namespaces(label_selector=selector_str, ).items:
    print(pod.metadata.namespace, pod.metadata.name)
```
````

`````

### `kr8s` vs `kubernetes_asyncio`

The official `kubernetes` library doesn't support asyncio so the `kubernetes_asyncio` exists to fill that gap. It is created in the same way by auto generating a library using an asyncio OpenAPI generator.

The code that is needed to use this library is the most verbose out of all of the libraries due to the use of async context managers for HTTP sessions and the documentation is even more minimal than the official library. Often developers need to look at the official docs and then try and translate it into `kubernetes_asyncio` code.

Here's an example of listing the Nodes in your cluster:

`````{tab-set}

````{tab-item} kr8s (async)
```python
import kr8s.asyncio

for node in await kr8s.asyncio.get("nodes"):
    print(node.name)
```
````

````{tab-item} kubernetes_asyncio
```python
from kubernetes_asyncio import client, config
from kubernetes_asyncio.client.api_client import ApiClient

await config.load_kube_config()

async with ApiClient() as api:
    v1 = client.CoreV1Api(api)
    nodes = await v1.list_node()
    for node in nodes.items:
        print(node.metadata.name)
```
````

`````

### `kr8s` vs `pykube-ng`

`pykube-ng` is a maintained fork of `pykube` which aims to be a lightweight and pythonic client. It uses no code generation and produces more readable code but doesn't support asyncio. It also has a very object driven API which can result in overly complex looking queries to do simple things like get a single resource.

Here's an example listing ready Pods:

`````{tab-set}

````{tab-item} kr8s
```python
import kr8s

for pod in kr8s.get("pods", namespace="kube-system"):
    if pod.ready():
        print(pod.name)
```
````

````{tab-item} pykube-ng
```python
import pykube

api = pykube.HTTPClient(pykube.KubeConfig.from_file())
for pod in pykube.Pod.objects(api).filter(namespace="kube-system"):
    if pod.ready:
        print(pod.name)
```
````

`````

### `kr8s` vs `lightkube`

Lightkube feels like the Typescript of the Python Kubernetes Client landscape. It has a strong emphasis on type safety and has a partially auto generated codebase to ensure strict schema validation on the client side, but manages to balance that well with a pleasant and pythonic API and supports both sync and async usage.

It feels like the API client that `kubernetes` + `kubernetes_asyncio` could've been. This is a different design goal to `kr8s` which is trying to replicate the `kubectl` experience in Python rather than exposing the Kubernetes HTTP API.

Here's an example of scaling a Deployment:

`````{tab-set}

````{tab-item} kr8s
```python
from kr8s.objects import Deployment

deploy = Deployment("metrics-server", namespace="kube-system")
deploy.scale(1)
```
````

````{tab-item} lightkube
```python
from lightkube import Client
from lightkube.resources.apps_v1 import Deployment
from lightkube.models.meta_v1 import ObjectMeta
from lightkube.models.autoscaling_v1 import ScaleSpec

client = Client()
obj = Deployment.Scale(
    metadata=ObjectMeta(name='metrics-server', namespace='kube-system'),
    spec=ScaleSpec(replicas=1)
)
client.replace(obj)
```
````

`````
