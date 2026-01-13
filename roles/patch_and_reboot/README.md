# patch_and_reboot

An Ansible role that orchestrates end-to-end Windows patching and reboot operations across multiple target hosts with comprehensive maintenance reporting.

## Overview

This role streamlines maintenance-window execution by providing a reusable, standardized automation pattern for patch + reboot cycles across distributed Windows server fleets. It executes the full patching workflow on each host (Windows Update install, validation, reboot when required) and consolidates all operational actions and outcomes into a comprehensive maintenance report.

## Outputs

When the state is present, this role outputs a variable `patch_and_reboot_present_facts` with the following attributes:

- **report_path** - The full path to the generated HTML maintenance report
- **report_name** - The name of the patch and reboot operation
- **target_hosts** - List of hosts that were targeted for patching
- **timestamp** - ISO8601 timestamp when the operation completed
- **metrics** - Dictionary containing operation metrics (total hosts, success/failure counts, timing)
- **email_sent** - Boolean indicating whether the report was sent via email
- **file_saved** - Boolean indicating whether the report was saved to file
- **operation_summary** - Summary statistics of the MECM patching operation

## Dependencies

- `microsoft.mecm` collection (required for MECM operations)
- `community.general` collection (required for email functionality when `patch_and_reboot_send_email` is true)
- `ansible.windows` collection (required for Windows operations)

## Role Variables

### General


- **patch_and_reboot_state** (str, required)
    - The state of the patch and reboot operation. Either present or absent.
    - If present, the patching operation will be executed. If absent, report files will be removed.

- **patch_and_reboot_name** (str, optional)
    - The title/name of the patching operation for reporting.
    - If not provided, a default name with timestamp will be used.

- **patch_and_reboot_output_directory** (str, optional)
    - The directory where the HTML maintenance report will be generated.
    - Defaults to the system temp directory (`C:\Windows\Temp`).

- **patch_and_reboot_filename** (str, optional)
    - The filename for the generated HTML report.
    - If not provided, a default filename with date will be used.

### Patching Configuration

- **patch_and_reboot_update_categories** (list, optional)
    - List of MECM update categories to install.
    - Defaults to `['Critical', 'Security', 'UpdateRollups']`.
    - Available categories: Critical, Security, UpdateRollups, FeaturePacks, ServicePacks, Tools, Updates

- **patch_and_reboot_update_ids** (list, optional)
    - List of specific update KB IDs to install (e.g. ['KB5000001', 'KB5000002']).
    - When specified, only these specific updates will be installed.

- **patch_and_reboot_reboot_timeout** (int, optional)
    - Maximum time to wait for system reboot to complete (in seconds).
    - Defaults to `600` (10 minutes).

- **patch_and_reboot_post_reboot_delay** (int, optional)
    - Time to wait after reboot before continuing (in seconds).
    - Defaults to `60` seconds.

- **patch_and_reboot_max_install_time** (int, optional)
    - Maximum time to allow for update installation (in seconds).
    - Defaults to `3600` (1 hour).

### Validation Configuration

- **patch_and_reboot_validate_services** (bool, optional)
    - If true, validates that critical Windows services are running after patching.
    - Defaults to `true`.

- **patch_and_reboot_validate_connectivity** (bool, optional)
    - If true, validates host connectivity after patching and reboot.
    - Defaults to `true`.

- **patch_and_reboot_validation_timeout** (int, optional)
    - Maximum time to wait for post-patch validation (in seconds).
    - Defaults to `300` (5 minutes).

### Delivery Options

- **patch_and_reboot_save_file** (bool, optional)
    - If true, saves the HTML maintenance report to a file.
    - Defaults to `true`.

- **patch_and_reboot_send_email** (bool, optional)
    - If true, sends the maintenance report via email.
    - Defaults to `false`.

### Email Configuration (Required when patch_and_reboot_send_email is true)

- **patch_and_reboot_email_to** (list, required for email)
    - List of email addresses to send the report to.

- **patch_and_reboot_email_from** (str, required for email)
    - Email address to send the report from.

