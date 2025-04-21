# Servy - Discord Minecraft Server Controller

![Servy](https://github.com/user-attachments/assets/6f8dc795-4292-428e-99da-dce3b272f532)


A Discord bot powered by AWS Lambda that allows server members to start, stop, and monitor a Minecraft server hosted on AWS EC2, with automatic shutdown when idle to save costs.

## Features

- 🎮 **Discord Integration**: Control your Minecraft server directly from Discord
- 💰 **Cost Saving**: Automatically shuts down when no players are online

## Architecture

Servy consists of two main components:

1. **Discord Bot Lambda**: Handles Discord interactions and server control commands
2. **Auto-Shutdown Lambda**: Monitors player activity and shuts down idle servers

The solution uses AWS DynamoDB to track server activity and idle time.

## Prerequisites

- AWS Account
- Discord Application with Bot
- Minecraft Server running on EC2
- Node.js 18+ for local development

## Setup Instructions

### 1. AWS Resources

First, set up the required AWS resources:

```bash
# Install AWS CLI if you haven't already
pip install awscli

# Configure AWS credentials
aws configure
```

Create a DynamoDB table:

```bash
aws dynamodb create-table \
  --table-name MinecraftServerStatus \
  --attribute-definitions AttributeName=ServerId,AttributeType=S \
  --key-schema AttributeName=ServerId,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
```

### 2. Discord Bot Setup

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Create a new application
3. Add a bot to your application
4. Enable all Privileged Gateway Intents
5. Copy your bot token and application ID
6. Use the OAuth2 URL Generator to create an invite link with the `applications.commands` scope
7. Invite the bot to your server

### 3. Lambda Function Deployment

#### Discord Bot Lambda:

1. Create a new Lambda function with Node.js 14+ runtime
2. Set the following environment variables:
   - `DISCORD_TOKEN`: Your Discord bot token
   - `DISCORD_PUBLIC_KEY`: Your Discord application public key
   - `APPLICATION_ID`: Your Discord application ID
   - `INSTANCE_ID`: Your EC2 instance ID
   - `MINECRAFT_REGION`: AWS region of your EC2 instance (default: 'us-east-1')
   - `ADMIN_ROLE_ID`: Discord role ID that has permission to stop the server
   - `DYNAMO_TABLE`: Name of your DynamoDB table (default: 'MinecraftServerStatus')

3. Deploy the code from the `discord-bot` directory

#### Auto-Shutdown Lambda:

1. Create another Lambda function with Node.js 14+ runtime
2. Set the following environment variables:
   - `INSTANCE_ID`: Your EC2 instance ID
   - `MINECRAFT_REGION`: AWS region of your EC2 instance (default: 'us-east-1')
   - `DYNAMO_TABLE`: Name of your DynamoDB table (default: 'MinecraftServerStatus')

3. Deploy the code from the `auto-shutdown` directory
4. Set up a CloudWatch Events rule to trigger this Lambda every 5 minutes

### 4. IAM Permissions

Both Lambda functions need IAM permissions:

- EC2 permissions: `ec2:DescribeInstances`, `ec2:DescribeInstanceStatus`, `ec2:StartInstances`, `ec2:StopInstances`
- DynamoDB permissions: `dynamodb:GetItem`, `dynamodb:PutItem`, `dynamodb:UpdateItem`

## Command Reference

Servy supports the following Discord slash commands:

- `/server` - Shows server control buttons (Start, Status, Stop)
- `/players` - Lists players currently online
- `/uptime` - Shows how long the server has been running
- `/help` - Displays help information

## Configuration Options

- **Idle Timeout**: Change `IDLE_TIMEOUT_SECONDS` in auto-shutdown Lambda (default: 600 seconds / 10 minutes)
- **Startup Grace Period**: Change `STARTUP_GRACE_PERIOD_MINUTES` in auto-shutdown Lambda (default: 15 minutes)

## Troubleshooting

### Common Issues

- **405 Method Not Allowed**: Check your Discord application credentials
- **Lambda Timeouts**: Increase Lambda timeout settings to at least 15 seconds
- **Missing Minecraft Server Status**: Ensure port 25565 is open in your EC2 security group

### Viewing Logs

Check CloudWatch Logs for detailed log information:

```bash
aws logs get-log-events --log-group-name /aws/lambda/YOUR_LAMBDA_NAME --limit 50
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the project
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Acknowledgements

- [Discord.js](https://discord.js.org/)
- [AWS SDK for JavaScript](https://aws.amazon.com/sdk-for-javascript/)
- [minecraft-server-util](https://github.com/PassTheMayo/minecraft-server-util)
- [node-mcstatus](https://github.com/jonhoo/node-mcstatus)
