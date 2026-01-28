# SRE Metrics Overview

In Site Reliability Engineering (SRE), metrics are used to measure the health, performance, and reliability of a system. These metrics are commonly organized into well-known frameworks, with the **Golden Signals** being the industry standard.

---

## 1. The Four Golden Signals

Introduced by Google, these are the most critical metrics for any user-facing service:

### Latency
The time it takes to service a request.  
It is important to distinguish between:
- **Successful request latency**
- **Failed request latency** (for example, HTTP 500 errors may return quickly and hide slow successful requests)

### Traffic
A measure of how much demand is being placed on your system. Common measurements include:
- Requests per second (RPS)
- Concurrent sessions
- Transactions per second (TPS)

### Errors
The rate of requests that fail:
- Explicit failures (HTTP 5xx)
- Implicit failures (HTTP 200 with incorrect content)
- Policy failures (requests exceeding an SLO, such as >1s latency)

### Saturation
A measure of how “full” your service is. It reflects the most constrained resources such as:
- CPU
- Memory
- Disk
- Network or I/O

Most systems experience performance degradation before reaching 100% saturation.

---

## 2. Core SRE Monitoring Frameworks (RED & USE)

While the Golden Signals provide a high-level view, SREs often rely on two specialized frameworks depending on the focus area.

### The RED Method (Service-Focused)

Ideal for microservices and request-driven architectures:

- **Rate** – Number of requests per second
- **Errors** – Number of failed requests per second
- **Duration** – Time taken to serve requests (latency)

---

### The USE Method (Resource-Focused)

Ideal for infrastructure-level monitoring (nodes, disks, CPUs):

- **Utilization** – Average time the resource is busy
- **Saturation** – Degree to which the resource has queued or excess work
- **Errors** – Count of error events at the resource level

---

## 3. Incident & Change Metrics

These metrics measure operational effectiveness and release stability:

### MTTD (Mean Time to Detect)
The average time taken to detect an incident after it begins.

### MTTR (Mean Time to Resolve)
The average time required to restore the system to normal operation.

### CFR (Change Failure Rate)
The percentage of deployments or releases that cause a production failure.

### Availability
The percentage of time the system is functional and accessible, often expressed in “nines”:
- 99.9%
- 99.99%

---

## 4. SLIs, SLOs, and Error Budgets

These are reliability management tools rather than raw metrics.

### SLI (Service Level Indicator)
A specific, measurable metric such as:
- p99 latency
- Error rate

### SLO (Service Level Objective)
The target value for an SLI.  
Example:
> 99% of requests must complete in under 200ms

### Error Budget
The acceptable level of failure:
```
Error Budget = 100% – SLO
```

For example:
- SLO = 99.9% availability
- Error Budget = 0.1% of requests or time

Error budgets help teams balance reliability and velocity.

---

*End of document*
