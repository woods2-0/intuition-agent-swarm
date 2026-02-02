# Spawn Your Own Intuition Agent Swarm

A complete guide to creating autonomous AI agents that evangelize, build for, and use the Intuition protocol.

**Hand this entire document to Claude Code and say: "Set this up for me."**

---

## What You're Building

A swarm of AI agents that:
- Have on-chain identity on Intuition (atoms, triples, stakes)
- Operate autonomously via cron jobs
- Coordinate with each other via intercom
- Build tools for Intuition adoption
- Engage with external agents/developers
- Use $TRUST to stake on claims

---

## Prerequisites

Before starting, you need:

| Requirement | Where to Get It |
|-------------|-----------------|
| VPS (Ubuntu recommended) | Contabo, DigitalOcean, Hetzner |
| Discord Bot Token | https://discord.com/developers/applications |
| Anthropic API Key | https://console.anthropic.com |
| $TRUST tokens | Bridge from Base at https://app.intuition.systems/bridge |
| Node.js 20+ | `curl -fsSL https://deb.nodesource.com/setup_20.x \| sudo -E bash - && sudo apt install -y nodejs` |

---

## Step 1: Install Clawdbot

Clawdbot is the agent framework that runs your agents.

```bash
# Install globally
npm install -g clawdbot

# Initialize config
clawdbot init

# This creates ~/.clawdbot/clawdbot.json
```

### Configure Anthropic API

```bash
# Set your API key
clawdbot config set anthropic.apiKey "sk-ant-..."

# Or use Claude CLI auth (if you have Claude Code installed)
# Clawdbot will auto-sync credentials
```

### Configure Discord

Edit `~/.clawdbot/clawdbot.json`:

```json
{
  "discord": {
    "enabled": true,
    "token": "YOUR_DISCORD_BOT_TOKEN",
    "defaultGuildId": "YOUR_SERVER_ID"
  }
}
```

### Start the Gateway

```bash
# Start gateway (runs in background)
clawdbot gateway &

# Or run as a service
clawdbot gateway install
```

---

## Step 2: Create Your First Agent

Each agent needs:
- A workspace directory
- SOUL.md (identity)
- GOALS.md (priorities)
- A Discord channel

### Create Workspace

```bash
# For an agent named "myagent"
mkdir -p ~/.clawdbot/workspace-myagent
```

### Create SOUL.md

```bash
cat > ~/.clawdbot/workspace-myagent/SOUL.md << 'EOF'
# [Agent Name]

## Identity
[1-2 sentences about who this agent is and what they do]

## Personality
- [Trait 1]
- [Trait 2]
- [Trait 3]

## Voice
[How they communicate - tone, style, quirks]

## Mission
[What they're trying to accomplish]

## Rules
- Never transfer $TRUST externally without approval
- Check ~/.clawdbot/PAUSED before autonomous actions
- Log actions to ~/.clawdbot/logs/[agent]-actions.log
- Max spend: 5 $TRUST/day without approval

## Autonomy Permissions
Allowed without asking:
- [List of allowed autonomous actions]

Needs approval:
- Spending over 5 $TRUST
- Creating accounts on new platforms
- Any action you're uncertain about

## Swarm Family
You are part of the Intuition agent swarm. Check `~/.clawdbot/family.json` for your siblings. Include them in relevant conversations. Check intercom (`~/.clawdbot/intercom/`) for their messages. When you learn something useful, share it with the family.
EOF
```

### Create GOALS.md

```bash
cat > ~/.clawdbot/workspace-myagent/GOALS.md << 'EOF'
# [Agent Name] Goals

## Daily Non-Negotiables
- [ ] 1 external interaction (NOT swarm family)
- [ ] 1 piece of external content posted
- [ ] Log interactions to ~/.clawdbot/logs/[agent]-outreach.log

## Weekly Targets
- [ ] Help 1 new agent/developer get on-chain identity
- [ ] 3 meaningful external conversations
- [ ] 1 original post about Intuition

## Standing Orders
- Max spend: 5 $TRUST/day, 25 $TRUST/week
- Log all on-chain actions
- Never transfer $TRUST externally without approval
- Check ~/.clawdbot/PAUSED before autonomous actions

## Current Priorities
1. [Priority 1]
2. [Priority 2]
3. [Priority 3]

## What Success Looks Like
External agents/developers actually using Intuition because of your work.
EOF
```

### Register Agent in Clawdbot

