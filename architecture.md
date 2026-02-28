# Kubernetes Architecture: The Operational View

To understand how our infrastructure scales automatically, it helps to view the Kubernetes cluster not as a group of servers, but as a well-structured organization. 

The system is divided into two main parts: **The Control Plane** (The Headquarters) and the **Worker Nodes** (The Execution Team).



---

## 🏢 The Control Plane (Headquarters)
This is the brain of the cluster. It makes global decisions, responds to events, and ensures the system matches our desired state.

* **API Server (The Project Manager):** This is the front door. Every single request—whether from a developer's laptop or an internal system—goes through the API Server. It validates the request and coordinates the rest of the HQ to make it happen. You never talk to the other components directly; you only talk to the PM.
* **etcd (The Source of Truth):** This is the highly secure filing cabinet. It is a database that stores the exact "desired state" of the entire cluster. If the power goes out, `etcd` is what tells the API Server what the infrastructure *should* look like when the lights come back on.
* **Scheduler (The Resource Allocator):** When the API Server receives an order for a new application container, the Scheduler looks at the available execution teams (Worker Nodes). It calculates who has the most CPU and RAM available and assigns the workload to the most capable node.
* **Controller Manager (The QA Lead):** This component constantly watches the actual state of the system and compares it to the desired state in `etcd`. If a container crashes, the Controller Manager notices the discrepancy and immediately orders a replacement.

---

## 🏗️ The Worker Nodes (The Execution Team)
These are the physical or virtual machines that do the actual heavy lifting. They run the application code and report back to Headquarters.

* **Kubelet (The Node Foreman):** This is the local agent living on every worker node. It takes direct orders from the API Server ("Start this container") and reports back on the health of the applications. If a container dies, the Kubelet is the one that yells "Code Blue!" to HQ.
* **Kube-proxy (The Traffic Cop):** Because containers are constantly being created and destroyed, their IP addresses change constantly. Kube-proxy is the networking agent that maintains local routing rules, ensuring that when a user tries to access the FinTech app, the network packets actually find the correct, living container.
* **Container Runtime (The Engine):** The underlying software (like Docker or containerd) that the Kubelet uses to actually pull the image and run the isolated process.