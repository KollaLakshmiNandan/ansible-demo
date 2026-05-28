# ansible-demo
Step 1: What is Ansible?
Ansible is an automation tool used to automate server configuration, software installation, deployments, and repetitive tasks.

Examples:

Install packages on 100 servers

Create users automatically

Deploy websites

Restart services

Step 2: Explain components
Inventory

Inventory stores the list of target machines.

Example:

INI

localhost ansible_connection=local
Explain:

localhost → current machine

ansible_connection=local → execute on the same machine

Playbook

Playbook is a YAML file containing tasks.

Example:

YAML

---
- name: Deploy HTML file
  hosts: localhost

  tasks:
    - name: Create HTML file
      copy:
        content: |
          <h1>Hello Team</h1>
          <p>Deployed using Ansible</p>

        dest: /tmp/index.html
Explain line by line:

--- → start of YAML

name → playbook name

hosts → target machine

tasks → list of actions

copy → built-in Ansible module

content → file data

dest → destination path

Step 3: Run live
Show commands:

Bash

mkdir ansible-demo
cd ansible-demo

touch inventory
touch deploy.yml
Run:

Bash

ansible-playbook -i inventory deploy.yml
Expected output:

Bash

PLAY RECAP

localhost : ok=2 changed=1 failed=0
Explain:

ok=2 → tasks executed successfully

changed=1 → something changed

failed=0 → no errors

Step 4: Show result
Bash

cat /tmp/index.html
or

Bash

open /tmp/index.html
Then say:

We manually created a file using a single command. Imagine deploying the same thing to 100 servers automatically. That's where Ansible becomes powerful.

For colleagues, end with one interview question:

"What is the difference between inventory and playbook?"

Answer:

Inventory → list of systems

Playbook → list of tasks to execute

This will sound like a proper trainer-led hands-on session rather than just reading commands.


ChatGPT said:
Great. Variables are the next logical topic after ad-hoc commands.

What are Variables?

Variables store values that can be reused instead of hardcoding them repeatedly.

Without variables:

YAML

---
- hosts: myservers

  tasks:
    - name: Create file
      copy:
        content: "Hello Nandy"
        dest: /tmp/Nandy.html
Problem:

If you want to change Nandy to Lakshmi or change the filename, you must edit multiple places.

With variables:

YAML

---
- name: Variables Demo
  hosts: myservers

  vars:
    username: Nandy
    filename: demo.html

  tasks:
    - name: Create file using variables
      copy:
        content: "Hello {{ username }}"
        dest: "/tmp/{{ filename }}"
Breakdown:


vars:             → define variables
username: Nandy   → variable 1
filename: demo.html → variable 2
{{ }}             → variable syntax
Run:

Bash

ansible-playbook -i inventory nand.yml
Verify:

Bash

cat /tmp/demo.html
Output:


Hello Nandy
Let's make it more interesting using your simulated servers:

YAML

---
- name: Variables with multiple servers
  hosts: myservers

  vars:
    company: MidTech

  tasks:
    - name: Create HTML file
      copy:
        content: |
          <h1>{{ inventory_hostname }}</h1>
          <p>Company: {{ company }}</p>

        dest: "/tmp/{{ inventory_hostname }}.html"
Output for server1.html:

HTML




<h1>server1</h1>
<p>Company: MidTech</p>
Output for server2.html:

HTML




<h1>server2</h1>
<p>Company: MidTech</p>
How to explain to colleagues:

Variables avoid repeating values and make playbooks reusable. If we change the variable once, it updates everywhere automatically.

Next topic after this: Loops ✅

ChatGPT said:
Handlers are special tasks that run only when notified by another task.

Purpose:

Execute something only when a change happens.

Real-world example:

Update Apache configuration file

Restart Apache only if the file changed

Without handlers:

YAML

---
- hosts: myservers

  tasks:
    - name: Copy config
      copy:
        content: "new config"
        dest: /tmp/config.txt

    - name: Restart service
      command: echo "Service Restarted"
Problem:

Restart runs every time, even if nothing changed.

With handlers:

Open:

Bash

nano nand.yml
Paste:

YAML

---
- name: Handler Demo
  hosts: myservers

  tasks:
    - name: Update file
      copy:
        content: "Hello Nandy"
        dest: /tmp/demo.txt

      notify: Restart Service

  handlers:
    - name: Restart Service
      command: echo "Service Restarted"
Run:

Bash

ansible-playbook -i inventory nand.yml
First run:


