# **DOB CLI - Remote Command Execution with Persona-based Authentication**

The **DOB CLI** provides an interface for executing remote commands securely on a host server using persona-based authentication. This document provides a comprehensive guide on how to configure and use the DOB CLI.

## Key Features
- **Persona-based Authentication:** Secure remote execution with signed tokens.
- **Proxy Commands:** Forward commands to a remote host seamlessly.
- **Configuration Management:** Easily set up and manage configurations.

## Prerequisites
1. Python 3.6+
2. Pip (Python package manager)
3. An operational host server with the DOB service running.

# **DOB CLI Installation**

## **Installation**

### **🔹 Linux Installation**

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
   WHL_URL="https://raw.githubusercontent.com/Devops-Bot-Official/dob-cli/master/dob_cli-1.0.0-py3-none-any.whl"
   WHL_FILE="dob_cli-1.0.0-py3-none-any.whl"

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
   ```

2. **Run the installation script**:
   ```bash
   chmod +x install_devops_bot.sh
   ./install_devops_bot.sh
   ```

3. **Verify Installation**:
   ```bash
   dob --help
   ```

---

### **🔹 macOS Installation**

1. **Create the Installation Script**:
   Copy the following script and save it as `install_devops_bot_macos.sh` on your system:

   ```bash
   #!/bin/bash

   # Exit immediately if any command fails
   set -e

   # Function to display a stage
   function show_stage() {
     echo -e "
[1;32m>>> $1[0m"
   }

   # Step 1: Check if Homebrew is installed
   if ! command -v brew &>/dev/null; then
     show_stage "Installing Homebrew..."
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   else
     show_stage "Homebrew is already installed."
   fi

   # Step 2: Ensure Python3 and pip3 are installed
   if ! command -v python3 &>/dev/null; then
     show_stage "Installing Python3..."
     brew install python
   else
     show_stage "Python3 is already installed."
   fi

   if ! command -v pip3 &>/dev/null; then
     show_stage "Ensuring pip3 is installed..."
     python3 -m ensurepip --default-pip
   else
     show_stage "pip3 is already installed."
   fi

   # Step 3: Download the .whl file
   WHL_URL="https://raw.githubusercontent.com/Devops-Bot-Official/dob-cli/master/dob_cli-1.0.0-cp39-cp39-macosx_11_0_universal2.whl"
   WHL_FILE="dob_cli-1.0.0-cp39-cp39-macosx_11_0_universal2.whl"

   show_stage "Downloading the package..."
   curl -L -o "$WHL_FILE" "$WHL_URL"

   # Step 4: Install the package
   show_stage "Installing DOB CLI..."
   pip3 install "$WHL_FILE"

   # Step 5: Clean up
   show_stage "Cleaning up..."
   rm -f "$WHL_FILE"

   # Final message
   show_stage "Installation complete! You can now use 'dob'."
   ```

2. **Run the installation script**:
   ```bash
   chmod +x install_devops_bot_macos.sh
   ./install_devops_bot_macos.sh
   ```

3. **Verify Installation**:
   ```bash
   dob --help
   ```

1. **Download and Run the Installation Script**:
   ```bash
   curl -L -o install_devops_bot_macos.sh https://raw.githubusercontent.com/Devops-Bot-Official/dob-cli/master/install_devops_bot_macos.sh
   chmod +x install_devops_bot_macos.sh
   ./install_devops_bot_macos.sh
   ```

2. **Verify Installation**:
   ```bash
   dob --help
   ```

---

# **Uninstalling DOB CLI**

## Steps to Uninstall DOB CLI

### 1. Uninstall the `dob-cli` Package

```bash
pip3 uninstall -y dob-cli
```

### 2. Remove Residual Files

```bash
sudo rm -rf /etc/dob-cli
```

### 3. Verify Cleanup

```bash
pip3 list | grep dob-cli
```

---

# **Usage**

### 1. Configuring DOB CLI

```bash
dob config --persona-name <persona_name> \
           --token <token> \
           --private-key-path <path_to_private_key.pem> \
           --host-endpoint <host_url>
```

### 2. Running Commands

```bash
dob <command> [options]
```

Examples:
```bash
dob vault show
dob aws screenplay --file-path test.yaml
```

---

# **📢 Support**
If you encounter issues, open a ticket in our GitHub repository **[here](https://github.com/Devops-Bot-Official/dob-cli/issues)**.

🚀 **Enjoy using DOB CLI!**

