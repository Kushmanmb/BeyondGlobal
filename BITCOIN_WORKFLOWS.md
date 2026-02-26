# Bitcoin Workflows Documentation

This repository includes comprehensive GitHub Actions workflows for Bitcoin-related development, improvements, and scaling using self-hosted runners.

## Overview

The workflows are designed to support Bitcoin Unlimited improvements, proposals, and transaction monitoring with scalable architecture using self-hosted runners and validators.

## Workflows

### 1. Bitcoin Network Validator (`bitcoin-validator.yml`)

**Purpose:** Validates Bitcoin integration and network connectivity.

**Triggers:**
- Push to `main` or `develop` branches
- Pull requests to `main` or `develop`
- Daily at midnight UTC (scheduled)
- Manual dispatch

**Jobs:**
- `validate-bitcoin-integration`: Validates Bitcoin node connectivity and integration tests
- `scale-validation`: Tests concurrent operations for scalability

**Requirements:**
- Self-hosted runner with label: `bitcoin-validator`
- Optional: Bitcoin CLI for node validation

**Usage:**
```bash
# Manually trigger validation
gh workflow run bitcoin-validator.yml
```

---

### 2. Bitcoin Improvement Proposal (`bitcoin-improvement-proposal.yml`)

**Purpose:** Validates and reviews Bitcoin Improvement Proposals (BIPs).

**Triggers:**
- Issues opened or labeled
- Pull requests opened, synchronized, or labeled
- Manual dispatch

**Jobs:**
- `bip-validation`: Validates BIP structure and Bitcoin Unlimited compatibility
- `bip-review`: Performs automated review of proposals

**Requirements:**
- Self-hosted runner with label: `bitcoin-validator`
- Issues or PRs must have `BIP` label to trigger

**Usage:**
```bash
# Create a new BIP
# 1. Open an issue or PR with title containing "BIP"
# 2. Add label "BIP"
# 3. Workflow will automatically validate

# Manually trigger
gh workflow run bitcoin-improvement-proposal.yml
```

**Artifacts:**
- BIP validation report (retained for 90 days)

---

### 3. Bitcoin Transaction Monitor (`bitcoin-transaction-monitor.yml`)

**Purpose:** Monitors Bitcoin transactions and blockchain activity.

**Triggers:**
- Every 15 minutes (scheduled)
- Push to `main` branch (when index.js or package.json changes)
- Manual dispatch

**Jobs:**
- `monitor-transactions`: Monitors blockchain activity and validates transaction integrity
- `alert-on-anomalies`: Sends alerts when anomalies are detected (runs on failure)

**Requirements:**
- Self-hosted runner with label: `bitcoin-validator`
- Optional: Bitcoin node for enhanced monitoring

**Usage:**
```bash
# Manually trigger monitoring
gh workflow run bitcoin-transaction-monitor.yml
```

**Artifacts:**
- Bitcoin monitoring reports (retained for 30 days)

---

### 4. Bitcoin Scaling and Performance (`bitcoin-scaling.yml`)

**Purpose:** Tests Bitcoin-related operations at scale.

**Triggers:**
- Push to `main` or `develop` branches
- Pull requests to `main`
- Weekly on Sundays at 2 AM UTC (scheduled)
- Manual dispatch with scale level option

**Jobs:**
- `performance-baseline`: Establishes performance baseline
- `scale-test-small`: Tests with 10 concurrent operations
- `scale-test-medium`: Tests with 50 concurrent operations
- `scale-test-large`: Tests with 100 concurrent operations
- `bitcoin-unlimited-scale`: Tests Bitcoin Unlimited-specific scaling

**Requirements:**
- Self-hosted runner with label: `bitcoin-validator`

**Usage:**
```bash
# Run with default (medium) scale
gh workflow run bitcoin-scaling.yml

# Run with specific scale level
gh workflow run bitcoin-scaling.yml -f scale_level=large
```

**Artifacts:**
- Performance baseline metrics
- Scaling reports (retained for 90 days)

---

### 5. Bitcoin Work Management (`bitcoin-work-management.yml`)

**Purpose:** Categorizes and manages Bitcoin-related work items.

**Triggers:**
- Issues opened, edited, labeled, or assigned
- Pull requests opened, edited, labeled, assigned, or review requested
- Manual dispatch

**Jobs:**
- `categorize-work`: Automatically categorizes Bitcoin-related work items
- `validate-work-item`: Validates work items against Bitcoin standards
- `track-work-progress`: Tracks work progress and generates summaries
- `manage-bitcoin-proposals`: Special handling for proposals

**Requirements:**
- Self-hosted runner with label: `bitcoin-validator`

**Usage:**
- Automatic: Workflows trigger on issue/PR events
- Manual: `gh workflow run bitcoin-work-management.yml`

**Detection:**
- Bitcoin-related keywords: bitcoin, btc, crypto, blockchain
- Bitcoin Unlimited: "Bitcoin Unlimited", BU
- Proposals: Label with `proposal`

**Artifacts:**
- Work summaries (retained for 30 days)

---

## Self-Hosted Runner Setup

### Prerequisites

