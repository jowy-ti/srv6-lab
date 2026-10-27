# SRv6 Lab - Quick Start Guide

## Steps to Execute

1. **Bootstrap the environment** (installs packages, sets up venv, configures sudoers)
   ```bash
   make setup
   ```

2. **Run preflight checks** (verify deps, FRR, kernel seg6, topology parsing)
   ```bash
   make doctor
   ```

3. **Validate the topology** (optional, no root required)
   ```bash
   make check
   ```

4. **Start the lab**
   ```bash
   make run
   ```
   - At the `mininet>` prompt, run your tests. 
   - Type `exit` to tear down the lab.

5. **Clean up after crashes/Ctrl-C**
   ```bash
   make clean
   ```

> **Note:** The topology YAML file **must be named `srv6.yaml`** and placed in the `topologies/` directory. The default entrypoint reads from `topologies/srv6.yaml`.

## Topology YAML Schema (Brief Guide)

Define your topology in `topologies/srv6.yaml` with the following structure:

```yaml
hosts:
  - name: h1        # Host node
  - name: h2

routers:
  - name: s1                       # Router name
    loopback: 2001:db8:1::1/64     # Required if defining SIDs
    ospf6: true                   # Optional (default: true)
    sids:                         # Optional local SIDs
      - address: 2001:db8:1::10   # /128, must be within loopback locator
        behavior: end.dx6         # end | end.x | end.dx6 (default: end.dx6)
        to: h1                    # Required for end.x/end.dx6, forbidden for end
        cost: 1                   # Optional (default: 1)

links:
  - [h1, s1]                      # Simple link

policies:                        # SRv6 steering policies
  - node: s1                     # Ingress router (must be a router)
    to: h2                       # Destination node
    through: [2001:db8:4::10, 2001:db8:6::10]  # Segment list (node names or IPv6)
    mode: encap                  # encap | inline (default: encap)
    cost: 1                      # Optional (default: 1)
```

**Key rules:**
- `routers` must have at least one entry.
- If a router has `sids`, it must have a `loopback` and each SID must be within that locator.
- `end` behavior is a transit waypoint (no `to`), `end.dx6`/`end.x` deliver to a local node via `to`.
- `policies.through` can contain router/node names or explicit IPv6 addresses.