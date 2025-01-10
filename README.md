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

## Installation


1. Build and install the package:
   ```bash
   python3 setup.py sdist bdist_wheel
   pip install dist/dob_cli-1.0.0-py3-none-any.whl
   ```

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

