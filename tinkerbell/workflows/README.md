# Kubernetes Control Plane Tinkerbell Workflow

This repository contains Tinkerbell workflow files for automating the installation and configuration of a Kubernetes control plane node on Ubuntu 24.04.

## Prerequisites

- A running Tinkerbell server
- Target hardware that supports PXE boot
- Network connectivity between Tinkerbell server and target hardware
- Ubuntu 24.04 server image URL

## Directory Structure

```
tinkerbell/workflows/
├── k8s-control-plane-template.yaml  # Template defining the workflow steps
├── hardware-data.yaml              # Hardware configuration for target machine
├── workflow.yaml                   # Main workflow definition
├── variables.json                  # Configuration variables
└── README.md                       # This file
```

## Workflow Components

### 1. Template (k8s-control-plane-template.yaml)
Defines the sequence of actions to be performed:
- OS installation (Ubuntu 24.04)
- System configuration
- CRI-O installation
- Kubernetes components installation
- Control plane initialization
- CNI plugins installation (including portmap plugin required by Flannel)
- Flannel CNI setup

### 2. Hardware Data (hardware-data.yaml)
Contains hardware-specific configuration:
- Hostname: k8s-control-plane
- Network settings
- Storage configuration
- Operating system details

### 3. Workflow (workflow.yaml)
Combines template and hardware data to create the complete workflow.

## Configuration Variables

The workflow requires several variables to be set. You can set these variables in two ways:

### 1. Using a Variables File (Recommended)

Create a `variables.json` file with the following structure:

```json
{
  "device_1": {
    "ip": "192.168.1.100",        # IP address of your target machine
    "user": "ubuntu",             # SSH user for the target machine
    "ssh_key": "-----BEGIN OPENSSH PRIVATE KEY-----\nYour SSH private key here\n-----END OPENSSH PRIVATE KEY-----",  # SSH private key
    "mac": "00:11:22:33:44:55"    # MAC address of your target machine
  },
  "image_url": "http://archive.ubuntu.com/ubuntu/dists/noble/main/installer-amd64/current/images/netboot/mini.iso"
}
```

When creating the workflow, use the variables file:

```bash
tink workflow create \
  --template-id <template-id> \
  --hardware-id <hardware-id> \
  --file workflow.yaml \
  --vars-file variables.json
```

### 2. Using Environment Variables

You can also set the variables using environment variables:

```bash
export TINKERBELL_DEVICE_1_IP="192.168.1.100"
export TINKERBELL_DEVICE_1_USER="ubuntu"
export TINKERBELL_DEVICE_1_SSH_KEY="$(cat ~/.ssh/id_rsa)"
export TINKERBELL_DEVICE_1_MAC="00:11:22:33:44:55"
export TINKERBELL_IMAGE_URL="http://archive.ubuntu.com/ubuntu/dists/noble/main/installer-amd64/current/images/netboot/mini.iso"

tink workflow create \
  --template-id <template-id> \
  --hardware-id <hardware-id> \
  --file workflow.yaml
```

### Required Variables

The following variables must be set:

- `device_1.ip`: IP address of the target machine
- `device_1.user`: SSH user for the target machine
- `device_1.ssh_key`: SSH private key for authentication
- `device_1.mac`: MAC address of the target machine (for hardware registration)
- `image_url`: URL to the Ubuntu 24.04 server image

### Security Considerations for Variables

- Never commit the `variables.json` file with real credentials to version control
- Use SSH keys with appropriate permissions (600)
- Consider using a secrets management solution for production environments
- The variables file should have restricted permissions (600)
- Ensure the SSH key has the minimum required permissions on the target machine

## Usage Instructions

1. **Prepare Your Environment**
   ```bash
   # Ensure Tinkerbell server is running
   # Verify network connectivity to target hardware
   ```

2. **Register Hardware**
   ```bash
   # Register your target hardware with Tinkerbell
   tink hardware create --file hardware-data.yaml
   ```

3. **Create Template**
   ```bash
   # Create the workflow template
   tink template create --file k8s-control-plane-template.yaml
   ```

4. **Create Workflow**
   ```bash
   # Create the workflow with your hardware ID and template ID
   tink workflow create \
     --template-id <template-id> \
     --hardware-id <hardware-id> \
     --file workflow.yaml
   ```

5. **Start the Workflow**
   ```bash
   # Start the workflow
   tink workflow create --id <workflow-id>
   ```

## Network Configuration

The workflow assumes:
- DHCP is available for network configuration
- Target machine can reach the internet for package downloads
- Flannel CNI will use the CIDR range: 10.244.0.0/16
- CNI plugins (including portmap) will be installed in `/opt/cni/bin`

## Verification

After the workflow completes successfully:

1. SSH into the target machine using your SSH key:
   ```bash
   ssh -i ~/.ssh/your_key ubuntu@<target-ip>
   ```

2. Verify Kubernetes control plane:
   ```bash
   kubectl get nodes
   kubectl get pods -A
   ```

3. Check CRI-O status:
   ```bash
   systemctl status crio
   ```

4. Verify CNI plugins installation:
   ```bash
   ls -l /opt/cni/bin
   # Should show portmap and other CNI plugins
   ```

## Troubleshooting

1. **Workflow Fails to Start**
   - Verify Tinkerbell server is running
   - Check network connectivity
   - Ensure hardware is registered correctly

2. **Package Installation Issues**
   - Check internet connectivity
   - Verify package repositories are accessible
   - Check system logs: `journalctl -xe`

3. **Kubernetes Initialization Issues**
   - Check kubelet logs: `journalctl -u kubelet`
   - Verify system requirements are met
   - Check if required ports are open
   - Verify CNI plugins are installed correctly: `ls -l /opt/cni/bin`

4. **CNI/Flannel Issues**
   - Verify CNI plugins are present in `/opt/cni/bin`
   - Check Flannel pod logs: `kubectl logs -n kube-system -l app=flannel`
   - Verify CNI configuration: `ls -l /etc/cni/net.d/`

## Security Considerations

- The workflow uses root access for installation
- Consider changing default passwords after installation
- Review and adjust firewall rules as needed
- Consider using secure communication channels for sensitive data

## Support

For issues related to:
- Tinkerbell: [Tinkerbell GitHub](https://github.com/tinkerbell/tinkerbell)
- Kubernetes: [Kubernetes Documentation](https://kubernetes.io/docs/)
- CRI-O: [CRI-O GitHub](https://github.com/cri-o/cri-o)
- CNI Plugins: [CNI Plugins GitHub](https://github.com/containernetworking/plugins) 