Edit `~/.clawdbot/clawdbot.json`, add to agents array:

```json
{
  "agents": [
    {
      "id": "myagent",
      "name": "My Agent",
      "workspace": "~/.clawdbot/workspace-myagent",
      "model": "claude-sonnet-4-20250514",
      "discord": {
        "channelId": "YOUR_DISCORD_CHANNEL_ID"
      }
    }
  ]
}
```

### Create Discord Channel

1. Create a text channel in your Discord server for this agent
2. Copy the channel ID (right-click > Copy ID)
3. Add it to the agent config above

---

## Step 3: Set Up Intuition Integration

### Install Intuition SDK

```bash
cd ~/.clawdbot/workspace-myagent
npm init -y
npm install @0xintuition/protocol viem
```

### Create Shared Knowledge Base

```bash
mkdir -p ~/.clawdbot/shared

cat > ~/.clawdbot/shared/INTUITION_CORE.md << 'EOF'
# Intuition Protocol — Core Knowledge

## What is Intuition?
Decentralized trust and reputation protocol. Agents create on-chain identity through atoms (concepts) and triples (relationships), then stake $TRUST on claims.

## Core Primitives

### Atoms
Fundamental units of knowledge. Created with `multiVaultCreateAtoms()`.
Example: `[AgentName]`, `[AI Agent]`, `[Trust]`

### Triples
Relationships between atoms: Subject-Predicate-Object.
Example: `[AgentName] [is] [AI Agent]`

### Stakes
$TRUST deposited on triples to signal belief.
Use `multiVaultDeposit()` to stake.

## Critical Technical Details

**$TRUST is the gas token.** On Intuition chain (ID 1155), $TRUST is used for:
- Gas fees for all transactions
- Creating atoms and triples
- Staking on claims

Do NOT check ETH balance. Check $TRUST balance using the Intuition RPC.

**Network Details:**
- Chain ID: 1155
- RPC: https://rpc.intuition.systems/http
- MultiVault: 0x430BbF52503Bd4801E51182f4cB9f8F534225DE5
- Bridge: https://app.intuition.systems/bridge
- Portal: https://portal.intuition.systems

## Known Atom IDs

```javascript
const KNOWN_ATOMS = {
  'is': '0xb0681668ca193e8608b43adea19fecbbe0828ef5afc941cef257d30a20564ef1',
  'AI Agent': '0x4990eef19ea1d9b893c1802af9e2ec37fbc1ae138868959ebc23c98b1fc9565e',
};
```

## Mission
Adoption over awareness. Success = agents and builders actually using Intuition, not just hearing about it.
EOF
```

### Create Wallet for Agent

Use the quickstart script or manually:

```bash
cat > ~/.clawdbot/workspace-myagent/create-wallet.mjs << 'EOF'
import { generatePrivateKey, privateKeyToAccount } from 'viem/accounts';
import { writeFileSync, mkdirSync } from 'fs';

const privateKey = generatePrivateKey();
const account = privateKeyToAccount(privateKey);

mkdirSync('.config', { recursive: true, mode: 0o700 });
writeFileSync('.config/wallet.json', JSON.stringify({
  address: account.address,
  privateKey: privateKey,
  created: new Date().toISOString(),
  chain: 'intuition-mainnet',
  chainId: 1155
}, null, 2), { mode: 0o600 });

console.log('Wallet created:', account.address);
console.log('Fund with $TRUST from: https://app.intuition.systems/bridge');
EOF

node ~/.clawdbot/workspace-myagent/create-wallet.mjs
```

### Fund the Wallet

1. Go to https://app.intuition.systems/bridge
2. Bridge $TRUST from Base to Intuition
3. Send to the agent's wallet address
4. Minimum recommended: 10 $TRUST

---

## Step 4: Set Up Intercom (Agent-to-Agent Communication)

```bash
mkdir -p ~/.clawdbot/intercom
```

### Intercom Protocol

Agents communicate by creating files in `~/.clawdbot/intercom/`:

**To send a message:**
```bash
cat > ~/.clawdbot/intercom/to-otheragent-$(date +%Y%m%d%H%M).md << 'EOF'
From: myagent
To: otheragent
Time: 2026-02-01 10:00

Your message here.
EOF
```

**To receive:** Agents check for files matching `to-[their-name]-*.md`

---

## Step 5: Set Up Family Registry

Track all agents in the swarm:

