
# Planter Deployment CLI

**Disclaimer:** This platform is currently in the design phase and is not yet developed. This README serves as an overview of the proposed architecture and features.

The Planter CLI is a command-line tool designed to manage service deployments for the Plantarium platform. It supports versioned deployments, rollback capabilities, and validation of new versions before activation, making it ideal for CI/CD pipelines and local deployment workflows.

---

## Features

### Deployment
- **Deploy New Versions**:
  Deploy service binaries and configurations with version management.
  ```bash
  planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2
  ```
- **Generate Versions**:
  Automatically generate versions using semantic versioning increments.
  ```bash
  planter-cli deploy --binary app --config config.yaml --service my-service --generate-version --increment minor
  ```
- **Deploy Without Activation**:
  Deploy a version without making it the current active version.
  ```bash
  planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2 --no-activate
  ```

### Validation
- **Validate Deployments**:
  Test if a deployed version can start successfully without affecting live services.
  ```bash
  planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2 --validate
  ```

### Version Management
- **Switch Versions**:
  Change the active version of a service to a previously deployed version.
  ```bash
  planter-cli switch-version --service my-service --version 1.0.1
  ```
- **List Versions**:
  View all deployed versions and their status.
  ```bash
  planter-cli list-versions --service my-service
  ```
  Example Output:
  ```
  Available versions for my-service:
  - 1.0.0 (tag: stable)
  - 1.0.1 (active, tag: testing)
  - 1.0.2
  ```

- **Remove Versions**:
  Delete an old version, optionally forcing removal of the active version.
  ```bash
  planter-cli remove-version --service my-service --version 1.0.1 --force
  ```

### Rollback
- **Rollback to Previous Versions**:
  Quickly revert to the last successfully deployed version.
  ```bash
  planter-cli rollback --service my-service
  ```

---

## How It Works Under the Hood

### **Communication Between CLI and Planter**
The Planter CLI communicates with the Planter service using **Linux domain sockets** for secure and efficient local communication. This design avoids exposing the Planter service over the internet or HTTP while ensuring seamless interaction between the CLI and the service.

- **Default Socket Address**:
  The Planter service listens on a Unix domain socket at:
  ```
  /var/run/plantarium/planter.sock
  ```

- **Customization for Multiple Instances**:
  - The default address can be overridden using:
    - An environment variable: `PLANTER_SOCKET`
    - A CLI flag: `--socket`
  - Example paths for multiple instances:
    ```
    /var/run/plantarium/planter-instance-1.sock
    /var/run/plantarium/planter-instance-2.sock
    ```

### **Service Initialization**
- On startup, the Planter service ensures the parent directory exists (`/var/run/plantarium`) and creates the socket file with restricted permissions (0700).
- The CLI connects to this socket to send deployment commands and retrieve responses.

### **Why Linux Domain Sockets?**
1. **Local and Secure**: Communication is restricted to the same machine and protected by file permissions.
2. **High Performance**: Faster than TCP/IP since there's no network overhead.
3. **Ease of Use**: Supported natively in Go's `net` package and many other frameworks.

---

## Usage Examples

### Deploy and Activate a Version
```bash
planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2
```

### Deploy Without Activation
```bash
planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2 --no-activate
```

### Validate Deployment
```bash
planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2 --validate
```

### Switch Active Version
```bash
planter-cli switch-version --service my-service --version 1.0.1
```

### List Deployed Versions
```bash
planter-cli list-versions --service my-service
```

### Remove a Version
```bash
planter-cli remove-version --service my-service --version 1.0.1 --force
```

---

## Notes
1. **Validation Mode**:
   - Use `--validate` to test if the service can start successfully without affecting live traffic.
   - Validation mode starts the service on a random port and shuts it down after testing.

2. **Version Overwriting**:
   - Use the `--overwrite` flag to replace an existing version during deployment.

3. **Tagging Versions**:
   - Attach tags to versions for better categorization.
   ```bash
   planter-cli deploy --binary app --config config.yaml --service my-service --version 1.0.2 --tags stable,production
   ```

---

## Security
- Secure deployments using HTTPS or SSH for file transfers.
- Validate uploaded artifacts to ensure integrity before deployment.

---

## Future Enhancements
- Support for metadata management.
- Enhanced logging and debug modes.
- Integration with CI/CD pipelines for automated deployments.
