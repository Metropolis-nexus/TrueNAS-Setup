# TrueNAS-Setup

- truenas_admin -> Two factor authentication -> Configure 2FA Secret

- Storage -> Create Pool
    - General Info -> Check "Encryption" -> AES-256-GCM
    - Data -> Layout -> Choose the appropriate RAIDZ config

- Dataset -> Dataset Details -> Edit -> Advanced Options
    - Preset -> Generic (always use generic for interoperability - NFSv4 ACL is bespoke)
    - Sync -> Always
    - Compression level -> ZFS
    - Checksum -> Blake 3
    - Exec -> Off

- Network
    - Uncheck all service announcements

- Credentials -> Users -> root
    - Add email
    - Add SSH keys

- System -> General -> GUI
    - Check "Web Interface HTTP -> HTTPS Redirect"
    - Uncheck "Usage Collection"

- System -> General -> Email
    - Configure SMTP notications

- System -> Advanced Settings -> Console
    - Enable Serial Console
    - Serial Speed -> 115200
    - Access -> Set session timeout to 14400 (4 hours)
    - Global Two Factor Authentication -> Enable Two Factor Authentication Globally

- System -> Advanced Settings -> NTP Servers
    - Delete all default NTP servers
    - Add time.cloudflare.com iburst prefer

- System -> Advanced Settings -> Storage
    - System Dataset Pool -> boot-pool

- System -> Services
    - Enable SSH
    - Start SSH
    - Uncheck "Allow Password Authentication"
