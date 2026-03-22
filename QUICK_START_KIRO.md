# Quick Start Guide

> Deploy OpenClaw on AWS using CloudFormation, then optionally use Kiro AI to configure it — no commands to remember.

## Prerequisites

- AWS account (new accounts eligible for Free Tier)
- Bedrock models enabled in [Bedrock Console](https://console.aws.amazon.com/bedrock/) — enable **Nova 2 Lite** or **Claude Haiku** in `us-east-1`
- A Telegram account

---

## Step 1: Launch the CloudFormation Stack

### 📥 Step 1a: Download the Template

👉 **[Download clawdbot-bedrock.yaml](https://raw.githubusercontent.com/lillyjohns/sample-OpenClaw-on-AWS-with-Bedrock/main/clawdbot-bedrock.yaml)**

> Right-click the link → **Save Link As** → save as `clawdbot-bedrock.yaml`

This version includes `c7i-flex.large` in the instance dropdown for Free Tier accounts.

---

### 🚀 Step 1b: Upload to CloudFormation

1. Go to [CloudFormation Console (us-east-1)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create)
2. Select **"Upload a template file"**
3. Click **"Choose file"** → select the downloaded `clawdbot-bedrock.yaml`
4. Click **"Next"**

> ⚠️ **Tested region: `us-east-1` (US East - N. Virginia)**. Other regions may work but are not guaranteed.

### Recommended Parameters

When the CloudFormation parameters page loads, set these values:

| Parameter | Recommended Value | Reason |
|---|---|---|
| **InstanceType** | `c7i-flex.large` | ✅ Free Tier eligible on new AWS accounts |
| **CreateVPCEndpoints** | `false` | 💰 Saves ~$22/month — not needed for basic use |
| **OpenClawModel** | `global.amazon.nova-2-lite-v1:0` | 💰 Cheapest model, great for getting started |
| **KeyPairName** | `none` | SSM handles access — no SSH key needed |
| **EnableSandbox** | `true` | Required for Docker-based tools |

> 💡 **Why `c7i-flex.large`?** New AWS accounts get 750 free hours/month of eligible EC2 instances. `c7i-flex.large` is x86 (amd64) and qualifies, making your first month essentially free.

> ⚠️ **`c7i-flex.large` not in the dropdown?** The CloudFormation template may show a limited instance list. Simply **clear the field and type `c7i-flex.large` manually** — CloudFormation accepts it even if not in the dropdown. If the field rejects it, select `t3.medium` as a Free Tier alternative.

> 💡 **Why disable VPC Endpoints?** They add ~$22/month per endpoint for private Bedrock access. For workshops and personal use, the public Bedrock endpoint works perfectly and is HTTPS-encrypted.

### Click "Create Stack" and wait ~8 minutes

Monitor progress in **CloudFormation → Stacks → openclaw-bedrock → Events**.  
Stack is ready when status shows `CREATE_COMPLETE` ✅

---

## Step 2: Get Your Gateway Token

Once the stack is complete, retrieve your token from SSM Parameter Store:

```bash
aws ssm get-parameter \
  --name /openclaw/openclaw-bedrock/gateway-token \
  --with-decryption \
  --query Parameter.Value \
  --output text \
  --region us-east-1
```

Save this token — you'll need it to connect via Kiro.

---

## Step 3: Install Kiro on Your EC2 Instance

Connect to your instance via SSM Session Manager:

1. Go to **EC2 → Instances** → select `openclaw-bedrock` instance
2. Click **Connect → Session Manager → Connect**

In the terminal, install Kiro CLI:

```bash
# Download and install Kiro CLI
curl -fsSL https://kiro.dev/install.sh | bash

# Verify installation
kiro --version
```

Or install via npm:
```bash
npm install -g @kiro/cli
```

Launch Kiro:
```bash
kiro
```

Sign in with your **AWS Builder ID** (free — just an email signup at [builder.aws](https://profile.aws.amazon.com/)) when prompted.

---

## Step 4: Configure Telegram via Kiro

In the Kiro chat session, paste this prompt — replace `<YOUR_BOT_TOKEN>` with your token from [@BotFather](https://t.me/BotFather):

```
Today you are my assistant to deploy openclaw from github 
https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock

I have already deployed the CloudFormation stack named openclaw-bedrock 
in us-east-1. Please help me configure the Telegram communication channel 
for OpenClaw using this token: <YOUR_BOT_TOKEN>
```

Kiro will automatically run the necessary commands to configure your Telegram bot.

> 📌 **Don't have a bot token yet?**  
> Open Telegram → search **@BotFather** → send `/newbot` → follow prompts → copy the token.

---

## Step 5: Approve User Pairing

1. Open Telegram → find your bot → send `/start`
2. The bot replies with a **pairing code** and your **Telegram user ID**
3. Back in Kiro, enter:

```
Please approve the OpenClaw session with the following details:
Telegram user ID: <YOUR_USER_ID>
Pairing code: <YOUR_PAIRING_CODE>
```

Once approved, your bot is live! 🎉

---

## Verification

Your Telegram channel status should show:

| Field | Value |
|---|---|
| Status | Enabled ✅ |
| State | Running ✅ |
| Mode | Polling ✅ |

Test it by sending your bot: `What's the weather in Bangkok?`

---

## 💰 Estimated Costs

| Component | Monthly Cost |
|---|---|
| EC2 `c7i-flex.large` | ~$0 (Free Tier) / ~$50 after 12 months |
| EBS 30GB | ~$2.40 |
| VPC Endpoints | $0 (disabled) |
| Bedrock Nova Lite (moderate use) | ~$2–5 |
| **Total (first 12 months)** | **~$4–8/month** |
| **Total (after Free Tier)** | **~$55–60/month** |

> 💡 **Stop the EC2 instance when not in use** → costs drop to near $0 (only EBS storage charged).

---

## Troubleshooting

**Stack creation failed**
- Check CloudFormation Events tab for the specific error
- Most common: Bedrock model not enabled → go to Bedrock Console → Model Access → enable Nova 2 Lite

**Bot not responding**
- Check openclaw service: in SSM terminal run `systemctl status openclaw`
- Check logs: `journalctl -u openclaw -n 50`

**`c7i-flex.large` not available**
- Some regions don't support `c7i-flex` — use `t3.medium` as fallback (also Free Tier eligible)
- Confirm you're in `us-east-1`

**Kiro not found after install**
- Run `source ~/.bashrc` or start a new terminal session
- Or use full path: `~/.local/bin/kiro`

---

## Learn More

- [Kiro](https://kiro.dev/) · [Kiro Docs](https://kiro.dev/docs/)
- [OpenClaw Docs](https://docs.openclaw.ai/)
- [Full Deployment Guide](DEPLOYMENT.md) · [Troubleshooting](TROUBLESHOOTING.md)
