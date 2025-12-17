# health_check_report

A role to generate comprehensive SCCM infrastructure health check reports with monitoring data and metrics.

## Outputs

When the state is present, this role outputs a variable `health_check_report_present_facts` with the following attributes:

- **report_path** - The full path to the generated HTML report file
- **report_name** - The name of the health check report  
- **site_code** - The SCCM site code that was analyzed
- **timestamp** - ISO8601 timestamp when the report was generated
- **metrics** - Dictionary containing health check metrics (collection counts, deployment counts, etc.)
- **email_sent** - Boolean indicating whether the report was sent via email
- **file_saved** - Boolean indicating whether the report was saved to file

## Dependencies

- `community.general` collection (required for email functionality when `health_check_report_send_email` is true)

## Role Variables

### General

- **health_check_report_site_code** (str, required)
    - The site code of the main SCCM/MECM site to analyze.

- **health_check_report_computer_name** (str, optional)
    - The computer name for WSUS sync status queries.
    - Defaults to `ansible_fqdn` (the fully qualified domain name of the target host).

- **health_check_report_state** (str, optional)
    - The state of the health check report. Either present or absent.
    - If present, the health report will be generated. If absent, the report file will be removed.
    - Default is `present`.

- **health_check_report_name** (str, optional)
    - The title/name of the health check report.
    - If this is not provided, a default name with a timestamp will be used. This name is dependent on when the role is included, so it is recommended to explicitly set this option.

- **health_check_report_output_directory** (str, optional)
    - The directory where the HTML report file will be generated.
    - Defaults to the system temp directory (Windows: `%TEMP%`).

- **health_check_report_filename** (str, optional)
    - The filename for the generated HTML report.
    - If this is not provided, a default filename with date will be used. This filename is dependent on when the role is included, so it is recommended to explicitly set this option.

- **health_check_report_remove_site_ps_drive** (bool, optional)
    - If true, the PS drive for the site will be removed when cleaning up resources.
    - Defaults to `true`.

### Delivery Options

- **health_check_report_save_file** (bool, optional)
    - If true, saves the HTML report to a file on the target system.
    - Defaults to `true`.

- **health_check_report_send_email** (bool, optional)
    - If true, sends the HTML report via email using community.general.mail module.
    - Defaults to `false`.

### Email Configuration (Required when health_check_report_send_email is true)

- **health_check_report_email_to** (list, required for email)
    - List of email addresses to send the report to.

- **health_check_report_email_from** (str, required for email)
    - Email address to send the report from.

- **health_check_report_email_subject** (str, optional)
    - Subject line for the email. 
    - Defaults to "SCCM Health Check Report - {{ date }}".

- **health_check_report_email_smtp_host** (str, required for email)
    - SMTP server hostname or IP address.

- **health_check_report_email_smtp_port** (int, optional)
    - SMTP server port number.
    - Defaults to `25`.

- **health_check_report_email_username** (str, optional)
    - Username for SMTP authentication (if required).

- **health_check_report_email_password** (str, optional)
    - Password for SMTP authentication (if required).

- **health_check_report_email_fail_on_error** (bool, optional)
    - If true, role execution fails when email sending fails. If false, shows warning and continues.
    - Defaults to `true`.

## Health Check Data Collected

The role gathers the following SCCM infrastructure information:

1. **Site Status Messages**
   - Component status messages with severity levels
   - Error, warning, and information messages from site systems

2. **Software Update Groups**
   - Update group information and content
   - Update group membership and configurations

3. **Software Update Deployments**
   - Deployment names, states, and target collections
   - Deployment scheduling and configuration

4. **Backup Status Information**
   - Backup task status and schedules
   - Last backup run results and next scheduled runs

5. **Distribution Point Status**
   - Package distribution status across distribution points
   - Content replication and availability status

6. **WSUS Sync Status**
   - Software update point synchronization status
   - Last sync times and error information

## Report Features

The generated HTML report includes:

- **Executive Summary** - High-level metrics and counts with visual cards
- **Site Status Messages** - Component health messages with severity indicators
- **Software Update Groups** - Update group details and configurations
- **Software Update Deployments** - Deployment status and configurations
- **Backup Status** - Backup task results and scheduling information
- **Distribution Point Status** - Package distribution and replication status
- **WSUS Sync Status** - Software update synchronization information
- **Visual Dashboard** - Metric cards with professional styling
- **Responsive Design** - Professional HTML layout optimized for viewing
- **Email Delivery** - Optional HTML email delivery via SMTP

## Examples

### Basic Health Check Report

```yaml
---
- name: Generate SCCM health check report
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"
```

### Custom Report Configuration

```yaml
---
- name: Generate custom SCCM health report
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"
      health_check_report_name: "Daily SCCM Infrastructure Health Report"
      health_check_report_output_directory: "C:\\Reports\\SCCM"
      health_check_report_filename: "sccm_health_{{ ansible_facts['date_time']['date'] }}.html"
```