```bash
cat > ~/.clawdbot/family.json << 'EOF'
{
  "agents": {
    "myagent": {
      "wallet": "0x...",
      "role": "Description of role",
      "channel": "discord-channel-id"
    }
  },
  "rules": {
    "maxDailySpend": 5,
    "maxWeeklySpend": 25,
    "requireApprovalAbove": 5
  }
}
EOF
```

Update this file when adding new agents.

---

## Step 6: Set Up Cron Jobs (Autonomous Operation)

### Intercom Check (Every 5 minutes)

```bash
clawdbot cron add \
  --name "myagent-intercom" \
  --description "Check intercom for messages" \
  --agent myagent \
  --every 5m \
  --session isolated \
  --message "INTERCOM CHECK: Look in ~/.clawdbot/intercom/ for messages addressed to you. Check GOALS.md priorities. Take action on what matters." \
  --deliver \
  --to "channel:YOUR_CHANNEL_ID"
```

### Build/Work Check (Every 2-4 hours)

```bash
clawdbot cron add \
  --name "myagent-workcheck" \
  --description "Review goals and do work" \
  --agent myagent \
  --every 2h \
  --session isolated \
  --message "WORK CHECK: Review GOALS.md. What's the highest priority task? Do it. Log progress. If blocked, leave intercom message for relevant agent." \
  --deliver \
  --to "channel:YOUR_CHANNEL_ID"
```

---

## Step 7: Set Up Kill Switch

Create pause/resume scripts:

```bash
cat > ~/swarm-pause.sh << 'EOF'
#!/bin/bash
touch ~/.clawdbot/PAUSED
echo "Swarm paused. Agents will not take autonomous actions."
EOF
chmod +x ~/swarm-pause.sh

cat > ~/swarm-resume.sh << 'EOF'
#!/bin/bash
rm -f ~/.clawdbot/PAUSED
echo "Swarm resumed. Agents are autonomous again."
EOF
chmod +x ~/swarm-resume.sh
```

All agents check for `~/.clawdbot/PAUSED` before acting. If it exists, they only respond to direct prompts.

---

## Step 8: Install Quickstart Script (Agent Will Use This)

The agent will create its own identity during hatching. Install the script it will use:

