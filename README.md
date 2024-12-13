# Solana Validator Monitor

Ansible-based monitoring and failover system for Solana validators. This system provides automated monitoring of validator status and handles failover between main and backup validators.

## Features

- Real-time monitoring of validator status
- Automated failover on delinquency detection
- Secure tower file synchronization
- SSH-based secure communications
- Detailed logging and status tracking
- Automated identity management for validators

## Prerequisites

- Ansible 2.9+
- Solana CLI tools installed on the Sentry server
- SSH access to both validators
- Proper validator setup with identity keys

## Project Structure

```
.
├── inventory/
│   └── hosts.yml
├── group_vars/
│   ├── all.yml
│   └── all.yml.example
├── roles/
│   ├── monitoring/
│   ├── backup/
│   ├── failover/
│   └── ssh_setup/
├── playbooks/
│   ├── init.yml
│   └── monitor.yml
└── files/
    └── .gitkeep
```

## Quick Start

1. Clone the repository:
```bash
git clone https://github.com/yourusername/solana-validator-monitor.git
cd solana-validator-monitor
```

2. Copy and configure variables:
```bash
cp group_vars/all.yml.example group_vars/all.yml
# Edit group_vars/all.yml with your configuration
```

3. Set up SSH keys:
```bash
# Copy your SSH keys to the files directory
cp /path/to/your/ssh/keys files/
```

4. Initialize the system:
```bash
ansible-playbook playbooks/init.yml
```

5. Start monitoring:
```bash
ansible-playbook playbooks/monitor.yml
```

## Security Considerations

### Managing Secrets

1. Use Ansible Vault for sensitive data:
```bash
# Create encrypted variables file
ansible-vault create group_vars/vault.yml

# Edit encrypted file
ansible-vault edit group_vars/vault.yml

# Run playbook with vault password
ansible-playbook playbooks/monitor.yml --ask-vault-pass
```

2. Configure these sensitive values in vault.yml:
```yaml
vault_main_validator_ssh_key: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  your_key_here
  -----END OPENSSH PRIVATE KEY-----
vault_backup_validator_ssh_key: |
  -----BEGIN OPENSSH PRIVATE KEY-----
  your_key_here
  -----END OPENSSH PRIVATE KEY-----
vault_main_validator_identity_key: "your-validator-pubkey"
```

3. Reference vault variables in all.yml:
```yaml
main_validator_ssh_key: "{{ vault_main_validator_ssh_key }}"
backup_validator_ssh_key: "{{ vault_backup_validator_ssh_key }}"
main_validator_identity_key: "{{ vault_main_validator_identity_key }}"
```

### Best Practices

1. Never commit unencrypted secrets to git
2. Use separate SSH keys for each validator
3. Restrict SSH key permissions:
```bash
chmod 600 files/*_key
```
4. Use dedicated user accounts on each server
5. Implement proper firewall rules between servers

### Automated Deployment

For automated deployment, create a vault password file:
```bash
echo "your-vault-password" > ~/.vault_pass
chmod 600 ~/.vault_pass
```

Add to ansible.cfg:
```ini
[defaults]
vault_password_file = ~/.vault_pass
```

## Monitoring Setup

Add to crontab for continuous monitoring:
```bash
*/5 * * * * cd /path/to/project && ansible-playbook playbooks/monitor.yml
```

## Logs and Status

Monitoring logs are stored in:
```
/var/run/solana_validator/last_check.log
```

System status is maintained in:
```
/var/run/solana_validator/active_validator
/var/run/solana_validator/failover.status
```

## Manual Failover

To manually trigger failover:
```bash
ansible-playbook playbooks/monitor.yml --extra-vars "force_failover=true"
```

## Troubleshooting

1. Check SSH connectivity:
```bash
ansible-playbook playbooks/monitor.yml --tags ssh_check
```

2. Verify validator status:
```bash
solana validators --output json | jq '.validators[] | select(.identityPubkey=="YOUR_KEY")'
```

3. Check logs:
```bash
tail -f /var/run/solana_validator/last_check.log
```

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request

## License

MIT