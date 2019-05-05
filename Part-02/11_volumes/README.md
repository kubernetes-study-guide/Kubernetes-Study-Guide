# storing data

So far we have seen that, in terms of:

- Data Retention - When a container dies, any data that container gets deleted. The same is for pods too.
- Data Sharing - The files/folders stored in a container's filesystem isn't accessible by any other containers/pods

However Kubernetes comes with features that lets you change the above default behaviours

## Data Retention

keep your data even if/when the container/pod dies.

Kubernetes also let's you share a container's folder with another container/pod. There are