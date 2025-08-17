# 🔹 1. Built-in modules

- These are modules that come bundled with Ansible itself (no need to install extra content).
- Example: copy, file, yum, apt, service.
- Execution:
  - They are executed on the managed node (target server), unless the module is specifically a "local" one (like add_host or meta).
  
# 🔹 2. Collections  
  
- Collections are just a packaging format in Ansible.
- They can contain:
  - Roles
  - Modules
  - Plugins
  - Playbooks
  - Docs/tests
- Collections can be installed from Ansible Galaxy or private repos.
- Example:
  - amazon.aws (for AWS modules/plugins/roles)
  - community.general (tons of community modules)
- Execution:
  - Modules inside collections (e.g., amazon.aws.ec2) often interact with cloud APIs.
  - When you run them:
    - The API calls happen from the control node (because AWS, Azure, GCP APIs aren’t running on the managed node).
    - Some other modules in collections (like Linux hardening modules) still run on the managed node.