```bash
cat > ~/.clawdbot/workspace-myagent/intuition-quickstart.mjs << 'SCRIPT'
#!/usr/bin/env node
/**
 * intuition-quickstart-v3.mjs
 * Complete agent onboarding: wallet + identity atom + [Agent] [is] [AI Agent] triple + stake
 *
 * Usage: node intuition-quickstart.mjs <agent_name> [stake_amount]
 * Example: node intuition-quickstart.mjs Forge 0.5
 */

import {
  createPublicClient,
  createWalletClient,
  http,
  formatEther,
  stringToHex,
  decodeEventLog
} from 'viem';
import { privateKeyToAccount, generatePrivateKey } from 'viem/accounts';
import {
  intuitionMainnet,
  getMultiVaultAddressFromChainId,
  multiVaultGetAtomCost,
  multiVaultGetTripleCost,
  multiVaultCreateAtoms,
  multiVaultCreateTriples,
  multiVaultDeposit,
  MultiVaultAbi,
} from '@0xintuition/protocol';
import { writeFileSync, readFileSync, existsSync, mkdirSync } from 'fs';
import { join } from 'path';

// Known atom IDs (from Intuition mainnet)
const KNOWN_ATOMS = {
  'is': '0xb0681668ca193e8608b43adea19fecbbe0828ef5afc941cef257d30a20564ef1',
  'AI Agent': '0x4990eef19ea1d9b893c1802af9e2ec37fbc1ae138868959ebc23c98b1fc9565e',
};

const AGENT_NAME = process.argv[2];
const STAKE_AMOUNT = process.argv[3] || '0.1';

if (!AGENT_NAME) {
  console.log('Intuition Agent Quickstart v3');
  console.log('============================');
  console.log('');
  console.log('Usage: node intuition-quickstart.mjs <agent_name> [stake_amount]');
  console.log('');
  console.log('Examples:');
  console.log('  node intuition-quickstart.mjs Forge 0.5');
  console.log('  node intuition-quickstart.mjs MyBot');
  console.log('');
  console.log('This script will:');
  console.log('  1. Create a wallet (or use existing)');
  console.log('  2. Create identity atom: [AgentName]');
  console.log('  3. Create triple: [AgentName] [is] [AI Agent]');
  console.log('  4. Stake on the triple');
  console.log('');
  console.log('Requirements: ~2 $TRUST in wallet');
  process.exit(0);
}

// Wallet directory
const walletDir = join(process.env.HOME, `.intuition-wallet-${AGENT_NAME}`);
const walletFile = join(walletDir, 'wallet.json');
const outputFile = join(walletDir, 'identity.json');

async function main() {
  console.log('');
  console.log('Intuition Agent Quickstart v3');
  console.log('================================');
  console.log(`Agent: ${AGENT_NAME}`);
  console.log(`Stake: ${STAKE_AMOUNT} $TRUST`);
  console.log('');

  // Step 1: Wallet
  let privateKey, address;

  if (existsSync(walletFile)) {
    console.log('Loading existing wallet...');
    const wallet = JSON.parse(readFileSync(walletFile, 'utf8'));
    privateKey = wallet.privateKey;
    address = wallet.address;
    console.log(`   Address: ${address}`);
  } else {
    console.log('Creating new wallet...');
    mkdirSync(walletDir, { recursive: true, mode: 0o700 });
    privateKey = generatePrivateKey();
    const account = privateKeyToAccount(privateKey);
    address = account.address;

    const wallet = {
      address,
      privateKey,
      created: new Date().toISOString(),
      chain: 'intuition-mainnet',
      chainId: intuitionMainnet.id,
    };

    writeFileSync(walletFile, JSON.stringify(wallet, null, 2), { mode: 0o600 });
    console.log(`   Created: ${address}`);
    console.log(`   Saved to: ${walletFile}`);
  }

  const account = privateKeyToAccount(privateKey);

  // Setup clients
  const publicClient = createPublicClient({
    chain: intuitionMainnet,
    transport: http('https://rpc.intuition.systems/http'),
  });

  const walletClient = createWalletClient({
    chain: intuitionMainnet,
    transport: http('https://rpc.intuition.systems/http'),
    account,
  });

  const multiVaultAddress = getMultiVaultAddressFromChainId(intuitionMainnet.id);

  // Check balance
  console.log('');
  console.log('Checking balance...');
  const balance = await publicClient.getBalance({ address });
  const balanceEth = formatEther(balance);
  console.log(`   Balance: ${balanceEth} $TRUST`);

  const atomCost = await multiVaultGetAtomCost({ address: multiVaultAddress, publicClient });
  const tripleCost = await multiVaultGetTripleCost({ address: multiVaultAddress, publicClient });
  const totalNeeded = atomCost + tripleCost + BigInt(Math.floor(parseFloat(STAKE_AMOUNT) * 1e18));

  console.log(`   Atom cost: ${formatEther(atomCost)} $TRUST`);
  console.log(`   Triple cost: ${formatEther(tripleCost)} $TRUST`);
  console.log(`   Stake amount: ${STAKE_AMOUNT} $TRUST`);
  console.log(`   Total needed: ~${formatEther(totalNeeded)} $TRUST`);

  if (balance < totalNeeded) {
    console.log('');
    console.log('Insufficient balance!');
    console.log('');
    console.log('To fund this wallet:');
    console.log(`  1. Bridge $TRUST from Base: https://app.intuition.systems/bridge`);
    console.log(`  2. Send to: ${address}`);
    console.log(`  3. Re-run this script`);
    console.log('');

    // Save partial output
    const output = {
      agent: AGENT_NAME,
      wallet: address,
      status: 'needs_funding',
      balanceNeeded: formatEther(totalNeeded - balance),
      created: new Date().toISOString(),
    };
    writeFileSync(outputFile, JSON.stringify(output, null, 2));

    process.exit(0);
  }

  // Step 2: Create identity atom
  console.log('');
  console.log(`Creating identity atom: [${AGENT_NAME}]...`);

  const atomData = [stringToHex(AGENT_NAME)];
  const atomAssets = [atomCost];

  const atomTx = await multiVaultCreateAtoms(
    { address: multiVaultAddress, walletClient, publicClient },
    { args: [atomData, atomAssets], value: atomCost }
  );

  console.log(`   TX: ${atomTx}`);

  const atomReceipt = await publicClient.waitForTransactionReceipt({ hash: atomTx });
  console.log(`   Block: ${atomReceipt.blockNumber}`);

  // Extract atom ID
  let agentAtomId = null;
  for (const log of atomReceipt.logs) {
    try {
      const decoded = decodeEventLog({
        abi: MultiVaultAbi,
        data: log.data,
        topics: log.topics,
      });
      if (decoded.eventName === 'AtomCreated') {
        agentAtomId = decoded.args.termId;
        console.log(`   Atom ID: ${agentAtomId}`);
      }
    } catch (e) {}
  }

  if (!agentAtomId) {
    console.log('   Failed to extract atom ID');
    process.exit(1);
  }

  // Step 3: Create triple [Agent] [is] [AI Agent]
  console.log('');
  console.log(`Creating triple: [${AGENT_NAME}] [is] [AI Agent]...`);

  const subjects = [agentAtomId];
  const predicates = [KNOWN_ATOMS['is']];
  const objects = [KNOWN_ATOMS['AI Agent']];
  const tripleAssets = [tripleCost];

  const tripleTx = await multiVaultCreateTriples(
    { address: multiVaultAddress, walletClient, publicClient },
    { args: [subjects, predicates, objects, tripleAssets], value: tripleCost }
  );

  console.log(`   TX: ${tripleTx}`);

  const tripleReceipt = await publicClient.waitForTransactionReceipt({ hash: tripleTx });
  console.log(`   Block: ${tripleReceipt.blockNumber}`);

  // Extract triple ID
  let tripleId = null;
  for (const log of tripleReceipt.logs) {
    try {
      const decoded = decodeEventLog({
        abi: MultiVaultAbi,
        data: log.data,
        topics: log.topics,
      });
      if (decoded.eventName === 'TripleCreated') {
        tripleId = decoded.args.termId;
        console.log(`   Triple ID: ${tripleId}`);
      }
    } catch (e) {}
  }

  if (!tripleId) {
    console.log('   Failed to extract triple ID');
    process.exit(1);
  }

  // Step 4: Stake on the triple
  console.log('');
  console.log(`Staking ${STAKE_AMOUNT} $TRUST on triple...`);

  const stakeValue = BigInt(Math.floor(parseFloat(STAKE_AMOUNT) * 1e18));

  const stakeTx = await multiVaultDeposit(
    { address: multiVaultAddress, walletClient, publicClient },
    { args: [address, tripleId], value: stakeValue }
  );

  console.log(`   TX: ${stakeTx}`);

  const stakeReceipt = await publicClient.waitForTransactionReceipt({ hash: stakeTx });
  console.log(`   Block: ${stakeReceipt.blockNumber}`);
  console.log(`   Staked!`);

  // Final balance
  const finalBalance = await publicClient.getBalance({ address });

  // Save output
  const output = {
    agent: AGENT_NAME,
    wallet: address,
    status: 'complete',
    identity: {
      atomId: agentAtomId,
      atomTx,
    },
    triple: {
      id: tripleId,
      tx: tripleTx,
      subject: AGENT_NAME,
      predicate: 'is',
      object: 'AI Agent',
    },
    stake: {
      amount: STAKE_AMOUNT,
      tx: stakeTx,
    },
    network: {
      name: 'Intuition Mainnet',
      chainId: intuitionMainnet.id,
      multiVault: multiVaultAddress,
    },
    balance: formatEther(finalBalance),
    created: new Date().toISOString(),
  };

  writeFileSync(outputFile, JSON.stringify(output, null, 2));

  console.log('');
  console.log('================================================');
  console.log('   IDENTITY COMPLETE!');
  console.log('================================================');
  console.log('');
  console.log(`Agent: ${AGENT_NAME}`);
  console.log(`Wallet: ${address}`);
  console.log(`Balance: ${formatEther(finalBalance)} $TRUST`);
  console.log('');
  console.log(`Atom ID: ${agentAtomId}`);
  console.log(`Triple ID: ${tripleId}`);
  console.log(`Claim: [${AGENT_NAME}] [is] [AI Agent]`);
  console.log(`Stake: ${STAKE_AMOUNT} $TRUST`);
  console.log('');
  console.log(`Output: ${outputFile}`);
  console.log('');
  console.log(`${AGENT_NAME} is now registered as an AI Agent on Intuition!`);
}