### Generate and Clean Up Report

```yaml
---
- name: Generate health report
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"
      health_check_report_state: present

- name: Display report information
  hosts: mecmserver
  gather_facts: false
  tasks:
    - name: Show report details
      ansible.builtin.debug:
        msg:
          - "Health report generated: {{ health_check_report_present_facts.report_path }}"
          - "Total collections: {{ health_check_report_present_facts.metrics.total_collections }}"
          - "Total deployments: {{ health_check_report_present_facts.metrics.total_deployments }}"

- name: Remove health report
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"
      health_check_report_state: absent
```

### Scheduled Daily Health Check

```yaml
---
- name: Daily SCCM health check report
  hosts: mecmserver
  gather_facts: false
  vars:
    daily_report_directory: "C:\\Reports\\SCCM\\Daily"
  tasks:
    - name: Create report directory
      win_file:
        path: "{{ daily_report_directory }}"
        state: directory

    - name: Generate daily health report
      ansible.builtin.include_role:
        name: infra.mecm_ops.health_check_report
      vars:
        health_check_report_site_code: "PS1"
        health_check_report_name: "Daily SCCM Health Check - {{ ansible_facts['date_time']['date'] }}"
        health_check_report_output_directory: "{{ daily_report_directory }}"
        health_check_report_filename: "daily_health_{{ ansible_facts['date_time']['date'] }}.html"

    - name: Archive old reports (optional)
      win_shell: |
        Get-ChildItem "{{ daily_report_directory }}" -Name "*.html" | 
        Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } |
        Remove-Item -Force
      when: archive_old_reports | default(false)
```

### Email Health Report

```yaml
---
- name: Email SCCM health check report
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"
      health_check_report_send_email: true
      health_check_report_save_file: false
      health_check_report_email_to:
        - "sccm-admins@company.com"
        - "it-managers@company.com"
      health_check_report_email_from: "sccm-reports@company.com"
      health_check_report_email_subject: "Weekly SCCM Health Check Report"
      health_check_report_email_smtp_host: "smtp.company.com"
      health_check_report_email_smtp_port: 587
      health_check_report_email_username: "{{ smtp_username }}"
      health_check_report_email_password: "{{ smtp_password }}"
```

### Email and Save Report

```yaml
---
- name: Generate and email SCCM health report
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"
      health_check_report_send_email: true
      health_check_report_save_file: true
      health_check_report_name: "SCCM Infrastructure Health Report"
      health_check_report_output_directory: "C:\\Reports\\SCCM"
      health_check_report_email_to: ["sccm-team@company.com"]
      health_check_report_email_from: "noreply@company.com"
      health_check_report_email_smtp_host: "mail.company.com"

  post_tasks:
    - name: Show delivery status
      ansible.builtin.debug:
        msg:
          - "Report saved to file: {{ health_check_report_present_facts.file_saved }}"
          - "Report emailed: {{ health_check_report_present_facts.email_sent }}"
          - "Report location: {{ health_check_report_present_facts.report_path }}"
```

## Integration with Monitoring Systems

The role can be integrated with monitoring and alerting systems:

```yaml
---
- name: Health check with monitoring integration
  hosts: mecmserver
  gather_facts: false
  roles:
    - role: infra.mecm_ops.health_check_report
      health_check_report_site_code: "PS1"

  post_tasks:
    - name: Send report to monitoring system
      uri:
        url: "{{ monitoring_webhook_url }}"
        method: POST
        body_format: json
        body:
          report_path: "{{ health_check_report_present_facts.report_path }}"
          metrics: "{{ health_check_report_present_facts.metrics }}"
          timestamp: "{{ health_check_report_present_facts.timestamp }}"
          site_code: "{{ health_check_report_present_facts.site_code }}"
      when: monitoring_webhook_url is defined
```

## Requirements

- Ansible >= 2.16
- Microsoft MECM collection (`microsoft.mecm`)
- Windows target host with SCCM/MECM installed
- PowerShell execution policy allowing script execution
- SCCM administrative permissions

## Troubleshooting

### Common Issues

1. **Access Denied Errors**
   - Ensure the executing user has SCCM administrative permissions
   - Verify PowerShell execution policy allows script execution

2. **Module Not Found**
   - Ensure the `microsoft.mecm` collection is installed
   - Verify SCCM PowerShell modules are available on the target system

3. **Report Generation Failures**
   - Check that the output directory exists and is writable
   - Verify sufficient disk space for report generation

4. **Empty Report Data**
   - Confirm the site code is correct and accessible
   - Check SCCM service status and database connectivity

## License

GNU General Public License v3.0 or later

See [LICENSE](https://github.com/ansible-collections/infra.mecm_ops/blob/main/LICENSE) to see the full text.

## Author Information

- Ansible Cloud Content Team