- **patch_and_reboot_email_subject** (str, optional)
    - Subject line for the email.
    - Defaults to "Windows Patch and Reboot Report - {{ date }}".

- **patch_and_reboot_email_smtp_host** (str, required for email)
    - SMTP server hostname or IP address.

- **patch_and_reboot_email_smtp_port** (int, optional)
    - SMTP server port number.
    - Defaults to `25`.

- **patch_and_reboot_email_username** (str, optional)
    - Username for SMTP authentication (if required).

- **patch_and_reboot_email_password** (str, optional)
    - Password for SMTP authentication (if required).

## Operation Workflow

The role executes the following workflow for each target host:

1. **Connectivity Check** - Verifies the host is accessible
2. **Pre-patch Information Gathering** - Collects system state before patching
3. **Update Search** - Searches for available updates in specified categories
4. **Update Installation** - Installs available updates (with timeout protection)
5. **Reboot (if required)** - Performs system reboot if updates require it
6. **Post-patch Validation** - Validates system health after patching
7. **Results Tracking** - Records operation results and metrics

## Report Features

The generated HTML maintenance report includes:

- **Executive Summary** - High-level metrics showing total/successful/failed hosts
- **Operation Summary** - Table showing status, update count, and reboot status for each host
- **Detailed Operation Log** - Complete timeline and details for all operations
- **Failed Operations** - Specific error details for any failed hosts
- **Successful Operations** - Update installation details for successful hosts
- **Professional Styling** - Clean, readable HTML layout optimized for management reporting

## Examples

### Basic MECM Patching Operation

```yaml
---
- name: Execute MECM patching maintenance
  hosts: windows_servers
  gather_facts: true
  roles:
    - role: infra.mecm_ops.patch_and_reboot
```

### Security Updates Only

```yaml
---
- name: MECM security patching maintenance
  hosts: web_servers
  gather_facts: true
  roles:
    - role: infra.mecm_ops.patch_and_reboot
      patch_and_reboot_name: "Monthly Security Patching - Web Servers"
      patch_and_reboot_update_categories:
        - Security
      patch_and_reboot_reboot_timeout: 900
```

### MECM Maintenance Window with Email Reporting

```yaml
---
- name: Monthly MECM maintenance window
  hosts: production_servers
  gather_facts: true
  roles:
    - role: infra.mecm_ops.patch_and_reboot
      patch_and_reboot_name: "Monthly Maintenance Window - Production Servers"
      patch_and_reboot_send_email: true
      patch_and_reboot_email_to:
        - "it-team@company.com"
        - "managers@company.com"
      patch_and_reboot_email_from: "maintenance@company.com"
      patch_and_reboot_email_subject: "Production Maintenance Completed"
      patch_and_reboot_email_smtp_host: "smtp.company.com"
```

### Custom Validation and Timing

```yaml
---
- name: Critical server MECM patching with extended timeouts
  hosts: critical_servers
  gather_facts: true
  roles:
    - role: infra.mecm_ops.patch_and_reboot
      patch_and_reboot_name: "Critical Server Maintenance"
      patch_and_reboot_max_install_time: 7200  # 2 hours
      patch_and_reboot_reboot_timeout: 1200    # 20 minutes
      patch_and_reboot_post_reboot_delay: 120  # 2 minutes
      patch_and_reboot_validation_timeout: 600 # 10 minutes
      patch_and_reboot_validate_services: true
      patch_and_reboot_validate_connectivity: true
```

### Emergency MECM Patching with Specific Updates

```yaml
---
- name: Emergency MECM security patching
  hosts: windows_servers
  gather_facts: true
  roles:
    - role: infra.mecm_ops.patch_and_reboot
      patch_and_reboot_name: "Emergency Security Patching - CVE Response"
      patch_and_reboot_update_categories:
        - Critical
        - Security
      patch_and_reboot_update_ids:
        - "KB5000001"
        - "KB5000002"
      patch_and_reboot_max_install_time: 1800  # 30 minutes
      patch_and_reboot_output_directory: "C:\\EmergencyMaintenance"
```

### Cleanup Previous Reports

