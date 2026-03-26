# TrueNAS-Setup

- truenas_admin -> Two factor authentication -> Configure 2FA Secret

- Storage -> Create Pool
    - General Info -> Name -> NAS01
    - General Info -> Check "Encryption" -> AES-256-GCM
    - Data -> Layout -> Choose the appropriate RAIDZ config

- Dataset -> NAS01 -> Dataset Details -> Edit -> Advanced Options
    - Sync -> Always
    - Compression level -> ZSTD
    - Checksum -> Blake 3
    - Exec -> Off

- Storage -> NAS01 -> Schedule Scrub -> At 00:30, only on Sunday, 30 Threadshold Days

- Dataset -> Add "Home" child dataset -> Set Quota to something low like 10 GiB

- Set default user homedir quota: `zfs set defaultuserquota=1g NAS01/Home`

- Data Protection
   - Periodic Snapshot Task -> Daily at 00:15, recursive, 2 weeks lifetime

- Network
    - Uncheck all service announcements

- Credentials -> Users -> root
    - Allow SSH access
    - Add email
    - Add NGINX's SSH public key

- Credentials -> Directory Services -> Add LDAP
    - Check "Enable Service"
    - Uncheck "Enable Account Cache"
    - Uncheck "Enable DNS Updates"
    - Credential Type -> LDAP Plain
    - Bind DN -> `cn=TrueNAS,ou=users,dc=ldap,dc=goauthentik,dc=io`
    - Server URLs -> `ldap://192.168.0.106`
    - Base DN -> `dc=ldap,dc=goauthentik,dc=io`
    - Schema -> RFC2307BIS
    - Uncheck "Use Standard Auxiliary Parameters"
        - Auxiliary Parameters

      ```
      chpass_provider = ldap
      access_provider = ldap
      ldap_group_name = cn
      ldap_access_order = filter
      ldap_access_filter = memberOf=cn=metropolis_storage,ou=groups,dc=ldap,dc=goauthentik,dc=io
      override_homedir = /mnt/NAS01/Home/%u
      ```
    
    - Uncheck "Use Standard Search Bases"
        - User Base DN -> `ou=users,dc=ldap,dc=goauthentik,dc=io`
        - Group Base DN -> `dc=ldap,dc=goauthentik,dc=io`
    - Uncheck "Use Standard Attribute Maps"
        - User Object Class -> user
        - Username Attribute -> cn
        - UID Attribute -> uidNumber
        - GID Attribute -> gidNumber
        - Last Change Attribute -> pwdChangedTime
        - Group Object Class -> group
        - Group GID Attribute -> gidNumber
        - Group Member Attribute -> member

- Credentials -> Certificates
    - Manually import certificates from NGINX reverse proxy VM

- System -> General -> GUI
    - GUI SSL Certificate -> Choose the certificate uploaded earlier
    - HTTPS Protocols -> TLSv1.3
    - Check "Web Interface HTTP -> HTTPS Redirect"
    - Uncheck "Usage Collection"

- System -> General -> Email
    - Configure SMTP notications

- System -> Advanced Settings -> Console
    - Uncheck "Show Text Console without Password Prompt" (Necessary as we have GPU passthrough and do not want the text console be available without authentication through the HDMI or Display ports)
    - Check "Enable Serial Console"
    - Serial Speed -> 115200
    - Access -> Set session timeout to 14400 (4 hours)
    - Global Two Factor Authentication -> Enable Two Factor Authentication Globally

- System -> Advanced Settings -> Cron Jobs **(This is a temporary hack job to deal with the home directories being set to 755 when override_homedir is used with SSSD)**
    - Description: Fix home directory permissions
    - Command: `for HOMEDIR in /mnt/NAS01/Home/*; do chmod 700 "${HOMEDIR}"; done`
    - Run As User: root
    - Daily at 0:00 AM
    - Hide Standard Output
    - Enabled

- System -> Advanced Settings -> NTP Servers
    - Delete all default NTP servers
    - Add time.cloudflare.com iburst prefer

- System -> Advanced Settings -> Storage
    - System Dataset Pool -> boot-pool

- System -> Services -> FTP (This is just pre-configuring. FTP is not used at the moment.)
    - Clients -> 30 (Just an arbitary high number)
    - Connections -> 0
    - Uncheck "Always Chroot"
    - Check "Allow Local User Login"
    - File Permissions -> 600
    - Directory Permissions -> 700
    - Enable TLS
        - Certificate -> Choose the certificate uploaded earlier
    - Minimum Passive Port -> 21000
    - Maximum Passive Port -> 21029
    - Check "Allow Transfer Resumption"
    - Masquerade Address -> Set to the external IP address

- System -> Services -> SSH
    - Enable SSH
    - Start SSH
    - Uncheck "Allow Password Authentication"
    - Auxiliary Parameters

    ```
    # LDAP SSH key
    AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys
    AuthorizedKeysCommandUser root

    # Security hardening
    DisableForwarding yes
    LoginGraceTime 15s
    MaxAuthTries 1
    PermitUserEnvironment no
    PermitUserRC no
    StrictModes yes

    # Use KeepAlive over SSH instead of with TCP to prevent spoofing
    TCPKeepAlive no
    ClientAliveInterval 15
    ClientAliveCountMax 4

    # Disabling unused authentication methods
    GSSAPIAuthentication no
    HostbasedAuthentication no
    PermitEmptyPasswords no
    KbdInteractiveAuthentication no
    KerberosAuthentication no
    ```

- System -> Alert Settings -> Remove SNMP Trap

## Apps

- Add a second virtual disk backed by SSDs in Proxmox
- Edit `/etc/pve/qemu-server/<vmid>.conf` and add `,serial=00000000000000000001` to the new disk configuration

- Turn the VM off, then back on to apply the new configuration

- Storage -> Create Pool
    - General Info -> Name -> Apps
    - Data -> Layout -> Stripe

- Storage -> Apps -> Disable scrubbing schedule

- Dataset -> Apps -> Dataset Details -> Edit -> Advanced Options
    - Sync -> Disabled
    - Compression level -> Off
    - Checksum -> Blake 3
    - Exec -> Off

- Apps
    - Choose Apps as the pool
    - Configuration -> Settings -> Preferred Trains
        - Stable
        - Community
        - Enterprise
