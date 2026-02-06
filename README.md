# Intersight Ansible automation for fleet management

## Variable Configuration

This application uses four variable files located in the `vars/` directory to configure Intersight fleet management deployments.
This guide documents all available variables and their usage.
Splitting up variables into files is optional, but helps organize your inputs.

| File | Purpose |
|------|---------|
| `vars/auth.yaml` | Intersight API authentication credentials and connection settings |
| `vars/basics.yaml` | Base configuration settings applied across all deployments |
| `vars/definitions.yaml` | Workload definitions, OS images, network settings, and firmware versions |
| `vars/deployments.yaml` | Organizations, maintenance schedules, and deployment configurations |

---

## vars/auth.yaml

### API Authentication Variables

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `api_private_key` | string | Yes | Path to your Intersight API private key file | `"./vars/SecretKey.txt"` |
| `api_key_id` | string | Yes | Your Intersight API key ID (found in Intersight UI) | `"68b6ebaa75646133018f474a/..."` |
| `api_uri` | string | Yes | Intersight API endpoint URL | `"https://intersight.com/api/v1"` |
| `validate_certs` | boolean | Yes | Enable/disable SSL certificate validation | `true` |

---

## vars/basics.yaml

### Basic Configuration Variables

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `base_name` | string | Yes | Prefix used for all created objects in Intersight | `"DEVNET_2290"` |
| `base_org` | string | Yes | Name of the base organization in Intersight | `"Base_Org"` |

---

## vars/definitions.yaml

This file contains workload definitions and shared configuration settings for chassis and servers.

### Workload Definitions

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `definitions` | list | Yes | List of workload definitions (Small, Medium, Large, etc.) |

**Definition Structure:**
```yaml
definitions:
  - name: Small                    # Unique name for this definition
    chassis_overrides: {}          # Optional chassis-level overrides
    blueprints:                    # List of blueprints to deploy
      - type: ubuntu               # Blueprint type (ubuntu, rhel, windows)
        slot_id: 1                 # Server slot number (1-4)
        overrides:                 # Optional blueprint-specific overrides
          Name: custom_name        # Custom workload name
```

### OS Image Configuration

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `scu_image` | string | Yes | Moid of the Server Configuration Utility image | `"693723906567613201340c5e"` |
| `ubuntu_image` | string | Yes | Moid of the Ubuntu OS image | `"698269a265676131015c9531"` |
| `rhel_image` | string | No | Moid of the Red Hat Enterprise Linux image | `""` |
| `windows_image` | string | No | Moid of the Windows Server image | `""` |
| `windows_key` | string | No* | Windows product key | `"XNFW3-39KQC-HV6XQ-..."` |
| `windows_edition` | string | No* | Windows edition name | `"Datacenter"` |

### OS Configuration

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `password` | string | Yes | Default password for OS installations | `"devnet_2290!"` |
| `os_vlans` | string | Yes | VLAN ID(s) for OS network configuration | `"5"` |

### Time & DNS Configuration

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `timezone` | string | Yes | Default timezone for deployments | `"Europe/Vienna"` |
| `ntp_servers` | list | Yes | List of NTP servers for time synchronization | `["pool.ntp.org"]` |
| `dns_servers` | list | Yes | List of DNS servers | `["1.1.1.1", "8.8.8.8"]` |

### IP Pool Configuration

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `ip_range` | list | Yes | IP address ranges for management network | See below |
| `netmask` | string | Yes | Subnet mask for management network | `"255.255.255.0"` |
| `gateway` | string | Yes | Default gateway for management network | `"192.168.0.1"` |

**IP Range Structure:**
```yaml
ip_range:
  - from: "192.168.0.2"    # Starting IP address
    size: 16               # Number of consecutive IPs to allocate
```

### Local User Accounts

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `local_users` | list | Yes | Local user accounts for server access |

**Local User Structure:**
```yaml
local_users:
  - username: admin        # Username for login
    role: admin            # User role (admin, user, readonly)
    password: "my_pass!"   # User password
```

