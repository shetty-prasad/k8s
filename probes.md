Kubernetes (K8s) probes are used to monitor the health of containers in a Pod. Probes help Kubernetes decide whether to:

Restart a container

Remove a container from service (load balancing)

Allow traffic to a container

There are three types of probes:

✅ 1. Liveness Probe

Purpose: Checks if the application is alive or dead.

If the liveness probe fails, Kubernetes will restart the container.

✅ 2. Readiness Probe

Purpose: Checks if the application is ready to accept traffic.

If it fails, the Pod is removed from the Service endpoint, but not restarted.

✅ 3. Startup Probe (Introduced in K8s 1.16+)

Purpose: Useful for applications that take a long time to start.

If defined, it runs first, and disables liveness and readiness checks until it succeeds.

🔍 Probe Configuration Types

Probes can use three mechanisms:

Type	Description

* HTTP GET :-	Sends an HTTP GET request to the container.
* TCP Socket :-	Attempts to open a TCP connection.
* Exec	:- Executes a command inside the container.

🛠️ Example: HTTP Liveness Probe
```
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 5
Explanation:
```
initialDelaySeconds: Wait 10s before first probe.

periodSeconds: Probe every 5s.

If the /health endpoint on port 8080 fails, the container is restarted.

🧠 Key Probe Parameters
Field	Purpose
initialDelaySeconds	Wait time before first check
periodSeconds	How often to perform the probe
timeoutSeconds	Timeout for the probe
successThreshold	Number of successes to consider it healthy
failureThreshold	Number of failures to consider it unhealthy

![image](https://github.com/user-attachments/assets/e2404fac-9bdf-4e81-8c05-a95cec5ff164)

🎯 Interview Questions with Answers
🔸 Q1: What is the difference between liveness and readiness probes?
Answer:

Liveness probe checks if the app is alive; failure causes a restart.

Readiness probe checks if the app is ready to serve requests; failure removes it from the service without restarting.

🔸 Q2: What happens if a readiness probe fails?

The pod is temporarily removed from the list of available endpoints. It won't receive traffic until it becomes ready again. The container is not restarted.

🔸 Q3: Can you use both liveness and readiness probes together?

Yes, and it's common. Liveness ensures the app is running, readiness ensures it’s ready to serve traffic.

🔸 Q4: What is the use of a startup probe?

Startup probe is useful for slow-starting applications. It gives the container time to initialize without triggering liveness failures.

🔸 Q5: What are the different ways to implement a probe?

Three ways:

HTTP GET: Call a health endpoint.

TCP Socket: Check if a port is open.

Exec: Run a custom command inside the container.

🔸 Q6: When would you prefer TCP or Exec over HTTP probes?

TCP: When the app doesn't expose HTTP but opens a port (like DBs).

Exec: When you want custom logic, like checking a file or process.

🔸 Q7: What happens if you don’t define any probes?

Kubernetes assumes the container is always healthy and will not restart it automatically even if it hangs or crashes silently.

🔸 Q8: How does Kubernetes react to a failing liveness probe?

It restarts the container using the defined restartPolicy.

🔸 Q9: What’s the effect of setting successThreshold: 3 in a readiness probe?

The probe must succeed 3 consecutive times before the pod is considered ready and included in service endpoints.

🔸 Q10: How do you test your probe locally before applying to a cluster?

* Run the same health check endpoint manually with curl.
* Use kubectl exec to run the same command as the exec probe.
* Check logs and readiness using kubectl describe pod.
* Would you like a hands-on YAML setup example for all 3 probes in one pod?

