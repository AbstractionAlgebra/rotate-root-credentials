# Linux Root Password Rotation with Ansible

Automated root password rotation for Ubuntu 22.04+ and RHEL 8+ systems using Ansible. Designed for 50-device networks with DISA STIG compliance requirements.

## Quick Start

1. **Configure inventory** - Update `inventory/production.yml` with your systems
2. **Set password** - Edit and encrypt `vault/secrets.yml` with new root password
3. **Run rotation** - Execute the playbook with `ansible-playbook rotate-creds.yml --ask-vault-pass`

## Project Structure

```
├── ansible.cfg                    # Ansible configuration
├── inventory/
│   └── production.yml             # System inventory (Ubuntu & RHEL)
├── group_vars/
│   └── all.yml                    # Common variables for all systems
├── roles/
│   └── credential-rotation/       # Main rotation role
│       ├── tasks/main.yml         # Password rotation logic
│       └── vars/main.yml          # Role variables
├── rotate-creds.yml                # Main rotation playbook
├── vault/
│   └── secrets.yml                # Encrypted password storage
├── logs/                          # Rotation logs and reports
└── README.md                      # This documentation
```

## Setup Instructions

### 1. Inventory Configuration

Edit `inventory/production.yml` to include your systems:

```yaml
ubuntu:
  hosts:
    ubuntu-server-01.domain.com:
    ubuntu-server-02.domain.com:
    # Add your Ubuntu 22.04+ systems

rhel:
  hosts:
    rhel-server-01.domain.com:
    rhel-server-02.domain.com:
    # Add your RHEL 8+ systems
```

### 2. Password Configuration

Set your new root password in the vault:

```bash
# Edit the vault file and set a DISA STIG compliant password
ansible-vault edit vault/secrets.yml

# Or encrypt an existing file
ansible-vault encrypt vault/secrets.yml
```

**Password Requirements (DISA STIG):**
- Minimum 14 characters
- Must contain uppercase letters
- Must contain lowercase letters
- Must contain numbers
- Must contain special characters

### 3. SSH Key Setup

Ensure your SSH keys are configured for root access on target systems:

```bash
# Copy your public key to all systems
ssh-copy-id root@target-system.domain.com
```

## Running Password Rotation

### Standard Rotation (All Systems)

```bash
# Run password rotation on all systems
ansible-playbook rotate-creds.yml --ask-vault-pass

# Run with specific inventory
ansible-playbook -i inventory/production.yml rotate-creds.yml --ask-vault-pass

# Dry run to check what would happen
ansible-playbook rotate-creds.yml --ask-vault-pass --check
```

### Targeted Rotation

```bash
# Rotate passwords only on Ubuntu systems
ansible-playbook rotate-creds.yml --ask-vault-pass --limit ubuntu

# Rotate passwords only on RHEL systems
ansible-playbook rotate-creds.yml --ask-vault-pass --limit rhel

# Rotate password on specific host
ansible-playbook rotate-creds.yml --ask-vault-pass --limit ubuntu-server-01.domain.com
```

## Monitoring and Logging

### Log Files

- `logs/rotation-YYYY-MM-DD.log` - Daily rotation logs
- `logs/errors-YYYY-MM-DD.log` - Error logs
- `logs/compliance-tracking.log` - Compliance audit trail
- `logs/offline-systems-YYYY-MM-DD.log` - Unreachable systems

### Success Verification

Check rotation success:

```bash
# View latest rotation log
tail -f logs/rotation-$(date +%Y-%m-%d).log

# Check compliance tracking
cat logs/compliance-tracking.log

# View any offline systems
cat logs/offline-systems-$(date +%Y-%m-%d).log
```

## Compliance Features

### DISA STIG Compliance
- 90-day password rotation interval
- Complex password requirements
- Audit logging and tracking
- Secure credential storage with Ansible Vault


## Safety Features

### Pre-flight Checks
- System connectivity verification
- OS version validation
- SSH service status confirmation
- Password policy compliance

### Error Handling
- 3-retry mechanism for network issues
- Automatic password backup before rotation
- SSH service restart after rotation
- Graceful failure handling with logging

### Recovery Procedures
- Password hash backup in `/var/backups/credential-rotation/`
- Automatic restore on rotation failure
- Manual recovery instructions in logs

## Troubleshooting

### Common Issues

**Connection Failures:**
- Verify SSH key authentication
- Check network connectivity
- Confirm target systems are accessible

**Permission Errors:**
- Ensure running as root or with sudo
- Verify SSH keys have proper permissions
- Check target system SSH configuration

**Vault Issues:**
- Confirm vault password is correct
- Verify secrets.yml is properly encrypted
- Check vault file permissions

### Manual Recovery

If automated recovery fails:

```bash
# Connect to affected system
ssh root@affected-system.domain.com

# Restore from backup
sudo usermod -p "$(cat /var/backups/credential-rotation/password_hash_TIMESTAMP)" root

# Verify SSH service
sudo systemctl status ssh  # Ubuntu
sudo systemctl status sshd # RHEL
```

## Maintenance

### Regular Tasks
- Review rotation logs weekly
- Update inventory as systems change
- Rotate vault password quarterly
- Archive old logs annually

### Scheduled Rotation
Set up automated 90-day rotation:

```bash
# Add to crontab for quarterly rotation
0 2 1 */3 * cd /path/to/rotate-root-credentials && ansible-playbook rotate-creds.yml --vault-password-file ~/.ansible_vault_pass
```

## Security Considerations

- Store vault password securely (consider using a password manager)
- Regularly audit access to this repository
- Review logs for suspicious activity
- Maintain backup of current vault file
- Use dedicated service accounts for automation

## Support

For issues or questions:
1. Check the troubleshooting section
2. Review log files in the `logs/` directory
3. Verify system requirements and configuration
4. Test connectivity and permissions manually