1. A dedicated machine (physical or virtual) for running workflows
2. Ubuntu 20.04+ or similar Linux distribution
3. Docker (optional, for containerized runners)
4. Bitcoin node (optional, for enhanced validation)

### Basic Setup

```bash
# 1. Create a runner directory
mkdir -p ~/actions-runner && cd ~/actions-runner

# 2. Download the latest runner package
curl -o actions-runner-linux-x64-2.311.0.tar.gz -L https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-linux-x64-2.311.0.tar.gz

# 3. Extract the installer
tar xzf ./actions-runner-linux-x64-2.311.0.tar.gz

# 4. Configure the runner
./config.sh --url https://github.com/Kushmanmb/BeyondGlobal --token YOUR_TOKEN --labels bitcoin-validator

# 5. Install as a service
sudo ./svc.sh install
sudo ./svc.sh start
```

### Adding Labels

The workflows use the `bitcoin-validator` label. Ensure your runner is configured with this label:

```bash
./config.sh --url https://github.com/YOUR_ORG/YOUR_REPO \
  --token YOUR_TOKEN \
  --labels bitcoin-validator,self-hosted
```

### Bitcoin Node Setup (Optional)

For enhanced monitoring and validation:

```bash
# Install Bitcoin Core
sudo add-apt-repository ppa:bitcoin/bitcoin
sudo apt-get update
sudo apt-get install bitcoind bitcoin-qt

# Configure bitcoin.conf
mkdir -p ~/.bitcoin
cat > ~/.bitcoin/bitcoin.conf << EOF
server=1
rpcuser=bitcoinrpc
rpcpassword=YOUR_SECURE_PASSWORD
rpcallowip=127.0.0.1
txindex=1
EOF

# Start Bitcoin daemon
bitcoind -daemon
```

---

## Workflow Configuration

### Environment Variables

Set these in your repository settings (Settings → Secrets and variables → Actions):

- `BITCOIN_RPC_USER`: Bitcoin RPC username (if using Bitcoin node)
- `BITCOIN_RPC_PASSWORD`: Bitcoin RPC password (if using Bitcoin node)
- `BITCOIN_RPC_HOST`: Bitcoin node host (default: localhost)

### Runner Requirements

**Minimum specifications:**
- CPU: 4 cores
- RAM: 8 GB
- Disk: 50 GB free space
- Network: Stable internet connection

**Recommended specifications (for large-scale tests):**
- CPU: 8+ cores
- RAM: 16+ GB
- Disk: 100+ GB free space
- Network: High-bandwidth connection

---

## Monitoring and Debugging

### View Workflow Runs

```bash
# List recent workflow runs
gh run list --workflow=bitcoin-validator.yml

# View specific run
gh run view RUN_ID

# View logs
gh run view RUN_ID --log
```

### Download Artifacts

```bash
# List artifacts from a run
gh run view RUN_ID --log-failed

# Download artifacts
gh run download RUN_ID
```

### Check Runner Status

```bash
# On the runner machine
sudo ./svc.sh status

# View runner logs
journalctl -u actions.runner.*
```

---

## Best Practices

1. **Security:**
   - Use dedicated runners for Bitcoin-related workflows
   - Keep runners updated with security patches
   - Use secrets for sensitive configuration
   - Restrict runner access to trusted repositories

2. **Performance:**
   - Monitor runner resource usage
   - Scale runners horizontally for parallel jobs
   - Use caching for dependencies
   - Clean up old workflow runs and artifacts

3. **Reliability:**
   - Monitor workflow success rates
   - Set up alerts for critical failures
   - Maintain runner redundancy
   - Regular health checks

4. **Maintenance:**
   - Review workflow logs regularly
   - Update dependencies periodically
   - Archive old artifacts
   - Document workflow changes

---

## Troubleshooting

### Runner Not Connecting

```bash
# Check runner status
sudo ./svc.sh status

# Restart runner
sudo ./svc.sh restart

# View logs
journalctl -u actions.runner.* -f
```

### Workflow Failures

1. Check workflow logs in GitHub Actions tab
2. Verify runner has required labels
3. Ensure dependencies are installed
4. Check Bitcoin node connectivity (if applicable)

### Bitcoin Node Issues

```bash
# Check Bitcoin node status
bitcoin-cli getblockchaininfo

# Restart Bitcoin node
bitcoin-cli stop
bitcoind -daemon

# View Bitcoin logs
tail -f ~/.bitcoin/debug.log
```

---

## Contributing

When adding new Bitcoin-related workflows:

1. Follow existing naming conventions
2. Use `bitcoin-validator` runner label
3. Include proper error handling
4. Document new workflows in this file
5. Test workflows before merging

---

## Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Self-Hosted Runners Guide](https://docs.github.com/en/actions/hosting-your-own-runners)
- [Bitcoin Unlimited Documentation](https://www.bitcoinunlimited.info/)
- [Bitcoin Core Documentation](https://bitcoin.org/en/bitcoin-core/)

---

## Support

For issues or questions:
- Open an issue in this repository
- Check workflow run logs
- Review runner logs
- Consult Bitcoin community resources
