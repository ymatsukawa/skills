# Standard SLO/SLI Web

## Critical
| SLI | SLO | Reason |
|---|---|---|
| **Request Availability** | `99.9% = (5XX response / all response)` in month | Base of web availability |
| **Request Error Rate** | `5xx < 0.1%` in "all requests" | Boundary that users feel "broken" or not. |
| **Request Latency** | `< 200ms of 99% requests` | Slow response makes users leaving. |

## High
| SLI | SLO | Reason |
|---|---|---|
| **Latency p95** | `p95 < 500ms` in month | Protects the typical user, not just outliers. |
| **LCP; Largest Contentful Paint** | `p75 < 2.5s` | Slow load hurts UX and SEO. |
| **INP; Interaction to Next Paint** | `p75 < 200ms` | Unresponsive UI feels broken. |
| **TTFB; Time to First Byte** | `p75 < 800ms` | Early signal of backend slowness. |
| **Throughput** | anomaly within `±X%` of baseline | Sudden drop or spike means trouble. |

## Medium
This is depends on web site specification.

| SLI | SLO | Reason |
|---|---|---|
| **CLS; Cumulative Layout Shift** | `p75 < 0.1` | Layout jumps cause mis-taps. |
| **Saturation (CPU/Memory)** | utilization `< 80%` | Saturation precedes outages. |
| **Frontend JS Error Rate** | error sessions `< 1%` | Client-side breakage invisible to 5xx. |
| **Critical Path Availability (Synthetic)** | key flow success `99.9%` | Guards login / checkout end to end. |
| **Dependency / API Error Rate** | external 5xx `< 0.5%` | Third-party failure spreads to you. |
| **Storage Capacity** | free space `> 20%` | Full disk breaks uploads and writes. |
