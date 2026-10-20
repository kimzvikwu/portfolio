# eBPF for Optimal Cluster Use

---

## 1. Operating System Limitations

Today's standard operating systems don't enable more than one Kubernetes (K8s) cluster, with the exception of:

- Hopper
- Ampere

---

## 2. Cost Issues

- NVIDIA GPUs run only one cluster at a time
- GPUs are very costly
- Berkeley Lab runs only one cluster on its Crays
  - Crays are astronomically costly
  - Crays take five years to purchase

---

## 3. Performance Issues

- If you run Ampere or Hopper, you need to tune cluster performance
- K8s trades off with requests and limits:
  - Memory
  - CPU
  - Hugepages
- What is the optimal request-versus-limit ratio to avoid thrashing?
- Hopper is 4.5 times faster than Ampere

---

## 4. eBPF Performance Tools

*(Diagram slide with legend — not reproduced. See the Brendan Gregg BPF tracing tools reference below.)*

---

## 5. eBPF to the Rescue!

The extended Berkeley Packet Filter (eBPF) allows you to tune performance across:

- **K8s:** cgroups, event tracing, syscalls, namespaces
- **Kernel:** sockets, TCP, IP, network
- **Wake stacks**

---

## 6. eBPF Has Security Considerations

There are key security issues with eBPF that need to be considered:

- Malicious kernel requests
- `LD_PRELOAD`
- TOCTOU — ptrace, seccomp, eBPF probe on syscall entry
- eBPF LSM — security monitoring on stable kernel modules

**Definitions**

- **TOCTOU** — time-of-check to time-of-use
- **LSM** — Linux Security Module

**Mitigations**

- Use gVisor to trace kernel calls, then secure those calls with seccomp (`runsc`)
- Use a different runtime class for non-default K8s workloads
- `SIGKILL` asynchronously

---

## References

- eBPF Summit 2023 — https://www.youtube.com/watch?v=EViAho-6qoc
- WCCFTech, NVIDIA Hopper H100 MLPerf results — https://wccftech.com/nvidia-clean-sweeps-mlperf-ai-benchmarks-with-hopper-h100-gpu-up-to-4-5x-performance-uplift-over-ampere-a100/
- Brendan Gregg, *Systems Performance* (Addison-Wesley Professional Computing Series)
- Brendan Gregg, Linux 4.x Tracing Tools: Using BPF Superpowers — https://www.slideshare.net/brendangregg/linux-4x-tracing-tools-using-bpf-superpowers
- CNCF Europe — https://www.youtube.com/watch?v=Xs3MBK17kCk
- Udemy, Certified Kubernetes Security Specialist — https://www.udemy.com/course/certified-kubernetes-security-specialist-certification
