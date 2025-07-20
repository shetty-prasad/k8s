The primary responsibility of the Kubernetes scheduler is to **assign pods to nodes based on a series of criteria. The creation of the pods is handled by kubectl .**
This ensures that the selected node has sufficient resources and meets any specific requirements. 
For instance, different nodes may be designated for certain applications or come with varied resource capacities. 
When multiple pods and nodes are involved, the scheduler assesses each pod against the available nodes through a two-phase process: filtering and ranking.
