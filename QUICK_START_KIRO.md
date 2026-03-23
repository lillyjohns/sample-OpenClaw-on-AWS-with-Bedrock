# Quick Start Guide

> This repository is a modified version of [aws-samples/sample-OpenClaw-on-AWS-with-Bedrock](https://github.com/aws-samples/sample-OpenClaw-on-AWS-with-Bedrock), adapted for a specific workshop event. Changes include: Free plan–friendly defaults (`c7i-flex.large`, VPC endpoints disabled) and a simplified setup flow using Kiro CLI installed directly on the EC2 instance.

## Prerequisites

- AWS account (new accounts eligible for Free Tier)
- A Telegram account + bot token from [@BotFather](https://t.me/BotFather)

> 📌 **Get a Telegram bot token first:** Open Telegram → search **@BotFather** → send `/newbot` → follow prompts → copy the token. Takes 2 minutes.

> 💡 **Tip:** Create a **fresh AWS account** for the workshop to get the full 12-month Free Tier + Bedrock credits, regardless of your existing account age.

---

## ⚠️ Set a Billing Alarm First

Before deploying anything, set a $1 billing alert so you're notified immediately if any charges appear:

1. Go to [Billing Console](https://console.aws.amazon.com/billing/home#/preferences) → enable **"Receive Billing Alerts"** → Save
2. Open **CloudShell** (top bar `>_` icon) and run:

```bash
aws budgets create-budget \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --budget '{"BudgetName":"alert-1usd","BudgetType":"COST","TimeUnit":"MONTHLY","BudgetLimit":{"Amount":"1","Unit":"USD"}}' \
  --notifications-with-subscribers '[{"Notification":{"NotificationType":"ACTUAL","ComparisonOperator":"GREATER_THAN","Threshold":1},"Subscribers":[{"SubscriptionType":"EMAIL","Address":"YOUR_EMAIL"}]}]'
```

> This sends an email the moment your charges exceed **$0.01**. Replace `YOUR_EMAIL` with your email address.

> ⚠️ **When using Kiro:** Never press `t` (trust all) — always review each action with `y` or `n`.

> ⚠️ **Delete the stack** when you're done with the workshop to avoid ongoing charges.

---

## Step 1: Launch the CloudFormation Stack

### 📥 Step 1a: Download the Template

👉 **[Download openclaw-bedrock.yaml](https://raw.githubusercontent.com/lillyjohns/sample-OpenClaw-on-AWS-with-Bedrock/main/openclaw-bedrock.yaml)**

> Right-click the link → **Save Link As** → save as `openclaw-bedrock.yaml`

---

### 🚀 Step 1b: Upload to CloudFormation

1. Go to [CloudFormation Console (us-east-1)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create)
2. Select **"Upload a template file"**
3. Click **"Choose file"** → select the downloaded `openclaw-bedrock.yaml`
4. Click **"Next"**

> ⚠️ **Tested region: `us-east-1` (US East - N. Virginia)**

### Step 1c: Set Stack Parameters

All defaults are pre-configured. Just verify and click through:

| Parameter | Default | Note |
|---|---|---|
| Stack name | `openclaw-bedrock` | Keep this exact name |
| **InstanceType** | `c7i-flex.large` | Recommended for workshop |
| **CreateVPCEndpoints** | `false` | 💰 Saves ~$22/month |
| OpenClawModel | `nova-2-lite` | Cheapest model |
| KeyPairName | `none` | Not needed |

**Click "Next" → "Next" → check acknowledgement box → "Create Stack"**

### Step 1d: Wait ~8 minutes

Monitor in **CloudFormation → Stacks → openclaw-bedrock → Events**.
Stack is ready when status shows `CREATE_COMPLETE` ✅

---

## Step 2: Connect to EC2 via Session Manager

1. Go to **EC2 → Instances** → select `openclaw-bedrock` instance
2. Click **Connect → Session Manager → Connect**
3. Switch to the ubuntu user:

```bash
sudo -u ubuntu -i
```

---

## Step 3: Install and Login to Kiro

Install Kiro CLI on the EC2 instance:

```bash
curl -fsSL https://cli.kiro.dev/install | bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
```

> ⚠️ The installer adds `kiro-cli` to `~/.local/bin` which isn't in your PATH by default. The command above adds it to `~/.bashrc` so it persists across sessions.

Then login with your **AWS Builder ID** using device flow (required on headless EC2):

```bash
kiro-cli login --use-device-flow
```

When prompted, select **"Use for Free with Builder ID"** and follow the steps:

1. A URL and a code will appear in the terminal
2. Open the URL in your **local browser** (laptop/phone)
3. Sign in with your **AWS Builder ID** (free signup at [profile.aws.amazon.com](https://profile.aws.amazon.com/))
4. Enter the verification code shown in the terminal
5. You should see `Login successful`

> 💡 **AWS Builder ID** is separate from your AWS account — just a free email signup.

---

## Step 4: Configure Telegram via Kiro

Start a Kiro chat session:

```bash
kiro-cli chat
```

Paste this prompt — replace `<YOUR_BOT_TOKEN>` with your token from BotFather (e.g. `1234567890:ABCdefGHIjklMNOpqrsTUVwxyz`):

```
I deployed the CloudFormation stack "openclaw-bedrock" in us-east-1.
Configure the Telegram channel for OpenClaw with this bot token: <YOUR_BOT_TOKEN>
OpenClaw is installed under the ubuntu user via NVM — run all openclaw commands as ubuntu.
```

Kiro will configure the Telegram bot automatically. Wait for it to complete.

---

## Step 5: Approve User Pairing

1. Open Telegram → find your bot → send `/start`
2. The bot replies with a **pairing code** and your **Telegram user ID**
3. Back in Kiro, enter:

```
Approve OpenClaw Telegram pairing. User ID: <YOUR_USER_ID>, Code: <YOUR_PAIRING_CODE>.
OpenClaw is installed under the ubuntu user via NVM. Run:

sudo -u ubuntu bash -c 'source /home/ubuntu/.nvm/nvm.sh && openclaw pairing approve --channel telegram <PAIRING_CODE> --notify'
```

Once approved — your bot is live! 🎉

---

## Verification

Send your Telegram bot: `What's the weather in Bangkok?`

It should reply within 10 seconds. ✅

---

## 💰 Estimated Costs

| Component | Monthly Cost |
|---|---|
| EC2 `c7i-flex.large` | ~$61/month |
| EBS 30GB | ~$2.40 |
| VPC Endpoints | $0 (disabled) |
| Bedrock Nova Lite (moderate use) | ~$2–5 |
| **Total (monthly)** | **~$65–70/month** |

> 💡 **Stop the EC2 instance when not in use** → costs drop to ~$2.40/month (only EBS storage charged).

---

## Troubleshooting

**Bedrock API throttling / "Too many requests" error**
- Free alternative: use [OpenRouter](https://openrouter.ai) which provides free model access ($0 limit tier)

  1. Go to [openrouter.ai](https://openrouter.ai) → Sign up (GitHub or email)
  2. Go to **Keys** → **Create Key** → set **Credit limit** to `0` (for $0 cost control) → copy your key
  3. In Kiro, paste this prompt (replace `<YOUR_API_KEY>` with your key, e.g. `sk-or-v1-xxx`):

```
Add OpenRouter API key <YOUR_API_KEY> to ~/.openclaw/openclaw.json as env.OPENROUTER_API_KEY,
set primary model to openrouter/stepfun/step-3.5-flash:free, then restart the openclaw gateway.
Run all commands as the ubuntu user.
```

> 💡 OpenRouter free tier includes models like **Step 3.5 Flash** (Stepfun) at $0 cost — great fallback when Bedrock is throttled.

---

## Learn More

- [Kiro](https://kiro.dev/) · [Kiro Docs](https://kiro.dev/docs/)
- [OpenClaw Docs](https://docs.openclaw.ai/)
