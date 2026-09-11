# Kubernetes Basics

* What it is:
    * Container orchestration service
    * Manages workloads, self healing, load balancing, service discovery, networking, storage, deployments
* Super high level:
    * **Pods** are the units that run your application.
    * **Deployments** maintain the desired number of pods and enable rolling updates.
    * **Services** provide stable networking and load balancing to pods.
    * **Autoscaling** adjusts capacity as demand changes.
    * **Self-healing** replaces failed pods automatically.
    * **Scheduling** spreads workloads across nodes while respecting resource constraints.
    * **Rolling deployments** reduce downtime during releases.
    * **Kubernetes orchestrates containers**—it is not your app itself, but the platform that keeps it running reliably.
