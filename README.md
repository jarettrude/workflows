# Reusable GitHub Actions Workflows

This repository contains centralized, reusable GitHub Actions workflows that can be used across multiple projects.

## Available Workflows

### 1. Xcode Build (`xcode-build.yml`)
Build Xcode projects or workspaces with configurable options.

### 2. Xcode Test (`xcode-test.yml`)
Run tests for Xcode projects with code coverage and detailed reporting.

---

## Usage

### Using Xcode Build Workflow

Create a workflow file in your project (e.g., `.github/workflows/build.yml`):

```yaml
name: Build iOS App

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'  # Or use 'project' for .xcodeproj
      scheme: 'MyApp'
      configuration: 'Debug'
      sdk: 'iphonesimulator'
      destination: 'platform=iOS Simulator,name=iPhone 15,OS=latest'
      xcode-version: 'latest-stable'
      runs-on: 'macos-14'  # Use 'self-hosted' for your own runners
      clean-build: false
      code-sign: false
```

#### Build Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `workspace` | No | - | Path to .xcworkspace file |
| `project` | No | - | Path to .xcodeproj file (use workspace OR project) |
| `scheme` | Yes | - | Xcode scheme to build |
| `configuration` | No | `Debug` | Build configuration (Debug/Release) |
| `sdk` | No | `iphonesimulator` | SDK to build against |
| `destination` | No | `platform=iOS Simulator,name=iPhone 15,OS=latest` | Build destination |
| `xcode-version` | No | `latest-stable` | Xcode version (GitHub-hosted only) |
| `runs-on` | No | `macos-14` | Runner type |
| `clean-build` | No | `false` | Clean before building |
| `build-for-testing` | No | `false` | Build for testing |
| `code-sign` | No | `false` | Enable code signing |
| `derived-data-path` | No | - | Custom derived data path |
| `extra-build-args` | No | - | Additional xcodebuild arguments |
| `working-directory` | No | `.` | Working directory |

#### Build Workflow Secrets (for Code Signing)

| Secret | Required | Description |
|--------|----------|-------------|
| `CERTIFICATE_BASE64` | No | Base64 encoded signing certificate (.p12) |
| `CERTIFICATE_PASSWORD` | No | Password for signing certificate |
| `PROVISIONING_PROFILE_BASE64` | No | Base64 encoded provisioning profile |
| `KEYCHAIN_PASSWORD` | No | Password for temporary keychain |

**Example with code signing:**

```yaml
jobs:
  build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      configuration: 'Release'
      sdk: 'iphoneos'
      code-sign: true
    secrets:
      CERTIFICATE_BASE64: ${{ secrets.IOS_CERTIFICATE }}
      CERTIFICATE_PASSWORD: ${{ secrets.CERT_PASSWORD }}
      PROVISIONING_PROFILE_BASE64: ${{ secrets.IOS_PROVISION_PROFILE }}
      KEYCHAIN_PASSWORD: ${{ secrets.KEYCHAIN_PASSWORD }}
```

---

### Using Xcode Test Workflow

Create a workflow file in your project (e.g., `.github/workflows/test.yml`):

```yaml
name: Run Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-test.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      configuration: 'Debug'
      sdk: 'iphonesimulator'
      destination: 'platform=iOS Simulator,name=iPhone 15,OS=latest'
      enable-code-coverage: true
      parallel-testing: true
      runs-on: 'macos-14'
```

#### Test Workflow Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `workspace` | No | - | Path to .xcworkspace file |
| `project` | No | - | Path to .xcodeproj file (use workspace OR project) |
| `scheme` | Yes | - | Xcode scheme to test |
| `configuration` | No | `Debug` | Build configuration |
| `sdk` | No | `iphonesimulator` | SDK to test against |
| `destination` | No | `platform=iOS Simulator,name=iPhone 15,OS=latest` | Test destination |
| `xcode-version` | No | `latest-stable` | Xcode version (GitHub-hosted only) |
| `runs-on` | No | `macos-14` | Runner type |
| `test-plan` | No | - | Specific test plan to run |
| `only-testing` | No | - | Run only specific tests (comma-separated) |
| `skip-testing` | No | - | Skip specific tests (comma-separated) |
| `enable-code-coverage` | No | `true` | Enable code coverage |
| `parallel-testing` | No | `true` | Enable parallel testing |
| `retry-tests-on-failure` | No | `false` | Retry failed tests |
| `clean-build` | No | `false` | Clean before building |
| `derived-data-path` | No | - | Custom derived data path |
| `result-bundle-path` | No | `TestResults.xcresult` | Path for test results |
| `extra-test-args` | No | - | Additional test arguments |
| `working-directory` | No | `.` | Working directory |

**Example with specific tests:**

```yaml
jobs:
  test:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-test.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      only-testing: 'MyAppTests/LoginTests,MyAppTests/ProfileTests'
      enable-code-coverage: true
```

---

## Using Self-Hosted Runners

Self-hosted runners **do not consume GitHub Actions minutes**, making them cost-free for private repositories (you only pay for your hardware/electricity).

### Quick Setup Example

```yaml
jobs:
  build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      runs-on: 'self-hosted'  # Or use '["self-hosted", "macOS", "ARM64"]' for labels
```

For detailed setup instructions, see [SELF_HOSTED_RUNNERS.md](./SELF_HOSTED_RUNNERS.md).

---

## Advanced Examples

### Multi-Platform Build

```yaml
name: Multi-Platform Build

on: [push, pull_request]

jobs:
  ios-build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp-iOS'
      sdk: 'iphonesimulator'
      destination: 'platform=iOS Simulator,name=iPhone 15,OS=latest'

  macos-build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp-macOS'
      sdk: 'macosx'
      destination: 'platform=macOS'
```

### Matrix Testing

```yaml
name: Matrix Testing

on: [push, pull_request]

jobs:
  test:
    strategy:
      matrix:
        include:
          - destination: 'platform=iOS Simulator,name=iPhone 15,OS=latest'
            name: 'iPhone 15'
          - destination: 'platform=iOS Simulator,name=iPhone 15 Pro Max,OS=latest'
            name: 'iPhone 15 Pro Max'
          - destination: 'platform=iOS Simulator,name=iPad Pro (12.9-inch) (6th generation),OS=latest'
            name: 'iPad Pro'

    name: Test on ${{ matrix.name }}
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-test.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      destination: ${{ matrix.destination }}
```

### Build and Test Pipeline

```yaml
name: Build and Test

on: [push, pull_request]

jobs:
  build:
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-build.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      configuration: 'Debug'

  test:
    needs: build
    uses: YOUR_USERNAME/workflows/.github/workflows/xcode-test.yml@main
    with:
      workspace: 'MyApp.xcworkspace'
      scheme: 'MyApp'
      enable-code-coverage: true
```

---

## Preparing Certificates for Code Signing

To use code signing with these workflows, you need to encode your certificates and provisioning profiles:

```bash
# Encode certificate
base64 -i YourCertificate.p12 | pbcopy

# Encode provisioning profile
base64 -i YourProfile.mobileprovision | pbcopy
```

Then add these as secrets in your repository settings.

---

## Tips

1. **Use specific Xcode versions** for reproducible builds on GitHub-hosted runners
2. **Enable code coverage** to track test quality over time
3. **Use matrix strategies** to test on multiple iOS versions/devices
4. **Self-hosted runners** save costs for private repos and allow custom hardware
5. **Pin workflow versions** using tags (e.g., `@v1.0`) instead of `@main` for stability

---

## Contributing

Feel free to submit issues or pull requests to improve these workflows.

## License

MIT License - feel free to use in your projects.
