# Self-Hosted Runners Guide

This guide will help you set up self-hosted GitHub Actions runners on your home server for running Xcode builds without consuming GitHub Actions minutes.

## Why Self-Hosted Runners?

### Advantages
- **Cost Savings**: Self-hosted runners do NOT consume GitHub Actions minutes for private repositories
- **Custom Hardware**: Use your own Mac hardware (Mac Mini, Mac Studio, etc.)
- **Faster Builds**: Potentially faster with local caching and powerful hardware
- **Custom Environment**: Pre-install tools, dependencies, and configurations
- **Data Privacy**: Sensitive code never leaves your infrastructure

### Considerations
- **Maintenance**: You're responsible for updates, security, and uptime
- **Power Costs**: Your electricity bill (usually minimal)
- **Network**: Requires stable internet connection
- **Initial Setup**: One-time setup effort

---

## Prerequisites

### Hardware
- **Mac Computer** (Mac Mini, Mac Studio, MacBook Pro, iMac, etc.)
  - Minimum: Intel Mac with 8GB RAM
  - Recommended: Apple Silicon (M1/M2/M3) with 16GB+ RAM
  - For best performance: Mac Studio with 32GB+ RAM

### Software
- **macOS** 12.0 (Monterey) or later
  - Latest macOS recommended for latest Xcode support
- **Xcode** installed (version matching your development needs)
  - Install from App Store or Apple Developer
- **Homebrew** (optional but recommended)

### Network
- Stable internet connection
- Static local IP (recommended) or DHCP reservation
- Firewall allows outbound HTTPS (GitHub API access)

---

## Step-by-Step Setup

### 1. Prepare Your Mac

#### Install Xcode
```bash
# Check if Xcode is installed
xcode-select -p

# If not installed, download from App Store or:
# https://developer.apple.com/xcode/

# Accept Xcode license
sudo xcodebuild -license accept

# Install Xcode Command Line Tools
xcode-select --install
```

#### Install Homebrew (Optional)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### Install Additional Tools
```bash
# CocoaPods (if needed)
sudo gem install cocoapods

# SwiftLint (optional)
brew install swiftlint

# xcbeautify (optional, for better build output)
brew install xcbeautify
```

#### Configure Mac for Always-On Operation

1. **System Settings → Energy Saver** (or Battery):
   - Prevent computer from sleeping automatically: **ON**
   - Wake for network access: **ON**
   - Start up automatically after a power failure: **ON**

2. **System Settings → Sharing**:
   - Enable "Remote Login" (SSH) for remote management
   - Set computer name (e.g., "GitHub-Runner-1")

3. **Disable Screen Saver**:
   - System Settings → Screen Saver
   - Set to "Never"

4. **Create Dedicated User** (Recommended):
   ```bash
   # Create a user for the runner
   sudo dscl . -create /Users/githubrunner
   sudo dscl . -create /Users/githubrunner UserShell /bin/bash
   sudo dscl . -create /Users/githubrunner RealName "GitHub Runner"
   sudo dscl . -create /Users/githubrunner UniqueID 510
   sudo dscl . -create /Users/githubrunner PrimaryGroupID 20
   sudo dscl . -create /Users/githubrunner NFSHomeDirectory /Users/githubrunner
   sudo dscl . -passwd /Users/githubrunner YourSecurePassword
   sudo createhomedir -c -u githubrunner

   # Add to admin group (optional)
   sudo dseditgroup -o edit -a githubrunner -t user admin
   ```

---

### 2. Add Runner to GitHub

#### For Repository-Level Runner

1. Go to your repository on GitHub
2. Navigate to **Settings → Actions → Runners**
3. Click **New self-hosted runner**
4. Select **macOS** as the operating system
5. Choose architecture:
   - **ARM64** for Apple Silicon (M1/M2/M3)
   - **x64** for Intel Macs

#### For Organization-Level Runner (Recommended for Multiple Repos)

1. Go to your organization on GitHub
2. Navigate to **Settings → Actions → Runners**
3. Click **New runner → New self-hosted runner**
4. Select **macOS** and architecture

GitHub will provide you with setup commands like:

```bash
# Create a folder
mkdir actions-runner && cd actions-runner

# Download the latest runner package
curl -o actions-runner-osx-arm64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-osx-arm64-2.311.0.tar.gz

# Extract the installer
tar xzf ./actions-runner-osx-arm64-2.311.0.tar.gz
```

