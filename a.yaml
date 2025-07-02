---
- name: RHEL Server Uptime Check with Thycotic
  hosts: all
  gather_facts: false
  vars:
    thycotic_url: "https://your-thycotic-server.example.com"
    thycotic_secret_id: 123  # ID of the secret containing SSH credentials
    thycotic_username: "{{ vault_thycotic_username }}"
    thycotic_password: "{{ vault_thycotic_password }}"

  tasks:
    - name: Retrieve SSH credentials from Thycotic
      delegate_to: localhost
      run_once: true
      no_log: true
      block:
        - name: Ensure thycotic python library is available
          pip:
            name: thycotic
            state: present

        - name: Get SSH credentials from Thycotic
          community.general.thycotic_secret_server:
            url: "{{ thycotic_url }}"
            username: "{{ thycotic_username }}"
            password: "{{ thycotic_password }}"
            secret_id: "{{ thycotic_secret_id }}"
            field: "Private Key"  # For SSH key authentication
          register: thycotic_secret
          no_log: true

    - name: Set SSH credentials as facts
      set_fact:
        ssh_username: "{{ thycotic_secret.secret['Username'] }}"
        ssh_private_key: "{{ thycotic_secret.secret['Private Key'] }}"
      no_log: true

    - name: Create temporary SSH key file
      delegate_to: localhost
      run_once: true
      no_log: true
      tempfile:
        state: file
      register: ssh_key_file

    - name: Write private key to temporary file
      delegate_to: localhost
      run_once: true
      no_log: true
      copy:
        content: "{{ ssh_private_key }}"
        dest: "{{ ssh_key_file.path }}"
        mode: 0600

    - name: Check server uptime via SSH
      ansible.builtin.command: uptime
      register: uptime_result
      vars:
        ansible_user: "{{ ssh_username }}"
        ansible_ssh_private_key_file: "{{ ssh_key_file.path }}"
        ansible_ssh_common_args: "-o StrictHostKeyChecking=no"

    - name: Display uptime results
      debug:
        msg: "{{ uptime_result.stdout }}"

    - name: Clean up temporary SSH key
      delegate_to: localhost
      run_once: true
      no_log: true
      file:
        path: "{{ ssh_key_file.path }}"
        state: absent
      when: ssh_key_file.path is defined
