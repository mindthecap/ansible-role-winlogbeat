
# WinLogBeat Install and Configure

An Ansible role that installs and configures WinLogBeat. The configuration supports SSL. Implementation of this role has had guidance from https://github.com/CUSystem/ansible-role-winlogbeat. Since this is a major rewrite, there will not be a pull-request.

## Prerequisites

# Windows Configuration:

Get the latest version of SSH for Windows https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH.

## Role Variables

### Download and installation (Defaults): 

    winlogbeat_download_url_base: 'https://artifacts.elastic.co/downloads/beats/winlogbeat'
    file_ext: '.zip'
    winlogbeat_version: '7.9.3'
    winlogbeat_download_file: 'winlogbeat-{{ winlogbeat_version }}-windows-x86_64'
    winlogbeat_extract_location: "C:/Program Files/Winlogbeat"
    winlogbeat_install_location: "C:/Program Files/Winlogbeat/{{ winlogbeat_download_file }}"

### Example ( Default) Input:

    winlogbeat_event_logs:
      - name: ForwardedEvents
        batch_read_size: 8192
        api: wineventlog-experimental
        forwarded: true
        ignore_older: 72h
        processors:
          - script:
              when.equals.winlog.channel: Security
              lang: "javascript"
              id: "sysmon"
              file: "{{ winlogbeat_install_location }}/module/security/config/winlogbeat-security.js"
          - script:
              when.equals.winlog.channel: Microsoft-Windows-Sysmon/Operational
              lang: "javascript"
              id: "sysmon"
              file: "{{ winlogbeat_install_location }}/module/sysmon/config/winlogbeat-sysmon.js"

### Tags and Fields can be added :


    winlogbeat_tags:
       - service-X
       - web-tier


    winlogbeat_fields:
       env: staging
       system: app

### Output Host options:

### Output defaults:

    winlogbeat_output:
      output.logstash:
        loadbalance: true
        bulk_max_size: 4096
        pipelining: 4
        ssl.certificate_authorities: "{{ winlogbeat_install_location }}/{{ inventory_hostname }}/ca.crt"
        ssl.certificate: "{{ winlogbeat_install_location }}/{{ inventory_hostname }}/{{ inventory_hostname }}.cer"
        ssl.key: "{{ winlogbeat_install_location }}/{{ inventory_hostname }}/{{ inventory_hostname }}.key"


Output can be to Elasticsearch and/or Logstash:

    winlogbeat_output_elasticsearch_hosts:
      - "localhost:9200"

    winlogbeat_output_logstash_hosts:
      - "localhost:5044"

Adding hosts enables that output.  No defaults provided.  Therefore, one or both need to be configured.


### SSL options (Defaults)

    winlogbeat_ssl_certificate_authorities:
      - '{{ winlogbeat_install_location }}\\ca.crt'
    #  - "C:\trust_store\ca.pem"
    winlogbeat_ssl_verification_mode: # none, full
    winlogbeat_ssl_renegotiation: #never, once, freely
    winlogbeat_ssl_supported_protocols: #Collection: [SSLv3, TLSv1, TLSv1.0, TLSv1.1, TLSv1.2]
    winlogbeat_ssl_cipher_suites: #Collection - See docs for available types
    winlogbeat_ssl_curve_types: #Collection - [P-256, P-384, P521]

# Certificates
    winlogbeat_ssl_enabled: true
    winlogbeat_ssl_src: "certs/{{ inventory_hostname }}"


### Logging options (Defaults)

    winlogbeat_enable_logging: false
    winlogbeat_log_level: warning
    winlogbeat_log_dir: "{{ winlogbeat_install_location }}/{{ winlogbeat_download_file }}/logs"
    winlogbeat_log_filename: mybeat.log


### Overrides (Defaults)

    winlogbeat_conf_overrides:
      xpack.monitoring.enabled: false
      http.enabled: true
      http.port: 5066
      queue.mem:
        events: 65536
        flush.min_events: 8096
        flush.timeout: "5s"

# Sample task:

    - hosts: win-wefc-serverid
      gather_facts: false
      roles:
       - winlogbeat

Installing and upgrading Winlogbeat
    ansible-playbook tasks/winlogbeat.yml --tags all,winlogbeat_install

Changing configuration and/or certificates
    ansible-playbook tasks/winlogbeat.yml
    
# License

GNU GPLv3

