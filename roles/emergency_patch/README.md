# emergency_patch

A role to create or delete temporary resources for an emergency patch deployment.

## Dependencies

N/A

## Role Variables

### General

- **emergency_patch_site_code** (str, required)
    - The site code of the main SCCM/MECM site.

- **emergency_patch_state** (str, required)
    - The state of the emergency patch resources. Either present or absent.
    - If present, the patch deployment will be created. If absent, the deployment will be removed.

### Software Update Group

- **emergency_patch_software_update_group_name** (str, optional)
    - The name of the software update group to manage.
    - If this is not provided, a default name with a timestamp will be used. This name is dependent on
      when the role is included, so it is recommended to explicitly set this option.

- **emergency_patch_software_update_group_description** (str, optional)
    - The description for the temporary software update group.

- **emergency_patch_software_update_ids** (list(str), optional)
    - A list of software update IDs to include in the update deployment.
    - These IDs are the CI IDs from the update properties in SCCM/MECM.
    - Either `emergency_patch_software_update_ids` or `emergency_patch_software_update_kbs` is required when
      `emergency_patch_state` is present

- **emergency_patch_software_update_kbs** (list(str), optional)
    - A list of software update KB numbers to include in the update deployment.
    - Either `emergency_patch_software_update_ids` or `emergency_patch_software_update_kbs` is required when
      `emergency_patch_state` is present


### Deployment

- **emergency_patch_deployment_name** (str, optional)
    - The name of the software update group to manage.
    - If this is not provided, a default name with a timestamp will be used. This name is dependent on
      when the role is included, so it is recommended to explicitly set this option.

- **emergency_patch_deployment_description** (str, optional)
    - The description for the temporary software update group.

- **emergency_patch_collection_name** (str, optional)
    - The name of the device collection to target for the deployment.
    - Either `emergency_patch_collection_name` or `emergency_patch_collection_id` is required when
      `emergency_patch_state` is present

- **emergency_patch_collection_id** (str, optional)
    - The CI ID of the device collection to target for the deployment.
    - Either `emergency_patch_collection_name` or `emergency_patch_collection_id` is required when
      `emergency_patch_state` is present

- **emergency_patch_deployment_enable_soft_deadline** (bool, optional)
    - Enable soft deadline for the deployment.
    - Defaults to `false`.

- **emergency_patch_deployment_allow_installation_outside_maintenance_window** (bool, optional)
    - Allow installation of updates outside the configured maintenance window.
    - Defaults to `true`.

- **emergency_patch_deployment_allow_restarts** (bool, optional)
    - Allow system restarts during the deployment if required by the updates.

- **emergency_patch_deployment_allow_metered_network_downloads** (bool, optional)
    - Allow downloads over metered network connections.

- **emergency_patch_deployment_allow_remote_distribution_point_downloads** (bool, optional)
    - Allow downloads from remote distribution points.

- **emergency_patch_deployment_allow_default_distribution_point_downloads** (bool, optional)
    - Allow downloads from default distribution points.

- **emergency_patch_deployment_disable_operations_manager_alerts** (bool, optional)
    - Disable Operations Manager alerts for this deployment.

- **emergency_patch_deployment_generate_operations_manager_alert_on_failure** (bool, optional)
    - Generate Operations Manager alert when the deployment fails.

- **emergency_patch_deployment_generate_success_threshold_alert** (bool, optional)
    - Generate an alert when the success threshold is reached.

- **emergency_patch_deployment_success_threshold** (int, optional)
    - The success threshold percentage for the deployment.

- **emergency_patch_deployment_allow_microsoft_update_downloads** (bool, optional)
    - Allow downloads directly from Microsoft Update if content is not available on distribution points.

- **emergency_patch_deployment_allow_branch_cache_downloads** (bool, optional)
    - Allow downloads using BranchCache.

- **emergency_patch_deployment_restart_servers_if_needed** (bool, optional)
    - Allow automatic restart of servers if required by the updates.

- **emergency_patch_deployment_restart_workstations_if_needed** (bool, optional)
    - Allow automatic restart of workstations if required by the updates.

- **emergency_patch_deployment_send_wake_up_packet** (bool, optional)
    - Send wake-up packets to wake sleeping devices for the deployment.

- **emergency_patch_deployment_deployment_timezone** (str, optional)
    - The timezone to use for deployment scheduling.

- **emergency_patch_deployment_user_notification_method** (str, optional)
    - The method for notifying users about the deployment.

- **emergency_patch_deployment_verbosity** (str, optional)
    - The verbosity level for deployment reporting.

- **emergency_patch_deployment_distribute_content** (bool, optional)
    - Whether to distribute content to distribution points.

- **emergency_patch_deployment_distribution_collection_name** (str, optional)
    - The name of the collection to use for content distribution.

- **emergency_patch_deployment_distribution_point_group_name** (str, optional)
    - The name of the distribution point group to use for content distribution.

- **emergency_patch_deployment_distribution_point_name** (str, optional)
    - The name of the specific distribution point to use for content distribution.

## Examples

```yaml
---
- name: Create emergency patch deployment
  hosts: mecmserver
  gather_facts: false

  roles:
    - role: infra.mecm.emergency_patch


- name: Install and wait for patch to complete
  hosts: all
  gather_facts: false
  tasks:
    - name: Install patch
      microsoft.mecm.install_updates:
        site_code: "{{ emergency_patch_site_code }}"
        wait_for_completion: true
        timeout_minutes: "{{ emergency_patch_timeout_minutes | default(omit) }}"
        allow_reboot: "{{ emergency_patch_deployment_allow_restarts | default(omit) }}"


- name: Delete emergency patch deployment
  hosts: mecmserver
  gather_facts: false

  roles:
    - role: infra.mecm.emergency_patch
      emergency_patch_state: absent
```

## License

GNU General Public License v3.0 or later

See [LICENSE](https://github.com/ansible-collections/infra.mecm_ops/blob/main/LICENSE) to see the full text.

## Author Information

- Ansible Cloud Content Team