---

### 3. Configure the Runner

Run the configuration script provided by GitHub:

```bash
# Configure the runner
./config.sh --url https://github.com/YOUR_ORG/YOUR_REPO --token YOUR_TOKEN

# You'll be prompted for:
# - Runner name (e.g., "mac-mini-m2")
# - Runner group (default is fine)
# - Labels (e.g., "self-hosted,macOS,ARM64,xcode-15")
# - Work folder (default _work is fine)
```

#### Recommended Labels

Add custom labels to help target specific runners:

- `self-hosted` (automatic)
- `macOS` (automatic)
- `ARM64` or `X64` (automatic)
- `xcode-15` (your Xcode version)
- `ios` (if for iOS builds)
- `swift` (if for Swift projects)
- Custom labels like: `fast`, `local`, `home-server`

Example:
```
Enter any additional labels (ex. label-1,label-2): xcode-15,ios,swift
```

---

### 4. Install Runner as a Service (Automatic Startup)

This ensures the runner starts automatically on boot:

```bash
# Install as a LaunchAgent (runs when user is logged in)
./svc.sh install

# Start the service
./svc.sh start

# Check status
./svc.sh status

# The runner will now start automatically on boot
```

#### Alternative: Run as LaunchDaemon (runs at system startup)

```bash
# Install as LaunchDaemon (runs even when no user is logged in)
sudo ./svc.sh install

# Start the service
sudo ./svc.sh start
```

#### Verify Runner is Active

1. Go to your repository/organization Settings → Actions → Runners
2. You should see your runner listed as "Idle" (green)

---

### 5. Using Your Self-Hosted Runner

Update your workflow to use the self-hosted runner:

```yaml
jobs:
  build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      runs-on: 'self-hosted'  # Use your runner
      # Or with specific labels:
      # runs-on: '["self-hosted", "macOS", "ARM64", "xcode-15"]'
```

Using JSON array for labels (more specific):
```yaml
jobs:
  build:
    runs-on: ["self-hosted", "macOS", "ARM64"]
```

---

## Security Best Practices

### 1. Runner Security

- **Use dedicated user account** for the runner (not your personal account)
- **Keep macOS and Xcode updated** for security patches
- **Enable FileVault** disk encryption
- **Enable Firewall** in System Settings
- **Use strong passwords** for user accounts
- **Disable auto-login** for security

### 2. Network Security

- **Place behind firewall**: Runner only needs outbound HTTPS to GitHub
- **Use static IP or DHCP reservation** for consistent connectivity
- **Consider VPN** if accessing from outside your network
- **Monitor access logs** regularly

### 3. Repository Permissions

- **Use separate runners** for different trust levels
  - Public repos: Dedicated runners (higher risk)
  - Private repos: Can share runners
- **Review Actions permissions** regularly
- **Limit who can trigger workflows** on self-hosted runners
- **Use environment protection rules** for sensitive deployments

### 4. Secrets Management

- **Never hardcode secrets** in workflows
- **Use GitHub Secrets** for sensitive data
- **Rotate secrets regularly**
- **Use environment-specific secrets** when possible

---

## Maintenance

### Regular Updates

```bash
# Update macOS
sudo softwareupdate -ia

# Update Xcode (via App Store or download)

# Update Homebrew packages
brew update && brew upgrade

# Update CocoaPods
sudo gem update cocoapods

# Update the runner software (when GitHub notifies you)
cd ~/actions-runner
./config.sh remove  # Remove old runner
# Then reconfigure with new version
```

### Monitor Runner Health

```bash
# Check runner status
cd ~/actions-runner
./svc.sh status

# View runner logs
tail -f ~/actions-runner/_diag/Runner_*.log

# System resource usage
top

# Disk space
df -h
```

### Clean Up Build Artifacts

```bash
# Clear Xcode derived data (saves disk space)
rm -rf ~/Library/Developer/Xcode/DerivedData/*

# Clear CocoaPods cache
pod cache clean --all

# Clear Homebrew cache
brew cleanup
```

### Automated Cleanup Script

Create a daily cleanup cron job:

```bash
# Edit crontab
crontab -e

# Add this line (runs daily at 2 AM)
0 2 * * * rm -rf ~/Library/Developer/Xcode/DerivedData/* && pod cache clean --all
```

---

## Troubleshooting

### Runner Not Connecting

