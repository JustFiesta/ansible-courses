# URI requests

## URI Module

Make HTTP/HTTPS requests to interact with web services, APIs, and endpoints. Supports various HTTP methods, authentication mechanisms, and response handling.

```yaml
- name: GET request with API token
  ansible.builtin.uri:
    url: https://api.example.com/v1/users
    method: GET
    headers:
      Authorization: "Bearer {{ api_token }}"
      Content-Type: "application/json"
    status_code: 200
    return_content: yes
  register: api_response

- name: Login POST request with JSON body
  ansible.builtin.uri:
    url: https://api.example.com/v1/users
    method: POST
    body: '{
      "email": "john@example.com",
      "password": "admin"
    }'
    body_format: json
    status_code: 201
  register: login_result

- name: Display token from previous step
  ansible.builtin.debug:
    var: login_result.json.token

- name: Handle different status codes
  ansible.builtin.uri:
    url: https://api.example.com/v1/data
    method: GET
    status_code: 200,404
  register: data_result
```

## Get URL Module - download data from sites

Download files from HTTP, HTTPS, or FTP URLs to remote or local systems.

```yaml
- name: Download file to remote host
  ansible.builtin.get_url:
    url: https://example.com/software.tar.gz
    dest: /tmp/software.tar.gz
    checksum: "sha256:abc123..."
    timeout: 30

- name: Download with authentication
  ansible.builtin.get_url:
    url: https://api.example.com/reports/daily.pdf
    dest: /tmp/report.pdf
    headers:
      Authorization: "Bearer {{ api_token }}"
```
