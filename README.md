# Instruqt Track: Kubernetes Cluster-as-a-Service with Aviatrix DCF

Converted from: https://github.com/AviatrixSystems/aviatrix-blueprints/tree/main/blueprints/k8s-cluster-aas

## Track Structure

```
k8s-cluster-aas/
├── track.yml                          # Track metadata
├── 01-prerequisites/
│   ├── config.yml                     # Architecture overview, tool verification
│   ├── setup-workstation              # Installs tools, clones repo
│   └── check-workstation              # Verifies tools + controller access
├── 02-network-layer/
│   ├── config.yml                     # Layer 1: Transit, VPCs, DCF
│   └── check-workstation              # Verifies VPCs + DCF rules exist
├── 03-cluster-layer/
│   ├── config.yml                     # Layer 2: EKS control planes (parallel)
│   └── check-workstation              # Verifies 3 clusters ACTIVE + kubectl contexts
├── 04-node-layer/
│   ├── config.yml                     # Layer 3: Node groups + Helm charts (parallel)
│   └── check-workstation              # Verifies nodes Ready + system pods Running
├── 05-dcf-policy/
│   ├── config.yml                     # DCF deep dive: SmartGroups, rules, design
│   └── check-workstation              # Verifies policy rules + SmartGroups count
├── 06-traffic-tests/
│   ├── config.yml                     # Deploy test workloads, run 8-test suite
│   └── check-workstation              # Verifies 8/8 tests passed
└── 07-cleanup/
    ├── config.yml                     # Reverse-order destroy
    └── check-workstation              # Verifies all AWS + Aviatrix resources gone
```

## Estimated Time

| Challenge | Activity | Time |
|-----------|----------|------|
| 01 | Prerequisites | 10 min |
| 02 | Network layer | 15 min (8 min deploy) |
| 03 | Cluster layer | 20 min (15 min deploy) |
| 04 | Node layer | 15 min (8 min deploy) |
| 05 | DCF policy | 15 min |
| 06 | Traffic tests | 15 min |
| 07 | Cleanup | 20 min |
| **Total** | | **~90 min** |

## Required Environment Variables (set by Instruqt sandbox)

| Variable | Description |
|----------|-------------|
| `AVIATRIX_CONTROLLER_IP` | IP/hostname of the Aviatrix Controller |
| `AVIATRIX_PASSWORD` | Controller admin password |
| `AVIATRIX_CID` | Pre-authenticated session token |
| `AVIATRIX_COPILOT_IP` | CoPilot IP for browser tabs |
| `AVIATRIX_AWS_ACCOUNT_NAME` | Aviatrix access account name in the controller |

## Instruqt Sandbox Requirements

- **Workstation** host: Ubuntu 22.04 LTS, outbound internet
- **AWS account**: Pre-onboarded in Aviatrix controller, sufficient EC2/EKS/VPC quotas
- **Aviatrix Controller**: Running, accessible from workstation, version supporting DCF + SmartGroups
- **Region**: us-west-2 (configurable via `aws_region` variable)

## Deploying This Track

```bash
# Install Instruqt CLI
# https://docs.instruqt.com/reference/cli/install

instruqt track push --directory ./k8s-cluster-aas

# Test locally
instruqt track test --directory ./k8s-cluster-aas
```

## Design Notes from Blueprint

1. `excluded_advertised_spoke_routes` goes on the **transit**, not spokes
2. Always deny **both directions** for isolated team pairs (asymmetric rules cause asymmetric routing)
3. Use **Public Internet SmartGroup** for default deny, not `0.0.0.0/0`
4. Calico runs in **policy-only mode** — VPC CNI handles pod IPs, Calico adds K8s NetworkPolicy
5. DCF sees **post-SNAT** traffic — use VPC SmartGroups for source, not pod CIDRs
