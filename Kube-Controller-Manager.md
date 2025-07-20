In Kubernetes, a controller acts like a department in an organization—each controller is tasked with handling a specific responsibility. 
For instance, one controller might monitor the health of nodes, while another ensures that the desired number of pods is always running. 
These controllers constantly observe system changes to drive the cluster toward its intended state.

The Node Controller, for example, checks node statuses every five seconds through the Kube API Server. 
If a node stops sending heartbeats, it is not immediately marked as unreachable; 
instead, there is a grace period of 40 seconds followed by an additional five minutes for potential recovery before its pods are rescheduled onto a healthy node.

<img width="1461" height="789" alt="image" src="https://github.com/user-attachments/assets/ddc9aa3d-76ca-4713-879e-f7cd0d323f23" />

<img width="971" height="769" alt="image" src="https://github.com/user-attachments/assets/c154a61a-1730-4fae-ae10-cc7f57dab938" />



