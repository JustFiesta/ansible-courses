# Custom Ansible Plugins

Extend Ansible's functionality by creating custom plugins for specific use cases like API interactions, data processing, or custom lookups.

[Custom Plugins ref](https://docs.ansible.com/ansible/latest/dev_guide/developing_plugins.html)

## Plugin Directory Structure

Place custom plugins in these standard locations

```shell
project/
├── lookup_plugins/
│   └── api_lookup.py
├── filter_plugins/
│   └── custom_filters.py
├── callback_plugins/
│   └── custom_callback.py
└── playbook.yml
```

### Custom Lookup Plugin Example

Create a custom lookup plugin to fetch data from external APIs and use it in your playbooks.

**Plugin File**: `lookup_plugins/api_lookup.py`

```python
from __future__ import (absolute_import, division, print_function)
__metaclass__ = type

DOCUMENTATION = r"""
  name: api_lookup
  author: Your Name <your.email@example.com>
  version_added: "0.1"
  short_description: Retrieve data from REST API
  description:
    - This lookup retrieves data from a REST API endpoint.
  options:
    endpoint:
      description: API endpoint to query
      required: True
    token:
      description: Authentication token
      required: False
"""

from ansible.errors import AnsibleError
from ansible.plugins.lookup import LookupBase
from ansible.utils.display import Display
import requests

display = Display()

class LookupModule(LookupBase):

    def run(self, terms, variables=None, **kwargs):
        api_base = "https://api.example.com/v1"
        endpoint = terms[0]
        token = kwargs.get('token')
        
        headers = {}
        if token:
            headers['Authorization'] = f"Bearer {token}"
        
        try:
            response = requests.get(f"{api_base}/{endpoint}", headers=headers)
            response.raise_for_status()
            return [response.json()]
            
        except requests.exceptions.RequestException as e:
            raise AnsibleError(f"API request failed: {str(e)}")
```

### Usage in Playbook

```shell
- name: Get users from API
  ansible.builtin.debug:
    msg: "{{ lookup('api_lookup', 'users', token=api_token) }}"

- name: Use API data in template
  ansible.builtin.template:
    src: user_report.j2
    dest: /tmp/user_report.html
  vars:
    users: "{{ lookup('api_lookup', 'users', token=api_token) }}"
```

## Custom Plugins Integration with Execution Environments

Custom plugins are integrated into Execution Environments by being included in the container image during the build process, ensuring your automation has access to all required custom functionality in any environment.

### Plugin Discovery in Execution Environments

Within the Execution Environment container, Ansible searches for plugins in these default paths:

- `/usr/share/ansible/plugins/` - system-wide plugin directory
- Paths configured via `environment variables` or `ansible.cfg`.

### Adding Plugins to Execution Environments

There are two primary approaches to include custom plugins in your Execution Environment.

#### Method 1: Include Plugin Files in Build Context

Add plugin source files directly to your execution environment build context and reference them in your `execution-environment.yml`.

```yaml
version: 3
dependencies:
  galaxy: requirements.yml
  python: requirements.txt

additional_build_files:
  - src: plugins/lookup/*.py
    dest: plugins/lookup

additional_build_steps:
  append_final:
    - COPY _build/plugins/lookup/ /usr/share/ansible/plugins/lookup/
```

With this directory structure:

```shell
project/
├── execution-environment.yml
├── plugins/
│   └── lookup/
│       ├── api_lookup.py
│       └── custom_lookup.py
├── requirements.yml
└── requirements.txt
```

#### Method 2: Package and Install via Python

For more complex plugins, create a Python package and install it via pip

```yaml
version: 3
dependencies:
  python: requirements.txt

additional_build_steps:
  prepend_final:
    - RUN pip install /tmp/custom_plugins-1.0.0-py3-none-any.whl
```