### Unified Edge Chassis Configuration

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `chassis_firmware` | string | Yes | Target firmware version for chassis | `"6.0(1.251006)"` |
| `uplink_ports` | string | Yes | Uplink port configuration | `"1"` |
| `jumbo_frames` | boolean | Yes | Enable/disable jumbo frames | `true` |
| `mgmt_vlan` | integer | Yes | Management VLAN ID | `5` |
| `allowed_vlans` | list | Yes | List of allowed VLANs on chassis | See below |

**Allowed VLAN Structure:**
```yaml
allowed_vlans:
  - AutoAllowOnUplinks: true    # Auto-configure on uplinks
    VlanId: 5                   # VLAN ID number
    VlanName: vlan_5            # VLAN name
```

### Unified Edge Server Configuration

| Variable | Type | Required | Description | Example |
|----------|------|----------|-------------|---------|
| `server_firmware` | string | Yes | Target firmware version for servers | `"6.0(1.251030)"` |

---

## vars/deployments.yaml

This file defines organizations, maintenance schedules, and deployment configurations.

### Organizations

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `organizations` | list | Yes | List of consumer organizations |

**Organization Structure:**
```yaml
organizations:
  - name: UK_I                    # Organization name
    chassis:                      # List of chassis serial numbers
      - 'WZP29XXXXXX'
      - 'WZP29XXXXXX'
```

### Maintenance Schedules

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `maintenance` | list | Yes | List of maintenance policies |

**Maintenance Policy Structure:**
```yaml
maintenance:
  - name: UK&I                           # Policy name
    org: UK_I                            # Associated organization
    schedules:                           # List of maintenance windows
      - Name: DailyUpdateWindow          # Schedule name
        StartTime: '2025-12-01T22:00:00+01:00'  # ISO 8601 start time
        Duration: PT6H                   # Duration (ISO 8601 format)
        Cadence: Weekly                  # Daily, Weekly, Monthly
        Params:                          # Cadence-specific parameters
          WeeklyRunEvery: 1              # Run every N weeks
          DayOfWeek:                     # Days of week
            - Monday
            - Tuesday
          ObjectType: scheduler.WeeklyCadenceParams
          RunEvery: 1
        EndOption: Never                 # Never, AfterOccurrences, OnDate
        FailureThreshold: 2              # Max consecutive failures
        EndTime: '2099-09-09T00:00:00Z' # End date (for Never, use far future)
        ObjectType: scheduler.RecurringScheduleParams
        TimeZone: "Europe/London"        # IANA timezone name
    enable_blocked_dates: true           # Enable maintenance blackouts
    block_dates:                         # Blocked maintenance periods
      - StartTime: '2026-12-17T16:00:00Z'
        EndTime: '2027-01-04T10:00:00Z'
```

### Deployment Groups

| Variable | Type | Required | Description |
|----------|------|----------|-------------|
| `deployment_groups` | list | Yes | List of deployment configurations |

**Deployment Group Structure:**
```yaml
deployment_groups:
  - name: UK                             # Deployment group name
    org: UK_I                            # Target organization (or 'Base')
    maintenance: "UK&I"                  # Associated maintenance policy
    version: "preferred"                 # "preferred" or specific version number
    blueprint_overrides:                 # Optional group-level overrides
      - Name: UnifiedEdgeChassis         # Blueprint name
        Blueprint:
          Moid: 68dc8a036e75733101ab7068  # Blueprint Moid
        Input:                           # Override input parameters
          Timezone: "Europe/London"
          NtpConfiguration:
            NtpServers:
              - "uk.pool.ntp.org"
          Dns:
            PrimaryDnsServer: "209.250.227.42"
            SecondaryDnsServer: "64.176.190.82"
    deployments:                         # List of individual deployments
      - definition: Small                # Reference to definition name
        qualifiers: "/api/v1/equipment/Chasses?$filter=(Tags.Key eq 'EMEA/UK/Small')"
      - definition: Large
        version: "1"                     # Override group version
        qualifiers: "/api/v1/equipment/Chasses?$filter=(Tags.Key eq 'EMEA/UK/Large')"
        blueprint_overrides:             # Override group-level overrides
          - Name: UnifiedEdgeChassis
            Blueprint:
              Moid: 68dc8a036e75733101ab7068
            Input:
              Dns:
                PrimaryDnsServer: "1.1.1.1"
```