## Mandatory Initial Server Configurations and SSH Authentication with Ansible

Here's how to use Ansible for the mandatory initial server configurations and switch from password authentication to SSH key-based authentication:

**1. Generate SSH Key Pair (on your Control Machine):**

* Before configuring servers, you'll need an SSH key pair for secure access.
* Use `ssh-keygen` to generate a key pair:

```bash
ssh-keygen -t rsa -b 4096
```

* Follow the prompts, entering a secure passphrase (optional) for the private key.
* This will create two files: `id_rsa` (private key) and `id_rsa.pub` (public key).

**2. Ansible User and Sudo Privileges:**

* You'll need an Ansible user with sudo privileges on the target servers.
* This can be a dedicated user or an existing user with appropriate permissions.

**3. Configure Servers with Ansible Playbook:**

* Create a playbook named `initial_server_config.yml` in your project directory.

Here's a sample playbook:

```yaml
---
- hosts: servers
  become: true  # Gain root privileges
  tasks:
    - name: Update package lists
      apt: update_cache=yes

    - name: Install basic utilities
      apt: 
        name: "{{ item }}" 
        state: present
      loop:
        - vim
        - net-tools

    - name: Create a user for Ansible
      user:
        name: ansible_user
        state: present
        groups: sudo  # Add user to sudo group

    - name: Set password for Ansible user (temporary)
      user:
        name: ansible_user
        password: "{{ ansible_password }}"  # Replace with a strong password
        state: present

    - name: Copy SSH public key to authorized_keys for Ansible user
      authorized_key:
        user: ansible_user
        key: "{{ lookup('file', '/home/your_user/.ssh/id_rsa.pub') }}"  # Replace with your path
        state: present

    - name: Disable password authentication for SSH
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: '^PasswordAuthentication'
        line: 'PasswordAuthentication no'
        state: present

    - name: Restart SSH service
      service:
        name: sshd
        state: restarted

  # Define variables for password and paths (optional)
  vars:
    ansible_password: "your_strong_password"  # Replace with a strong password
```

**Explanation:**

* This playbook updates package lists, installs basic utilities, creates an `ansible_user`, sets a temporary password for the user, and copies your SSH public key to the user's `authorized_keys` file.
* The `lineinfile` module disables password authentication in SSH configuration.
* Finally, the SSH service is restarted.
* Define the `ansible_password` variable in the `vars` section with a strong password (replace with your actual password).

**4. Run the Playbook:**

* With your SSH key pair generated and the playbook created, run:

```bash
ansible-playbook django_deploy.yml
```

* This will connect to your servers (using the temporary password), configure them, and switch to SSH key-based authentication.

**5. Secure Login with SSH Key:**

* After the playbook execution, you can connect to your servers using your SSH private key:

```bash
ssh -i /home/your_user/.ssh/id_rsa ansible_user@server_ip
```

* Replace the paths and usernames with yours. You won't be prompted for a password since the server recognizes your SSH key.

**Remember:**

* This is a basic configuration. You might need additional security measures depending on your environment.
* Don't share your private key with anyone.
* Consider using a password manager for your Ansible user's temporary password.

By following these steps, you can securely configure your servers using Ansible and prepare them for further deployment tasks like deploying your Django backend.