```yaml
---
- name: Clean up old maintenance reports
  hosts: windows_servers
  gather_facts: false
  roles:
    - role: infra.mecm_ops.patch_and_reboot
      patch_and_reboot_state: absent
      patch_and_reboot_output_directory: "C:\\Maintenance\\Reports"
      patch_and_reboot_filename: "old_maintenance_report.html"
```

### Integration with Monitoring Systems

```yaml
---
- name: Automated MECM patching with monitoring integration
  hosts: windows_servers
  gather_facts: true
  roles:
    - role: infra.mecm_ops.patch_and_reboot
      patch_and_reboot_name: "Automated Monthly MECM Patching"

  post_tasks:
    - name: Send results to monitoring system
      uri:
        url: "{{ monitoring_webhook_url }}"
        method: POST
        body_format: json
        body:
          operation: "windows_patching"
          timestamp: "{{ patch_and_reboot_present_facts.timestamp }}"
          total_hosts: "{{ patch_and_reboot_present_facts.operation_summary.total_hosts }}"
          successful_hosts: "{{ patch_and_reboot_present_facts.operation_summary.successful_hosts }}"
          failed_hosts: "{{ patch_and_reboot_present_facts.operation_summary.failed_hosts }}"
          report_path: "{{ patch_and_reboot_present_facts.report_path }}"
      when: monitoring_webhook_url is defined

    - name: Create ServiceNow ticket for failures
      servicenow.servicenow.snow_record:
        instance: "{{ snow_instance }}"
        username: "{{ snow_username }}"
        password: "{{ snow_password }}"
        table: incident
        data:
          short_description: "Patching failures detected"
          description: "{{ patch_and_reboot_present_facts.operation_summary.failed_hosts }} hosts failed during patching"
          priority: 2
      when: 
        - patch_and_reboot_present_facts.operation_summary.failed_hosts | int > 0
        - snow_instance is defined
```

## Operational Considerations

### Pre-requisites
- All target hosts must be Windows systems
- Target hosts must be accessible via WinRM
- Ansible user must have administrator privileges on target systems
- Windows Update service must be running on target systems

### Timing and Scheduling
- Plan adequate maintenance windows based on `patch_and_reboot_max_install_time`
- Consider `patch_and_reboot_reboot_timeout` for systems with slow boot times
- Factor in `patch_and_reboot_post_reboot_delay` for application startup times

### Error Handling
- Operations continue on failure (other hosts are still processed)
- Failed operations are tracked and reported separately
- Use block/rescue patterns for additional error handling

### Performance
- Operations are executed sequentially across hosts (not parallel)
- Use `async` and `poll` for long-running update installations
- Monitor system resources during patching operations

## Requirements

- Ansible >= 2.16
- Windows target hosts with WinRM configured
- Administrator privileges on target systems
- PowerShell execution policy allowing script execution
- Network connectivity between Ansible controller and target hosts

## Troubleshooting

### Common Issues

1. **Connection Failures**
   - Verify WinRM is configured and accessible on target hosts
   - Check firewall rules allowing WinRM traffic
   - Validate credentials and permissions

2. **Update Installation Timeouts**
   - Increase `patch_and_reboot_max_install_time` for slow systems
   - Check available disk space on target systems
   - Verify Windows Update service is running

3. **Reboot Failures**
   - Increase `patch_and_reboot_reboot_timeout` for slow-booting systems
   - Check for stuck processes preventing clean shutdown
   - Verify system hardware health

4. **Validation Failures**
   - Increase `patch_and_reboot_validation_timeout`
   - Check critical service dependencies
   - Verify network connectivity post-reboot


## Security Considerations

- Use encrypted connections (WinRM over HTTPS)
- Store credentials securely (Ansible Vault)
- Limit update categories to required patches only
- Test patches in non-production environments first
- Monitor patch installation for unexpected behavior

## License

GNU General Public License v3.0 or later

See [LICENSE](https://github.com/redhat-cop/infra.mecm_ops/blob/main/LICENSE) to see the full text.

## Author Information

- Ansible Cloud Content Team