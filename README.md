```markdown
# Solana Validator Monitoring and Failover System

Ansible-based system for monitoring Solana validators and managing automatic failover between main and backup validators.

## Features

### Monitoring
- Real-time monitoring of validator delinquent status
- SSH connectivity checks for both validators
- Automatic tower file backups
- Detailed logging of system state

### Automatic Failover
Handles different scenarios of validator failures:
- Service failure (validator process not running)
- Server unavailability (server down/unreachable)
- Node lag (validator fell behind and marked delinquent)

### Manual Rollback
Provides safe manual rollback functionality when returning to main validator.

## Requirements

- Ansible 2.10+
- Solana CLI tools installed on Sentry server
- SSH access to both validators
- Proper validator setup with identity keys

## Installation

1. Clone the repository:
```bash
git clone https://github.com/your-repo/solana-validator-monitor.git
cd solana-validator-monitor
```

2. Configure variables in `group_vars/all.yml`:
```yaml
# Main Validator
main_validator_ip: "YOUR_MAIN_IP"
main_validator_ssh_port: YOUR_SSH_PORT
main_validator_identity_key: "YOUR_VALIDATOR_PUBKEY"

# Backup Validator
backup_validator_ip: "YOUR_BACKUP_IP"
backup_validator_ssh_port: YOUR_SSH_PORT
```

3. Place SSH keys in `files/` directory:
```bash
cp /path/to/your/ssh/key files/validator_key
chmod 600 files/validator_key
```

4. Initialize the system:
```bash
ansible-playbook playbooks/init.yml
```

## Usage

### Regular Monitoring
```bash
ansible-playbook playbooks/monitor.yml
```
Add to crontab for continuous monitoring:
```bash
*/5 * * * * cd /path/to/project && ansible-playbook playbooks/monitor.yml
```

### Manual Rollback
When you want to return to main validator:
```bash
ansible-playbook playbooks/rollback.yml
```

## System States

### Active States
System tracks active validator in `/var/run/solana_validator/active_validator`:
- "main" - Main validator is active
- "backup" - Backup validator is active

### Failover Process
1. Detects main validator delinquency
2. Changes main validator's identity to unstaked
3. Transfers tower file to backup validator
4. Activates backup validator with staked identity

### Rollback Process (Manual)
1. Verifies backup validator is currently active
2. Checks main validator accessibility
3. Transfers tower file back to main validator
4. Restores proper identity keys
5. Updates system state

## File Structure
```
.
├── inventory/
│   └── hosts.yml
├── group_vars/
│   └── all.yml
├── roles/
│   ├── monitoring/
│   ├── backup/
│   ├── failover/
│   ├── rollback/
│   └── ssh_setup/
├── playbooks/
│   ├── init.yml
│   ├── monitor.yml
│   └── rollback.yml
└── files/
    └── validator_key
```

## Logging

System maintains logs in `/var/run/solana_validator/`:
- `last_check.log` - Latest monitoring status
- `failover.status` - Failover event records
- `active_validator` - Current active validator

## Safety Features

- SSH connectivity verification
- Proper identity key management
- Prevents duplicate active validators
- Safe tower file transfers
- Multiple failure scenario handling

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request
