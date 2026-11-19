# ADR-001: Amazon Lightsail with cPanel instead of EC2

**Status:** Accepted

## Context
A cPanel/WHM server hosting ~80 domains was 92% full and costing about R2,470 per month, with no scalability. The client was cost-sensitive and needed predictable billing. cPanel had to be kept because the client's customers and processes depend on it.

## Options
1. **Lightsail + cPanel:** 4 vCPU / 8 GB / 160 GB SSD plus 160 GB block storage, flat monthly pricing, static IP included.
2. **EC2 (t3.xlarge) + EBS gp3 500 GB + cPanel:** 16 GB RAM and far more headroom, with a 1-year reserved instance.

## Decision
Option 1. It is the closest match to the existing server experience, resolves the disk risk, and is about R700 per month cheaper than the current server (about R1,912 against R2,470). EC2 was about R150 per month more than the current server.

## Consequences
- Lower and predictable cost; simple to operate.
- Limited scalability. If the workload grows, move to EC2 using the same cPanel backups.
- Block storage and snapshots handle backups; a separate off-server copy is still recommended.
- cPanel licensing adds a fixed monthly cost to either option.

## Notes
Estimates were based on public AWS pricing at the time of writing and a fixed exchange rate, so they will drift.
