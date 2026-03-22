# Quick Start Guide

> Deploy OpenClaw on AWS using CloudFormation, then configure it with Kiro AI in AWS CloudShell — no software to install on your computer.

## Prerequisites

- AWS account (new accounts eligible for Free Tier)
- Bedrock models enabled in [Bedrock Console](https://console.aws.amazon.com/bedrock/) — enable **Nova 2 Lite** in `us-east-1`
- A Telegram account + bot token from [@BotFather](https://t.me/BotFather)

> 📌 **Get a Telegram bot token first:** Open Telegram → search **@BotFather** → send `/newbot` → follow prompts → copy the token. Takes 2 minutes.

---

## Step 1: Launch the CloudFormation Stack

### 📥 Step 1a: Download the Template

👉 **[Download clawdbot-bedrock.yaml](https://raw.githubusercontent.com/lillyjohns/sample-OpenClaw-on-AWS-with-Bedrock/main/clawdbot-bedrock.yaml)**

> Right-click the link → **Save Link As** → save as `clawdbot-bedrock.yaml`

---

### 🚀 Step 1b: Upload to CloudFormation

1. Go to [CloudFormation Console (us-east-1)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create)
2. Select **"Upload a template file"**
3. Click **"Choose file"** → select the downloaded `clawdbot-bedrock.yaml`
4. Click **"Next"**

> ⚠️ **Tested region: `us-east-1` (US East - N. Virginia)**

### Step 1c: Set Stack Parameters

All defaults are pre-configured for Free Tier. Just set the **stack name** and click through:

| Parameter | Default | Note |
|---|---|---|
| Stack name | `openclaw-bedrock` | Keep this exact name |
| **InstanceType** | `c7i-flex.large` | ✅ Free Tier eligible |
| **CreateVPCEndpoints** | `false` | 💰 Saves ~$22/month |
| OpenClawModel | `nova-2-lite` | Cheapest model |
| KeyPairName | `none` | Not needed |

**Click "Next" → "Next" → check acknowledgement box → "Create Stack"**

### Step 1d: Wait ~8 minutes

Monitor in **CloudFormation → Stacks → openclaw-bedrock → Events**.
Stack is ready when status shows `CREATE_COMPLETE` ✅

---

## Step 2: Open AWS CloudShell

Once the stack is `CREATE_COMPLETE`:

1. In the AWS Console top bar, click the **CloudShell icon** (terminal icon `>_`)
2. Wait for the shell to initialize (~30 seconds)

> 💡 **CloudShell is free** and has AWS CLI pre-installed. No setup needed.

---

## Step 3: Login to Kiro in CloudShell

In the CloudShell terminal, run:

```bash
kiro-cli login
```

When prompted, select **"AWS Builder ID"** and follow the steps:

1. A URL will appear — open it in your browser
2. Sign in with your **AWS Builder ID** (free signup at [profile.aws.amazon.com](https://profile.aws.amazon.com/))
3. Enter the verification code shown in CloudShell
4. Return to CloudShell — you should see `Login successful`

> 💡 **Don't have an AWS Builder ID?** It's separate from your AWS account — just an email signup, completely free.

---

## Step 4: Configure Telegram via Kiro

In CloudShell, start a Kiro chat session:

```bash
kiro-cli chat
```

Paste this prompt — replace `<YOUR_BOT_TOKEN>` with your token from Step 0:

```
Today you are my assistant to deploy openclaw from github
https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock

I have already deployed the CloudFormation stack named openclaw-bedrock
in us-east-1. Please help me configure the Telegram communication channel
for OpenClaw using this token: <YOUR_BOT_TOKEN>

Important: OpenClaw is installed under the ubuntu user via NVM.
When running commands via SSM, always use:
sudo -u ubuntu bash -c 'export NVM_DIR="/home/ubuntu/.nvm" && source $NVM_DIR/nvm.sh && openclaw <command>'
or use the wrapper: sudo -u ubuntu /usr/local/bin/openclaw <command>
```

Kiro will automatically connect to your EC2 via SSM and configure the Telegram bot. Wait for it to complete.

---

## Step 5: Approve User Pairing

1. Open Telegram → find your bot → send `/start`
2. The bot replies with a **pairing code** and your **Telegram user ID**
3. Back in Kiro (CloudShell), enter:

```
Please approve the OpenClaw session with the following details:
Telegram user ID: <YOUR_USER_ID>
Pairing code: <YOUR_PAIRING_CODE>
```

Once approved — your bot is live! 🎉

---

## Verification

Send your Telegram bot: `What's the weather in Bangkok?`

It should reply within 10 seconds. ✅

You can also check channel status via Kiro:
```
Show me the openclaw channel status
```

---

## 💰 Estimated Costs

| Component | Monthly Cost |
|---|---|
| EC2 `c7i-flex.large` | ~$0 (Free Tier, first 12 months) |
| EBS 30GB | ~$2.40 |
| VPC Endpoints | $0 (disabled) |
| Bedrock Nova Lite (moderate use) | ~$2–5 |
| **Total (first 12 months)** | **~$4–8/month** |
| **Total (after Free Tier)** | **~$55–60/month** |

> 💡 **Stop EC2 when not in use** → costs drop to near $0 (only EBS storage ~$2.40/month charged).

---

## Troubleshooting

**Stack creation failed**
- Most common: Bedrock model not enabled
- Fix: [Bedrock Console](https://console.aws.amazon.com/bedrock/) → Model Access → enable Nova 2 Lite → retry stack

**`kiro-cli` not found in CloudShell**
- CloudShell may need to be refreshed: close and reopen
- Try: `which kiro-cli` to verify installation path

**Bot not responding after pairing**
- In CloudShell: `aws ssm start-session --target <INSTANCE_ID> --region us-east-1`
- Then check: `systemctl status openclaw` and `journalctl -u openclaw -n 30`

**`c7i-flex.large` not available**
- Confirm you're in `us-east-1`
- Fallback: use `t3.medium` (also Free Tier eligible)

---

## Learn More

- [Kiro](https://kiro.dev/) · [Kiro Docs](https://kiro.dev/docs/)
- [OpenClaw Docs](https://docs.openclaw.ai/)
- [Full Deployment Guide](DEPLOYMENT.md) · [Troubleshooting](TROUBLESHOOTING.md)
