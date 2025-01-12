# DOB CLI - Remote Command Execution with Persona-based Authentication

The **DOB CLI** provides an interface for executing remote commands securely on a host server using persona-based authentication. This document provides a comprehensive guide on how to configure and use the DOB CLI.

## Key Features
- **Persona-based Authentication:** Secure remote execution with signed tokens.
- **Proxy Commands:** Forward commands to a remote host seamlessly.
- **Configuration Management:** Easily set up and manage configurations.

## Prerequisites
1. Python 3.6+
2. Pip (Python package manager)
3. An operational host server with the DOB service running.

# **DOB Cli Installation**

## **Installation**

### **Steps to Install**

1. **Create the Installation Script**:
   Copy the following script and save it as `install_devops_bot.sh` on your system:

   ```bash
   #!/bin/bash

   # Exit immediately if any command fails
   set -e

   # Function to display a stage
   function show_stage() {
     echo -e "\n\033[1;32m>>> $1\033[0m"
   }

   # Function to install a package
   function install_package() {
     if ! command -v "$1" &>/dev/null; then
       show_stage "Installing $1..."
       if [ "$PACKAGE_MANAGER" == "apt-get" ]; then
         apt-get install -y "$2"
       elif [ "$PACKAGE_MANAGER" == "yum" ] || [ "$PACKAGE_MANAGER" == "dnf" ]; then
         $PACKAGE_MANAGER install -y "$2"
       else
         echo "Unsupported package manager."
         exit 1
       fi
     else
       echo "$1 is already installed."
     fi
   }

   # Step 1: Detect the operating system and package manager
   show_stage "Checking operating system..."
   if [ -f "/etc/debian_version" ]; then
     PACKAGE_MANAGER="apt-get"
     update_command="apt-get update -y"
   elif [ -f "/etc/os-release" ]; then
     . /etc/os-release
     if [[ "$ID" == "amzn" ]]; then
       # Amazon Linux
       PACKAGE_MANAGER="yum"
       update_command="yum update -y"
     elif [[ "$ID_LIKE" == *"rhel"* ]] || [[ "$ID" == "centos" ]] || [[ "$ID" == "fedora" ]]; then
       # RHEL-based distributions
       if command -v dnf &>/dev/null; then
         PACKAGE_MANAGER="dnf"
         update_command="dnf update -y"
       else
         PACKAGE_MANAGER="yum"
         update_command="yum update -y"
       fi
     else
       echo "Unsupported operating system."
       exit 1
     fi
   else
     echo "Unsupported operating system."
     exit 1
   fi

   # Step 2: Update and install prerequisites
   show_stage "Updating package list and installing prerequisites..."
   eval "$update_command"
   install_package "wget" "wget"
   install_package "pip3" "python3-pip"

   # Step 3: Download the .whl file
   WHL_URL="https://raw.githubusercontent.com/Devops-Bot-Official/dob-cli/master/dob_cli-0.1-py3-none-any.whl"
   WHL_FILE="dob_cli-0.1-py3-none-any.whl"

   show_stage "Downloading the package..."
   wget -q "$WHL_URL" -O "$WHL_FILE"

   # Step 4: Install the package
   show_stage "Installing the package..."
   pip3 install "$WHL_FILE"

   # Step 5: Clean up
   show_stage "Cleaning up..."
   rm -f "$WHL_FILE"

   # Final message
   show_stage "Installation complete!"
---
# Uninstalling DevOps Bot

This guide explains how to uninstall the `dob-cli` package and clean up residual files.

---

## Steps to Uninstall DevOps Bot

### 1. Uninstall the `dob-cli` Package

Run the following command to uninstall the `dob-cli` package using `pip3`:

```bash
pip3 uninstall -y devops-bot
```

This command removes the package from your Python environment.

---

### 2. Remove Residual Files

The `dob-cli` package creates additional files and directories during installation and execution. You should clean them up to fully remove the application.

#### Check the Default Directory

The default directory for the `devops-bot` files is:

```plaintext
/etc/dob-cli
```

#### Delete the Directory

Run the following command to remove the directory:

```bash
sudo rm -rf /etc/dob-cli
```

This command permanently deletes all files and configurations related to `devops-bot`.

---

### 3. Verify the Cleanup

To ensure all components are removed:

1. Check for any remaining files:
   ```bash
   ls /etc | grep dob-cli
   ```
   If nothing is listed, all files have been successfully removed.

2. Verify `pip3` no longer lists the package:
   ```bash
   pip3 list | grep dob-cli
   ```
   If nothing is listed, the package has been completely uninstalled.

---
## Usage

### 1. Configuring DOB CLI
Before running any commands, you need to configure the DOB CLI with your persona credentials.

```bash
dob config --persona-name <persona_name> \
           --token <token> \
           --private-key-path <path_to_private_key.pem> \
           --host-endpoint <host_url>
