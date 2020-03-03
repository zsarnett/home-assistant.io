---
title: Certificate Expiry
description: Instructions on how to set up HTTPS (SSL) certificate expiry sensors within Home Assistant.
logo: home-assistant.png
ha_category:
  - Network
ha_release: 0.44
ha_iot_class: Configurable
ha_config_flow: true
ha_codeowners:
  - '@Cereal2nd'
  - '@jjlawren'
---

The `cert_expiry` sensor fetches information from a configured URL and displays the certificate expiry in days.

## Configuration

There are 2 options in configuring the `cert_expiry` sensor:

- Via the Home Assistant user interface where it will let you enter a host and port for the certificate to check.
- Via the Home Assistant `configuration.yaml` file.


```yaml
# Example configuration.yaml entry
sensor:
  - platform: cert_expiry
    host: home-assistant.io
```

{% configuration %}
host:
  description: The host FQDN (or IP) to retrieve certificate from.
  required: true
  type: string
port:
  description: The port number where the server is running.
  required: false
  default: 443
  type: integer
{% endconfiguration %}

## Attributes

The Certificate Expiry entities provide extra attributes to represent the state of the certificate.

| Name | Description |
| ---- | ----------- |
| `is_valid` | If the certificate is able to be validated: `True` / `False`.
| `error` | A human-readable error description if the certificate is considered invalid, "None" otherwise.

<div class='note warning'>
Make sure that the URL exactly matches your endpoint or resource.
</div>
