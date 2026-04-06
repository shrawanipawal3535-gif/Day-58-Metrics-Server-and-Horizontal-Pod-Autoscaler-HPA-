# Day-58-Metrics-Server-and-Horizontal-Pod-Autoscaler-HPA

# Task

Yesterday you set resource requests and limits. Today you put that to work. Install the Metrics Server so Kubernetes can see actual resource usage, then set up a Horizontal Pod Autoscaler that scales your app up under load and back down when things calm down.

# Expected Output

- Metrics Server installed and kubectl top returning data
- An HPA that auto-scales pods under load
- A markdown file: day-58-metrics-hpa.md

# Challenge Tasks

# Task 1: Install the Metrics Server

1. Check if it is already running: kubectl get pods -n kube-system | grep metrics-server
2. If not, install it:
   - Minikube: minikube addons enable metrics-server
   - Kind/kubeadm: apply the official manifest from the metrics-server GitHub releases
3. On local clusters, you may need the --kubelet-insecure-tls flag (never in production)
4. Wait 60 seconds, then verify: kubectl top nodes and kubectl top pods -A

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/7ea23c10-3448-46d1-8f45-252994faaab7" />



# Task 2: Explore kubectl top

1. Run kubectl top nodes, kubectl top pods -A, kubectl top pods -A --sort-by=cpu
2. kubectl top shows real-time usage, not requests or limits — these are different things
3. Data comes from the Metrics Server, which polls kubelets every 15 seconds

<img width="1336" height="932" alt="Image" src="https://github.com/user-attachments/assets/5abd2c70-c6be-4743-813b-7afa094d49eb" />







# Task 4: Create an HPA (Imperative)

1. Run: kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
2. Check: kubectl get hpa and kubectl describe hpa php-apache
3. TARGETS may show <unknown> initially — wait 30 seconds for metrics to arrive

This scales up when average CPU exceeds 50% of requests, and down when it drops below.

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/c8601daa-a887-4be0-9204-5b12b1e5908f" />










# Task 5: Generate Load and Watch Autoscaling

1. Start a load generator: kubectl run load-generator --image=busybox:1.36 --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://php-apache; done"
2. Watch HPA: kubectl get hpa php-apache --watch
3. Over 1-3 minutes, CPU climbs above 50%, replicas increase, CPU stabilizes
4. Stop the load: kubectl delete pod load-generator
5. Scale-down is slow (5-minute stabilization window) — you do not need to wait

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/ce2950ea-75ce-40b9-9d82-154640a0bf32" />






# Task 6: Create an HPA from YAML (Declarative)

1. Delete the imperative HPA: kubectl delete hpa php-apache
2. Write an HPA manifest using autoscaling/v2 API with CPU target at 50% utilization
3. Add a behavior section to control scale-up speed (no stabilization) and scale-down speed (300 second window)
4. Apply and verify with kubectl describe hpa

autoscaling/v2 supports multiple metrics and fine-grained scaling behavior that the imperative command cannot configure.

<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/07a74848-a0e1-4eab-9965-5793a29d72d2" />

# Task 7: Clean Up

Delete the HPA, Service, Deployment, and load-generator pod. Leave the Metrics Server installed.



<img width="1920" height="1080" alt="Image" src="https://github.com/user-attachments/assets/91c32db0-140e-4246-9a5e-2f7719a2d863" />
