# Load Aware Scheduling

In 2020, a KEP proposed introducing a "Real Load Aware Scheduling" scheduler plugin called "Trimaran" to the Kubernetes' scheduler plugins repo.[^1]

In its first and current draft, it focuses on providing CPU load-aware scheduling and does not focus on "Memory, Network, and Disk utilization\[...]" [^2]

[^1]:<https://github.com/kubernetes-sigs/scheduler-plugins/tree/master/kep/61-Trimaran-real-load-aware-scheduling>
[^2]: <https://github.com/kubernetes-sigs/scheduler-plugins/tree/master/kep/61-Trimaran-real-load-aware-scheduling#non-goals>
