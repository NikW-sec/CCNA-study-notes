# 6.0 Automation and Programmability (10%)

## 6.1 Why automation

Manual, box-by-box CLI configuration is slow, inconsistent, and the biggest single source of outages (typos, drift, undocumented changes). Automation gives repeatability, version control, and the ability to validate a change before it ships.

## 6.2 Traditional vs controller-based networking

In a traditional network every device runs its own **control plane** (routing protocol decisions, STP, ARP) and **data plane** (actually forwarding frames and packets), and you configure each one individually.

Controller-based networking centralizes the control plane in an **SDN controller**, which programs the devices' data planes. The controller exposes:

- a **northbound interface (NBI)** — a REST API upward to applications and scripts, which is how you or a portal ask for intent;
- a **southbound interface (SBI)** — NETCONF, RESTCONF, OpenFlow, or SNMP downward, which is how the controller programs the devices.

**Cisco DNA Center** is the campus example: intent-based, with assurance, automated provisioning, and policy applied to groups rather than to interfaces.

## 6.3 Data formats

- **JSON** — key/value pairs in `{}`, arrays in `[]`, values are string, number, boolean, null, object, or array. The most common REST format.
- **XML** — tag-based with opening and closing tags, more verbose, used by NETCONF.
- **YAML** — indentation-based, no brackets, lists prefixed with `-`. Used by Ansible playbooks and human-edited config.

```json
{
  "device": {
    "hostname": "R1",
    "interfaces": [
      { "name": "GigabitEthernet0/0", "ip": "192.168.1.1", "enabled": true }
    ]
  }
}
```

## 6.4 REST APIs

REST is stateless and runs over HTTP(S). CRUD maps to the verbs: **GET** (read), **POST** (create), **PUT** (replace), **PATCH** (modify), **DELETE** (remove).

Key response codes: 200 OK, 201 Created, 400 bad request, 401 unauthorized, 403 forbidden, 404 not found, 500 server error. Authentication is usually a token obtained from a login call and sent in a header.

## 6.5 Configuration management tools

| | Puppet | Chef | Ansible |
|---|---|---|---|
| Language | Puppet DSL | Ruby | YAML |
| Model | pull | pull | **push** |
| Agent | required | required | **agentless** |
| Transport | TCP 8140 | TCP 10002 | **SSH / NETCONF** |
| Files | manifests | recipes/cookbooks | playbooks/inventory |

Agentless and push-based is why Ansible dominates network automation: nothing has to be installed on the switch.

All three are **declarative** — you describe the desired end state and the tool works out how to reach it, making runs idempotent (running twice changes nothing the second time).

## 6.6 JSON-encoded config vs CLI

The exam asks you to read a JSON blob and say what it configures, or to spot which JSON matches a given CLI. Practise mapping `"ip"`/`"mask"` keys onto `ip address`, `"shutdown": false` onto `no shutdown`, and nested objects onto config sub-modes.
