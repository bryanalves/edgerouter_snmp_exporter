# Prometheus SNMP Generator for Edgerouter

This generates an snmp-exporter configuration suitable for gathering a bunch of useful stats from edgerouter devices.
This was originally based on snmpwalking an Edgerouter Lite, but should be relevant for most other devices in the product line.

# Usage

Take the checked in snmp.yml and run snmp-exporter with it, ex:

```
docker run --rm -p 9116:9116 -v `pwd`/snmp.yml:/etc/snmp_exporter/snmp.yml prom/snmp-exporter
```

Then query it with your edgerouter:

`curl "localhost:9116/snmp?target=192.168.1.1&module=ubiquiti_edgemax"`

Once working, use a snippet like the following to have prometheus start ingesting these:

```
scrape_configs:
  - job_name: 'snmp'
    static_configs:
      - targets:
        - 192.168.1.1  # SNMP device.
    metrics_path: /snmp
    params:
      module: [ubiquiti_edgemax]
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: my-service-name:9116  # The SNMP exporter's Service name and port.
```

# Regenerating snmp.yml

The provided generator.yml is what was used to generate the snmp.yml.  To regenerate it:

```
docker run --rm \
  -v /usr/share/snmp/mibs:/root/.snmp/mibs \
  -v $PWD/generator.yml:/opt/generator.yml:ro \
  -v $PWD/:/opt/ snmp-generator generate
```

This assumes you have net-snmp mibs installed locally