```bash
# Check internet connectivity
ping github.com

# Check runner logs
cd ~/actions-runner
cat _diag/Runner_*.log

# Restart the service
./svc.sh stop
./svc.sh start
```

### Xcode Command Line Tools Issues

```bash
# Reset Xcode command line tools
sudo xcode-select --reset

# Point to correct Xcode
sudo xcode-select -s /Applications/Xcode.app/Contents/Developer

# Verify
xcodebuild -version
```

### Disk Space Issues

```bash
# Check disk usage
df -h

# Find large files
du -sh ~/Library/Developer/Xcode/DerivedData
du -sh ~/Library/Caches

# Clean up
rm -rf ~/Library/Developer/Xcode/DerivedData/*
rm -rf ~/Library/Caches/CocoaPods
```

### Runner Offline After macOS Update

```bash
# Reconfigure the service
cd ~/actions-runner
./svc.sh stop
./svc.sh start

# If still issues, reinstall service
./svc.sh uninstall
./svc.sh install
./svc.sh start
```

---

## Advanced Configuration

### Multiple Runners on One Mac

You can run multiple runner instances:

```bash
# Create separate directories
mkdir ~/actions-runner-1
mkdir ~/actions-runner-2

# Configure each separately
cd ~/actions-runner-1
./config.sh --url ... --token ...
./svc.sh install

cd ~/actions-runner-2
./config.sh --url ... --token ...
./svc.sh install
```

### Different Xcode Versions

Keep multiple Xcode versions for compatibility:

```bash
# Install Xcode from Apple Developer
# Rename Xcode apps
sudo mv /Applications/Xcode.app /Applications/Xcode_15.0.app
# Download and install different version
# Rename it to Xcode_14.3.app

# Switch versions in workflow
sudo xcode-select -s /Applications/Xcode_15.0.app/Contents/Developer
```

### Custom Environment Variables

Add to runner's environment:

```bash
# Edit runner's .env file
cd ~/actions-runner
nano .env

# Add variables
CUSTOM_VAR=value
HOMEBREW_PREFIX=/opt/homebrew
```

---

## Cost Comparison

### GitHub-Hosted Runners (for private repos)
- **macOS runners**: $0.08/minute
- **Typical iOS build**: 10-20 minutes
- **Cost per build**: $0.80 - $1.60
- **100 builds/month**: $80 - $160/month

### Self-Hosted Runner (Mac Mini M2)
- **Hardware**: $599 one-time (Mac Mini M2)
- **Electricity**: ~$5/month (24/7 at 0.15 kW)
- **Internet**: Usually included in existing plan
- **Total monthly**: ~$5 + ($599/24 months) = ~$30/month
- **Break-even**: After ~4-6 months of regular builds

**Conclusion**: Self-hosted runners pay for themselves quickly if you build regularly!

---

## Example Runner Configuration

Here's a complete example for a production-ready runner:

```bash
# 1. Create dedicated user
sudo dscl . -create /Users/githubrunner
# ... (user setup commands from above)

# 2. Login as githubrunner user
su - githubrunner

# 3. Install Xcode (download from Apple Developer)

# 4. Configure Xcode
sudo xcodebuild -license accept
xcode-select --install

# 5. Install tools
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
sudo gem install cocoapods
brew install swiftlint xcbeautify

# 6. Set up runner
mkdir ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-osx-arm64-2.311.0.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-osx-arm64-2.311.0.tar.gz
tar xzf ./actions-runner-osx-arm64-2.311.0.tar.gz

# 7. Configure
./config.sh --url https://github.com/YOUR_ORG --token YOUR_TOKEN
# Add labels: xcode-15,ios,swift,fast

# 8. Install as service
./svc.sh install
./svc.sh start

# 9. Set up cleanup cron
crontab -e
# Add: 0 2 * * * rm -rf ~/Library/Developer/Xcode/DerivedData/*
```

---

## Resources

- [GitHub Actions Self-Hosted Runners Documentation](https://docs.github.com/en/actions/hosting-your-own-runners)
- [GitHub Actions Security Hardening](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)
- [Xcode Command Line Tools](https://developer.apple.com/xcode/)
- [Actions Runner Releases](https://github.com/actions/runner/releases)

---

## Next Steps

1. Set up your Mac as described above
2. Install the runner
3. Test with a simple workflow from this repo
4. Monitor for a few days to ensure stability
5. Migrate production workflows
6. Set up monitoring and maintenance routines

Good luck with your self-hosted runner setup! 🚀