TASK [Update file]
changed: [server1]

RUNNING HANDLER [Restart Service]
changed: [server1]
Second run:


TASK [Update file]
ok: [server1]

(No handler runs)
Why?

Because:


First run:
File changed → notify sent → handler runs ✅

Second run:
No change → no notify → handler skipped ✅
Real company example:

YAML

---
- hosts: webservers

  tasks:
    - name: Update nginx config
      copy:
        src: nginx.conf
        dest: /etc/nginx/nginx.conf

      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
Explanation to colleagues:

Handlers prevent unnecessary actions. If configuration changes, restart the service; otherwise do nothing.

Flow:


Task changes file
        ↓
notify
        ↓
Handler triggered
        ↓
Service restarted
This is heavily used in real DevOps projects.
ChatGPT said:
Great. Templates are one of the most important Ansible topics because they create dynamic files using variables.

Difference:


files/      → static content
templates/  → dynamic content with variables
Example:

Without templates:

HTML




<h1>Hello Nandy</h1>
Every server gets the same content ❌

With templates:

HTML




<h1>Hello {{ inventory_hostname }}</h1>
Each server gets different content automatically ✅

Let's do hands-on.

Step 1: Create template folder
Bash

mkdir -p templates
Create template file:

Bash

nano templates/index.j2
Paste:

HTML




<html>
<body>
<h1>Server Name: {{ inventory_hostname }}</h1>
<p>Welcome {{ username }}</p>
<p>Environment: {{ env }}</p>
</body>
</html>
Save.

Step 2: Update nand.yml
Open:

Bash

nano nand.yml
Paste:

YAML

---
- name: Template Demo
  hosts: myservers

  vars:
    username: Nandy
    env: Development

  tasks:
    - name: Deploy template
      template:
        src: templates/index.j2
        dest: "/tmp/{{ inventory_hostname }}.html"
Step 3: Run
Bash

ansible-playbook -i inventory nand.yml
Expected:


changed: [server1]
changed: [server2]
changed: [server3]
changed: [server4]
changed: [server5]
Step 4: Verify
Bash

cat /tmp/server1.html
Output:

HTML




<html>
<body>
<h1>Server Name: server1</h1>
<p>Welcome Nandy</p>
<p>Environment: Development</p>
</body>
</html>
For server3:

Bash

cat /tmp/server3.html
Output:

HTML




<html>
<body>
<h1>Server Name: server3</h1>
<p>Welcome Nandy</p>
<p>Environment: Development</p>
</body>
</html>
Explain to colleagues:

Templates allow us to generate dynamic configuration files or web pages. One template can create different output for different servers using variables.
Great. Vault is used to store sensitive information securely in Ansible.

Examples of sensitive data:

Passwords

API keys

Database credentials

SSH keys

AWS access keys

Without Vault ❌

YAML

---
- hosts: myservers

  vars:
    db_password: mypassword123
Problem:

Anyone opening the playbook can see the password.

With Vault ✅

Step 1: Create encrypted file
Run:

Bash

ansible-vault create secrets.yml
It will ask:


New Vault password:
Confirm New Vault password:
Enter a password (example):


nandy123
Now an editor opens.

Add:

YAML

db_user: admin
db_password: secret123
Save:

Ctrl + X

Y

Enter

Step 2: View the file
Run:

Bash

cat secrets.yml
You'll see encrypted text:


$ANSIBLE_VAULT;1.1;AES256
3536363462356334...
Meaning:

Password is hidden and encrypted.

Step 3: Use vault variables in playbook
Open:

Bash

nano nand.yml
Paste:

YAML

---
- name: Vault Demo
  hosts: myservers

  vars_files:
    - secrets.yml

  tasks:
    - name: Display username
      debug:
        msg: "User is {{ db_user }}"
Step 4: Run playbook
Bash

ansible-playbook -i inventory nand.yml --ask-vault-pass
Enter:


nandy123
Output:


User is admin
Useful commands:

Edit vault

Bash

ansible-vault edit secrets.yml
Change password

Bash

ansible-vault rekey secrets.yml
Decrypt file

Bash

ansible-vault decrypt secrets.yml
Encrypt existing file

Bash

ansible-vault encrypt secrets.yml
Explain to colleagues:

Vault securely stores sensitive information by encrypting it so passwords and keys are not exposed in playbooks.

Real company usage:


AWS Secret Keys
Database Passwords
SSH Credentials
API Tokens
You've now covered another important intermediate Ansible topic: Vault 🔐