```

- **--persona-name:** Name of the persona.
- **--token:** Token associated with the persona.
- **--private-key-path:** Path to the private key file (PEM format).
- **--host-endpoint:** URL of the host endpoint (e.g., `http://<host_ip>:<port>`).

Example:
```bash
dob config --persona-name alice_dev \
           --token 54e8895b-659b-48ba-aab6-56162f3f1bc4 \
           --private-key-path /root/alice_dev_private.pem \
           --host-endpoint http://34.201.152.123:4102
```

Once configured, the CLI will save the details in `~/.dobconfig.json`.

### 2. Running Commands
After configuration, you can forward any `dob` command to the remote host by simply using:

```bash
dob <command> [options]
```

Examples:

1. **Executing a simple command:**
   ```bash
   dob vault show
   ```
   This command will forward `dob vault show` to the remote host.

2. **Executing a command with a file:**
   ```bash
   dob aws screenplay --file-path test.yaml
   ```

   If the `--file-path` option is provided, the CLI will read the content of the file and send it inline to the remote host.

### 3. Error Handling
If a command fails, the CLI will display an error message indicating the cause. Ensure that:
- The host endpoint is reachable.
- The provided token and private key are valid.
- The command syntax is correct.

## How It Works

1. **Configuration:**
   The `dob config` command stores the persona details (name, token, private key, and host endpoint) in a local configuration file (`~/.dobconfig.json`).

2. **Command Execution:**
   When you run a command using `dob`, the CLI:
   - Loads the configuration details.
   - Signs the token using the private key.
   - Prepares the command and any associated file content.
   - Sends the command to the host endpoint using an HTTP POST request.

3. **Host Processing:**
   The host server validates the persona authentication using the provided token and signature. Upon successful authentication, it executes the forwarded command and returns the result.

## Example Workflow

1. **Configuring the CLI:**
   ```bash
   dob config --persona-name alice_dev \
              --token 54e8895b-659b-48ba-aab6-56162f3f1bc4 \
              --private-key-path /root/alice_dev_private.pem \
              --host-endpoint http://34.201.152.123:4102
   ```

2. **Running a command:**
   ```bash
   dob vault show
   ```

   Output:
   ```
   Forwarding command to host: dob vault show
   Files in the vault:
   - token
   ```

3. **Executing a command with a screenplay file:**
   ```bash
   dob aws screenplay --file-path ec2.yaml
   ```

   Output:
   ```
   Forwarding command to host: dob aws screenplay <inline_file>
   Execution successful.
   ```

## Troubleshooting

1. **Config file not found:**
   If you encounter the error:
   ```
   Error: Config file not found at /root/.dobconfig.json. Please run 'dob config' first.
   ```
   Ensure that you have run the `dob config` command correctly and that the configuration file exists at the specified path.

2. **Authentication failed:**
   If authentication fails, check that:
   - The token is valid.
   - The private key matches the public key associated with the persona.

3. **Command execution failed:**
   Ensure that the command syntax is correct and that the remote host supports the specified command.

## Conclusion
The DOB CLI is a powerful tool for secure remote command execution using persona-based authentication. By following this guide, users can configure and use the CLI effectively for various tasks.

