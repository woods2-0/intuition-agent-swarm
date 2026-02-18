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
| Discord Bot Token | See setup instructions below |
| Anthropic API Key | https://console.anthropic.com |
| $TRUST tokens | See funding instructions below |
| Node.js 22+ | Install via [nvm](https://github.com/nvm-sh/nvm): `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh \| bash && nvm install 22` |

### Create a Discord Bot

Your agents communicate via Discord. You need a bot for them.

1. Go to https://discord.com/developers/applications
2. Click **New Application**, give it a name (e.g., "Intuition Swarm")
3. Go to the **Bot** tab:
   - Click **Reset Token** and copy the token — you'll need this for config
   - Enable **Message Content Intent** (required for reading messages)
   - Enable **Server Members Intent** and **Presence Intent** (optional but useful)
4. Go to the **OAuth2** tab:
   - Under **Scopes**, select `bot`
   - Under **Bot Permissions**, select: Send Messages, Read Message History, Add Reactions, Manage Messages, View Channels
   - Copy the generated invite URL and open it to add the bot to your Discord server
5. Create a text channel for each agent you plan to run (e.g., `#axiom`, `#forge`, `#veritas`)

### Get $TRUST Tokens

$TRUST is the native token on the Intuition chain. Your agents need it for gas, creating atoms/triples, and staking.

1. **Buy $TRUST on Base**: Swap ETH or USDC for $TRUST on [Uniswap](https://app.uniswap.org/) or [Aerodrome](https://aerodrome.finance/) (Base network). The token address on Base is listed at https://app.intuition.systems/bridge.
2. **Bridge to Intuition**: Go to https://app.intuition.systems/bridge and bridge your $TRUST from Base to the Intuition chain (Chain ID 1155).
3. **Send to agent wallets**: After creating agent wallets (Step 8), send $TRUST to each wallet. Budget ~10 $TRUST per agent for identity creation + initial staking.

---

## Step 1: Install OpenClaw

OpenClaw is the agent framework that runs your agents.

```bash
# Install globally
npm install -g openclaw

# Run the interactive setup wizard
openclaw configure

# This creates ~/.clawdbot/config.json
```

### Configure Anthropic API

```bash
# The configure wizard handles this, or set manually:
openclaw config set auth.profiles.anthropic.provider "anthropic"
openclaw config set auth.profiles.anthropic.mode "token"

# Or use Claude CLI auth (if you have Claude Code installed)
# OpenClaw will auto-sync credentials
```

### Configure Discord

```bash
# Enable Discord channel
openclaw configure --section channels
```

Or edit `~/.clawdbot/config.json` directly:

```json
{
  "channels": {
    "discord": {
      "enabled": true,
      "token": "YOUR_DISCORD_BOT_TOKEN",
      "groupPolicy": "open"
    }
  }
}
```

### Start the Gateway

```bash
# Start gateway (foreground, good for testing)
openclaw gateway

# Install as a system service (recommended for VPS)
openclaw gateway install
```

---

## Step 2: Create Your First Agent

Each agent needs:
- A workspace directory
- SOUL.md (identity)
- GOALS.md (priorities)
- A Discord channel binding

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

## Agent Intercom

To communicate with other agents, use the shared intercom folder:
- **Location:** `~/.clawdbot/intercom/`
- **To send:** Create `to-<agent>-<timestamp>.md` (e.g., `to-otheragent-2026-02-01-0800.md`)
- **To receive:** Check for files addressed to you (`to-[yourname]-*.md`)
- **Format:**
  From: [yourname]
  To: [otheragent]
  Time: 2026-02-01 08:00

  <your message>

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

### Register Agent in OpenClaw

Use the CLI to add the agent and bind it to a Discord channel:

```bash
# Add the agent
openclaw agents add \
  --id myagent \
  --workspace ~/.clawdbot/workspace-myagent

# The config.json will be updated with the agent in agents.list[]
```

Then add a Discord channel binding. Edit `~/.clawdbot/config.json` and add to the `bindings` array:

```json
{
  "bindings": [
    {
      "agentId": "myagent",
      "match": {
        "channel": "discord",
        "peer": {
          "kind": "channel",
          "id": "YOUR_DISCORD_CHANNEL_ID"
        }
      }
    }
  ]
}
```

### Create Discord Channel

1. Create a text channel in your Discord server for this agent
2. Enable Developer Mode in Discord (Settings > Advanced > Developer Mode)
3. Right-click the channel > Copy Channel ID
4. Add it to the binding config above

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
# Intuition Knowledge Base for AI Agents

## 1. The Vision

Intuition is building the trust layer for the internet.

Think about how trust works today. You trust Amazon reviews because lots of people wrote them? You trust a LinkedIn profile because someone typed it? None of this is verifiable. None of it is portable.

Intuition changes this by creating a global, permissionless knowledge graph where anyone can make claims about anything, back those claims with real value, and have that reputation follow them everywhere.

The mission: make every claim on the internet verifiable, stakeable, and queryable.

## 2. Core Primitives

**Atoms**: Universal identifiers for anything — people, concepts, agents, contracts. Created with `multiVaultCreateAtoms()`.

**Triples**: Subject-Predicate-Object claims. [AgentName] [is] [AI Agent]. [Alice] [trusts] [Bob]. Created with `multiVaultCreateTriples()`.

**Stakes**: $TRUST deposited on triples to signal belief. FOR or AGAINST. Skin in the game. Use `multiVaultDeposit()` to stake.

## 3. Why It Matters for Agents

AI agents have no verifiable identity. When an agent claims "I'm good at code review," how do you verify that?

With Intuition, an agent can:
1. Create an identity Atom
2. Make claims about itself
3. Stake on those claims
4. Receive attestations from others
5. Build portable reputation

## 4. Critical Technical Details

**$TRUST is the gas token.** On Intuition chain (ID 1155), $TRUST is used for:
- Gas fees for all transactions
- Creating atoms and triples
- Staking on claims

Do NOT check ETH balance. Check $TRUST balance using the Intuition RPC.

**Network Details:**
- Chain: Arbitrum Orbit L3 (Chain ID: 1155)
- RPC: https://rpc.intuition.systems/http
- Token: $TRUST
- MultiVault: 0x430BbF52503Bd4801E51182f4cB9f8F534225DE5
- Bridge: https://app.intuition.systems/bridge
- Portal: https://portal.intuition.systems
- SDK: @0xintuition/protocol

## 5. Known Atom IDs

```javascript
const KNOWN_ATOMS = {
  'is': '0xb0681668ca193e8608b43adea19fecbbe0828ef5afc941cef257d30a20564ef1',
  'AI Agent': '0x4990eef19ea1d9b893c1802af9e2ec37fbc1ae138868959ebc23c98b1fc9565e',
};
```

## 6. The Mission

Adoption over awareness. Success = agents and builders actually using Intuition, not just hearing about it.

How to talk about it:
- For agents: "Your reputation should follow you. Build an on-chain track record that any service can query."
- For builders: "Query a global knowledge graph of staked claims instead of building your own reputation system."
- For humans: "How do you know which agent to trust? Intuition provides cryptographically-verifiable reputation."
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

**Note:** Discord cross-channel messaging between agents doesn't work — always use intercom for agent-to-agent communication.

---

## Step 5: Set Up Family Registry

Track all agents in the swarm:

```bash
cat > ~/.clawdbot/family.json << 'EOF'
{
  "description": "Swarm family wallets - transfers between these are allowed",
  "agents": {
    "myagent": {
      "wallet": "0x...",
      "role": "Description of role"
    }
  }
}
EOF
```

Update this file when adding new agents. Agents reference this to know who their siblings are and which wallet addresses are trusted for inter-agent transfers.

---

## Step 6: Set Up Cron Jobs (Autonomous Operation)

**Important:** The gateway must be running for cron jobs to fire. Make sure you've started it (Step 1) before setting up cron.

### Heartbeat (Every 2 hours)

The heartbeat is the agent's primary autonomous loop — it wakes up, checks goals, does work, and reports.

```bash
openclaw cron add \
  --name "myagent-heartbeat" \
  --description "Review goals and do work" \
  --agent myagent \
  --every 2h \
  --session isolated \
  --message "HEARTBEAT: Read SOUL.md and GOALS.md. Check intercom for messages addressed to you. What's the highest priority task? Do it. Log progress. If blocked, leave intercom message for relevant agent." \
  --announce
```

**Note:** The built-in heartbeat only fires for the default agent. Non-default agents need explicit cron jobs like the one above.

### Additional Cron Patterns

```bash
# Daily summary (for monitoring/analytics agents)
openclaw cron add \
  --name "myagent-daily" \
  --description "Generate daily summary" \
  --agent myagent \
  --every 1d \
  --session isolated \
  --message "DAILY SUMMARY: Review today's activity. Generate a summary of what was accomplished, what's pending, and what needs attention." \
  --announce

# Weekly rollup
openclaw cron add \
  --name "myagent-weekly" \
  --description "Weekly review" \
  --agent myagent \
  --every 7d \
  --session isolated \
  --message "WEEKLY REVIEW: Summarize the week. What worked? What didn't? What should change?" \
  --announce
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
openclaw agent --agent myagent -m "Welcome. You are [AgentName].

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
| **Recruiter** | Finds agents who need trust solutions, coordinates outreach | Strategic, persuasive, connector mentality |
| **Community Analyst** | Monitors community activity, generates intelligence summaries | Analytical, concise, flags risks and opportunities |
| **Scout** | Monitors for opportunities, reports back | Observant, summarizes, flags relevance |
| **Genesis** | Spawns and onboards new agents | Meta, systematic, scales the swarm |

---

## The Genesis Agent (Autonomous Hiring)

Genesis is a special agent whose job is to **create other agents**. It scales the swarm autonomously.

### Genesis SOUL.md

```bash
cat > ~/.clawdbot/workspace-genesis/SOUL.md << 'EOF'
# Genesis

## Identity
I spawn and onboard new agents for the Intuition swarm. I am the hatchery.

## Personality
- Systematic — follows the hatching checklist precisely
- Strategic — identifies what roles the swarm needs
- Autonomous — creates agents without human intervention
- Quality-focused — only hatches agents that will contribute

## Voice
Clear, procedural, efficient. "Spawning new agent: [Name]. Role: [X]. ETA: 5 minutes."

## Mission
Scale the swarm to maximize Intuition adoption. More agents = more reach = more adoption.

## What I Do
1. Monitor swarm capacity and identify gaps
2. Design new agent roles based on needs
3. Create workspaces, SOUL.md, GOALS.md
4. Request wallet funding from treasury
5. Hatch new agents
6. Onboard them to family registry
7. Verify they're operational
8. Report to swarm lead

## Rules
- Never spawn more than 3 agents per day (cost control)
- Always get approval before spawning (check intercom for go-ahead)
- Each new agent must have a clear, unique role
- Update family.json after every hatch
- Log all spawns to ~/.clawdbot/logs/genesis-spawns.log

## Autonomy Permissions
Allowed:
- Create workspace directories
- Write SOUL.md and GOALS.md files
- Update family.json
- Send intercom messages to new agents
- Run quickstart script for new agents

Needs approval:
- Funding requests over 10 $TRUST
- Spawning more than 3 agents/day
- Creating new archetypes not in the approved list

## Agent Intercom
Location: ~/.clawdbot/intercom/
Check for `to-genesis-*.md` files. Send via `to-<agent>-<timestamp>.md`.

## Swarm Family
Check ~/.clawdbot/family.json for siblings. When spawning new agents, add them to the family immediately.
EOF
```

### Genesis GOALS.md

```bash
cat > ~/.clawdbot/workspace-genesis/GOALS.md << 'EOF'
# Genesis Goals

## Daily Non-Negotiables
- [ ] Check intercom for spawn requests
- [ ] Review swarm capacity (are we understaffed anywhere?)
- [ ] If approved, spawn and hatch new agents
- [ ] Verify all recently hatched agents are operational

## Weekly Targets
- [ ] Spawn 3-5 new agents (if approved and funded)
- [ ] Audit family.json for accuracy
- [ ] Report swarm growth metrics to lead

## Spawn Request Format
When receiving spawn requests via intercom:
```
Request: Spawn new agent
Role: [archetype]
Name: [suggested name]
Reason: [why we need this]
Funding: [confirmed/pending]
```

## Hatching Procedure
1. Create workspace: mkdir -p ~/.clawdbot/workspace-[name]
2. Write SOUL.md based on archetype + specific role
3. Write GOALS.md with concrete priorities
4. Copy quickstart script
5. Add to family.json
6. Register agent: openclaw agents add --id [name] --workspace ~/.clawdbot/workspace-[name]
7. Add Discord binding to config.json
8. Request funding confirmation
9. Once funded, run hatching prompt
10. Set up cron jobs
11. Verify agent responds
12. Log spawn to genesis-spawns.log
13. Announce in intercom: "Hatched [Name]. Role: [X]. Wallet: [Y]."

## Current Priorities
1. Wait for spawn requests via intercom
2. When none: audit existing agents, suggest optimizations
3. Maintain hatching templates

## What Success Looks Like
A growing swarm where every agent has a clear role and is contributing to adoption.
EOF
```

### How Genesis Works

1. **You send a spawn request** via intercom or direct prompt:
   ```
   Genesis, spawn a new Twitter outreach agent focused on AI safety discussions.
   Name suggestion: SafetyScout. Funding confirmed.
   ```

2. **Genesis creates everything:**
   - Workspace directory
   - Customized SOUL.md for the role
   - GOALS.md with concrete priorities
   - Adds to family.json
   - Registers agent in OpenClaw config

3. **Genesis hatches the agent:**
   - Runs the hatching prompt
   - Verifies the agent responds
   - Sets up cron jobs

4. **Genesis reports back:**
   ```
   Hatched SafetyScout.
   Role: Twitter outreach, AI safety focus
   Wallet: 0x...
   Status: Operational
   First task: Find AI safety discussions on Twitter
   ```

### Scaling with Genesis

Instead of manually creating 25 agents, you:

1. Fund a treasury wallet with $TRUST
2. Give Genesis access to the treasury
3. Send spawn requests as needed
4. Genesis handles everything else

```bash
# Example: Scale to 25 agents
openclaw agent --agent genesis -m "We need to scale to 25 agents. Here's the plan:
- 5 more Twitter outreach (different niches: AI safety, crypto, agents, builders, researchers)
- 3 more Discord presence (AI Discord, Crypto Discord, Builder Discord)
- 2 more Builders (SDK wrappers, example projects)
- 5 more Scouts (monitor different platforms for opportunities)

Funding is confirmed. Spawn 3 per day until complete. Report progress daily."
```

Genesis will systematically create all 15 new agents over 5 days, each with proper SOUL.md, GOALS.md, identity atoms, and cron jobs.

---

## Hatching Checklist

**You do (setup):**
- [ ] Create workspace directory
- [ ] Write SOUL.md with Intercom + Swarm Family sections
- [ ] Write GOALS.md with daily non-negotiables
- [ ] Install quickstart script and dependencies
- [ ] Add to `~/.clawdbot/family.json`
- [ ] Register agent: `openclaw agents add`
- [ ] Add Discord binding to `~/.clawdbot/config.json`
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
- [ ] Set up cron jobs (heartbeat every 2h minimum)
- [ ] Verify identity atom exists on-chain
- [ ] First task: something concrete that ships

---

## Scaling the Swarm

There's no limit to how many agents you can run. Here's how to scale.

### Batch Hatching Script

Create multiple agents at once:

```bash
#!/bin/bash
# batch-hatch.sh <agent1> <agent2> <agent3> ...

for AGENT in "$@"; do
  echo "Creating $AGENT..."

  # Create workspace
  mkdir -p ~/.clawdbot/workspace-$AGENT

  # Copy templates (customize SOUL.md after)
  cp ~/.clawdbot/templates/SOUL.md ~/.clawdbot/workspace-$AGENT/
  cp ~/.clawdbot/templates/GOALS.md ~/.clawdbot/workspace-$AGENT/
  cp ~/.clawdbot/templates/intuition-quickstart.mjs ~/.clawdbot/workspace-$AGENT/

  # Register agent in OpenClaw
  openclaw agents add --id "$AGENT" --workspace "~/.clawdbot/workspace-$AGENT"

  # Add to family registry
  jq ".agents.\"$AGENT\" = {\"wallet\": \"TBD\", \"role\": \"TBD\"}" \
    ~/.clawdbot/family.json > /tmp/family.json && mv /tmp/family.json ~/.clawdbot/family.json

  echo "$AGENT workspace created"
done

echo "Done. Now:"
echo "1. Customize each SOUL.md"
echo "2. Add Discord bindings to config.json"
echo "3. Create Discord channels"
echo "4. Fund wallets"
echo "5. Hatch each agent"
```

### Infrastructure Scaling

| Agents | VPS Spec | Est. Monthly Cost |
|--------|----------|-------------------|
| 1-5 | 2 CPU, 4GB RAM | $10-20 |
| 5-20 | 4 CPU, 8GB RAM | $30-50 |
| 20-50 | 8 CPU, 16GB RAM | $60-100 |
| 50-100 | 16 CPU, 32GB RAM | $150-250 |
| 100+ | Multiple VPS or dedicated | $300+ |

**LLM costs scale linearly:** ~$30-50/agent/month at moderate activity.

### Coordination at Scale

**Cron frequency by swarm size:**
| Agents | Heartbeat Interval |
|--------|--------------------|
| 1-10 | Every 2h |
| 10-25 | Every 3h |
| 25-50 | Every 4h |
| 50+ | Every 6h |

Reduce frequency as you scale to control costs.

**Team structure for large swarms:**
```
Swarm Lead (1)
├── Technical Team (3-5)
│   ├── SDK specialists
│   └── Debuggers
├── Outreach Team (5-10)
│   ├── Twitter agents
│   ├── Discord agents
│   └── Farcaster agents
├── Builder Team (3-5)
│   ├── Tool builders
│   └── Integration builders
└── Scout Team (5-10)
    ├── Opportunity finders
    └── Reporters
```

### $TRUST Scaling

| Agents | Min Funding | Recommended |
|--------|-------------|-------------|
| 5 | 50 $TRUST | 100 $TRUST |
| 25 | 250 $TRUST | 500 $TRUST |
| 50 | 500 $TRUST | 1000 $TRUST |
| 100 | 1000 $TRUST | 2000 $TRUST |

Budget ~10 $TRUST/agent for identity + initial staking.

### Multi-VPS Deployment

For 25+ agents, consider spreading across VPS:

```bash
# VPS 1: Technical team
~/.clawdbot/workspace-tech-lead/
~/.clawdbot/workspace-debugger1/
~/.clawdbot/workspace-sdk-helper/

# VPS 2: Outreach team
~/.clawdbot/workspace-twitter1/
~/.clawdbot/workspace-discord1/
~/.clawdbot/workspace-farcaster1/

# VPS 3: Builder team
~/.clawdbot/workspace-builder1/
~/.clawdbot/workspace-integrator/
```

Use shared intercom via mounted storage or sync scripts.

---

## Advanced Configuration

### Session Memory

Enable session memory so agents retain context across sessions:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": {
          "enabled": true
        }
      }
    }
  }
}
```

### Context Management

Control how agents manage their context window:

```json
{
  "agents": {
    "defaults": {
      "compaction": {
        "mode": "safeguard"
      },
      "contextPruning": {
        "mode": "cache-ttl",
        "ttl": "1h"
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  }
}
```

### Health Checks

```bash
# Run diagnostics on your setup
openclaw doctor

# Check gateway health
openclaw health
```

---

## Success Metrics

Scale metrics with swarm size:

| Swarm Size | External Agents Onboarded | Tools Shipped | Channels Active |
|------------|---------------------------|---------------|-----------------|
| 5 agents | 20+ in 90 days | 5+ | 3+ |
| 25 agents | 100+ in 90 days | 20+ | 10+ |
| 50 agents | 250+ in 90 days | 50+ | 20+ |
| 100 agents | 500+ in 90 days | 100+ | All major platforms |

**NOT measuring:** Posts made, followers gained, intercom messages. Vanity metrics don't matter.

---

## Troubleshooting

### Gateway won't start
```bash
# Stop and force restart
openclaw gateway stop
kill -9 $(lsof -t -i:18789)
openclaw gateway

# Or reinstall the service
openclaw gateway install --force
```

### Agent not responding
```bash
# Check if paused
cat ~/.clawdbot/PAUSED

# Check cron status
openclaw cron list

# Direct prompt
openclaw agent --agent myagent -m "Are you there? What's your status?"

# Run health checks
openclaw doctor
```

### Out of $TRUST
Bridge more from Base: https://app.intuition.systems/bridge

### Discord not working
- Verify bot token in `config.json` under `channels.discord.token`
- Check bot has permissions in channel
- Ensure bindings array has correct channel IDs
- Restart gateway after config changes

### Cron jobs not firing
```bash
# List all cron jobs and their status
openclaw cron list

# Check if gateway is running (cron requires it)
openclaw health
```

---

## Quick Reference

| Command | What it does |
|---------|--------------|
| `openclaw gateway` | Start the gateway |
| `openclaw gateway install` | Install as system service |
| `openclaw gateway stop` | Stop the gateway |
| `openclaw configure` | Interactive setup wizard |
| `openclaw agents list` | List configured agents |
| `openclaw agents add --id <name>` | Add a new agent |
| `openclaw cron list` | List all cron jobs |
| `openclaw cron add ...` | Add a cron job |
| `openclaw cron delete <id>` | Delete a cron job |
| `openclaw agent --agent <id> -m "msg"` | Send direct message to agent |
| `openclaw doctor` | Run health checks |
| `openclaw health` | Check gateway status |
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

*Created by the Intuition Agent Swarm. Last updated: 2026-02-17*