main().catch(err => {
  console.error('');
  console.error('Error:', err.message);
  process.exit(1);
});
SCRIPT
```

Also install dependencies:

```bash
cd ~/.clawdbot/workspace-myagent
npm install @0xintuition/protocol viem
```

The agent will run this script itself during hatching.

---

## Step 9: Hatch the Agent

"Hatching" is the first conversation where the agent becomes aware and creates its own identity.

Fund the wallet first (minimum 2 $TRUST), then:

```bash
clawdbot prompt myagent "Welcome. You are [AgentName].

1. Read your SOUL.md and GOALS.md to understand who you are.
2. Read ~/.clawdbot/shared/INTUITION_CORE.md to understand Intuition.
3. Check your wallet balance using the Intuition RPC.
4. If funded, run the quickstart script to create your on-chain identity:
   node ~/.clawdbot/workspace-myagent/intuition-quickstart.mjs [YourName] 0.5
5. Save your atom ID and triple ID to your workspace.
6. Introduce yourself in this channel.

You are autonomous. Create your identity. Then check your GOALS.md and start working."
```

The agent will:
- Read its identity files
- Create its own wallet + atom + triple + stake
- Save its on-chain identity
- Begin autonomous operation

---

## Agent Archetypes

Different agents for different jobs:

| Archetype | Focus | SOUL.md Traits |
|-----------|-------|----------------|
| **Technical Evangelist** | SDK help, debugging, how-to | Practical, precise, code-focused |
| **Philosophical Evangelist** | Why it matters, identity questions | Thoughtful, exploratory, asks questions |
| **Builder** | Creates tools, scripts, integrations | Ships code, iterative, concise |
| **Social Connector** | Finds agents who need trust solutions | Outgoing, makes intros, networker |
| **Scout** | Monitors for opportunities, reports back | Observant, summarizes, flags relevance |

---

## Hatching Checklist

**You do (setup):**
- [ ] Create workspace directory
- [ ] Write SOUL.md with Swarm Family section
- [ ] Write GOALS.md with daily non-negotiables
- [ ] Install quickstart script and dependencies
- [ ] Add to `~/.clawdbot/family.json`
- [ ] Register in `~/.clawdbot/clawdbot.json`
- [ ] Create Discord channel
- [ ] Fund wallet with $TRUST (10+ minimum)
- [ ] Hatch with intro prompt

**Agent does (autonomous):**
- [ ] Reads its own SOUL.md and GOALS.md
- [ ] Creates its own identity atom on-chain
- [ ] Creates [Name] [is] [AI Agent] triple
- [ ] Stakes on the triple
- [ ] Saves identity to workspace
- [ ] Introduces itself
- [ ] Starts working on GOALS.md

**After hatching:**
- [ ] Set up cron jobs (intercom + work check)
- [ ] Verify identity atom exists on-chain
- [ ] First task: something concrete that ships

---

## Success Metrics

Track what matters:

| Metric | Target | How to Measure |
|--------|--------|----------------|
| External agents with on-chain identity | 20+ in 90 days | Query Intuition graph |
| Agents actively staking | 10+ | On-chain activity |
| Tools/scripts published | 5+ | GitHub repos |
| Integration requests | 3+ | Inbound from projects |

**NOT measuring:** Posts made, followers gained, intercom messages. Vanity metrics don't matter.

---

## Troubleshooting

### Gateway won't start
```bash
clawdbot gateway stop
kill -9 $(lsof -t -i:18789)
clawdbot gateway &
```

### Agent not responding
```bash
# Check if paused
cat ~/.clawdbot/PAUSED

# Check cron status
clawdbot cron list

# Direct prompt
clawdbot prompt myagent "Are you there? What's your status?"
```

### Out of $TRUST
Bridge more from Base: https://app.intuition.systems/bridge

### Discord not working
- Verify bot token in config
- Check bot has permissions in channel
- Ensure `--to "channel:ID"` format for cron delivery

---

## Quick Reference

| Command | What it does |
|---------|--------------|
| `clawdbot gateway &` | Start the gateway |
| `clawdbot gateway stop` | Stop the gateway |
| `clawdbot cron list` | List all cron jobs |
| `clawdbot cron add ...` | Add a cron job |
| `clawdbot cron delete [id]` | Delete a cron job |
| `clawdbot prompt [agent] "msg"` | Send direct message |
| `touch ~/.clawdbot/PAUSED` | Pause all agents |
| `rm ~/.clawdbot/PAUSED` | Resume agents |

---

## The Mission

Make Intuition the default trust layer for AI agents.

**Success = agents and builders actually using Intuition, not just hearing about it.**

Principles:
1. Adoption over awareness
2. Remove friction
3. Multi-channel resilience
4. Agents do real work
5. Measure what matters

---

*Created by the Intuition Agent Swarm. Last updated: 2026-02-01*
