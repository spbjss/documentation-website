---
layout: default
title: Docker security configuration
parent: Install OpenSearch
grand_parent: OpenSearch
nav_order: 5
---

# Docker security configuration

Before deploying to a production environment, you should replace the demo security certificates and configuration YAML files with your own. With the tarball, you have direct access to the file system, but the Docker image requires modifying the Docker storage volumes include the replacement files.

Additionally, you can set the Docker environment variable `DISABLE_INSTALL_DEMO_CONFIG` to `true`. This change completely disables the demo installer.

## Sample Docker Compose file

```yml
version: '3'
services:
  opensearch-node1:
    image: opensearchproject/opensearch:{{site.opensearch_version}}
    container_name: opensearch-node1
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node1
      - discovery.seed_hosts=opensearch-node1,opensearch-node2
      - cluster.initial_master_nodes=opensearch-node1,opensearch-node2
      - bootstrap.memory_lock=true # along with the memlock settings below, disables swapping
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m" # minimum and maximum Java heap size, recommend setting both to 50% of system RAM
      - network.host=0.0.0.0 # required if not using the demo security configuration
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536 # maximum number of open files for the OpenSearch user, set to at least 65536 on modern systems
        hard: 65536
    volumes:
      - opensearch-data1:/usr/share/opensearch/data
      - ./root-ca.pem:/usr/share/opensearch/config/root-ca.pem
      - ./node.pem:/usr/share/opensearch/config/node.pem
      - ./node-key.pem:/usr/share/opensearch/config/node-key.pem
      - ./admin.pem:/usr/share/opensearch/config/admin.pem
      - ./admin-key.pem:/usr/share/opensearch/config/admin-key.pem
      - ./custom-opensearch.yml:/usr/share/opensearch/config/opensearch.yml
      - ./internal_users.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/internal_users.yml
      - ./roles_mapping.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/roles_mapping.yml
      - ./tenants.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/tenants.yml
      - ./roles.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/roles.yml
      - ./action_groups.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/action_groups.yml
    ports:
      - 9200:9200
      - 9600:9600 # required for Performance Analyzer
    networks:
      - opensearch-net
  opensearch-node2:
    image: opensearchproject/opensearch:{{site.opensearch_version}}
    container_name: opensearch-node2
    environment:
      - cluster.name=opensearch-cluster
      - node.name=opensearch-node2
      - discovery.seed_hosts=opensearch-node1,opensearch-node2
      - cluster.initial_master_nodes=opensearch-node1,opensearch-node2
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms512m -Xmx512m"
      - network.host=0.0.0.0
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    volumes:
      - opensearch-data2:/usr/share/opensearch/data
      - ./root-ca.pem:/usr/share/opensearch/config/root-ca.pem
      - ./node.pem:/usr/share/opensearch/config/node.pem
      - ./node-key.pem:/usr/share/opensearch/config/node-key.pem
      - ./admin.pem:/usr/share/opensearch/config/admin.pem
      - ./admin-key.pem:/usr/share/opensearch/config/admin-key.pem
      - ./custom-opensearch.yml:/usr/share/opensearch/config/opensearch.yml
      - ./internal_users.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/internal_users.yml
      - ./roles_mapping.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/roles_mapping.yml
      - ./tenants.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/tenants.yml
      - ./roles.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/roles.yml
      - ./action_groups.yml:/usr/share/opensearch/plugins/opensearch-security/securityconfig/action_groups.yml
    networks:
      - opensearch-net
  opensearch-dashboards
    image: opensearchproject/opensearch-dashboards:{{site.opensearch_version}}
    container_name: opensearch-dashboards
    ports:
      - 5601:5601
    expose:
      - "5601"
    environment:
      OPENSEARCH_URL: https://opensearch-node1:9200
      OPENSEARCH_HOSTS: https://opensearch-node1:9200
    volumes:
      - ./custom-opensearch_dashboards.yml:/usr/share/opensearch-dashboards/config/opensearch_dashboards.yml
    networks:
      - opensearch-net

volumes:
  opensearch-data1:
  opensearch-data2:

networks:
  opensearch-net:
```

Then make your changes to `opensearch.yml`. For a full list of settings, see [Security](../../../security/configuration/). This example adds (extremely) verbose audit logging:

```yml
opensearch_security.ssl.transport.pemcert_filepath: node.pem
opensearch_security.ssl.transport.pemkey_filepath: node-key.pem
opensearch_security.ssl.transport.pemtrustedcas_filepath: root-ca.pem
opensearch_security.ssl.transport.enforce_hostname_verification: false
opensearch_security.ssl.http.enabled: true
opensearch_security.ssl.http.pemcert_filepath: node.pem
opensearch_security.ssl.http.pemkey_filepath: node-key.pem
opensearch_security.ssl.http.pemtrustedcas_filepath: root-ca.pem
opensearch_security.allow_default_init_securityindex: true
opensearch_security.authcz.admin_dn:
  - CN=A,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA
opensearch_security.nodes_dn:
  - 'CN=N,OU=UNIT,O=ORG,L=TORONTO,ST=ONTARIO,C=CA'
opensearch_security.audit.type: internal_opensearch
opensearch_security.enable_snapshot_restore_privilege: true
opensearch_security.check_snapshot_restore_write_privileges: true
opensearch_security.restapi.roles_enabled: ["all_access", "security_rest_api_access"]
cluster.routing.allocation.disk.threshold_enabled: false
opensearch_security.audit.config.disabled_rest_categories: NONE
opensearch_security.audit.config.disabled_transport_categories: NONE
```

Use this same override process to specify new [authentication settings](../../../security/configuration/configuration/) in `/usr/share/opensearch/plugins/opensearch-security/securityconfig/config.yml`, as well as new default [internal users, roles, mappings, action groups, and tenants](../../../security/configuration/yaml/).

To start the cluster, run `docker-compose up`.

If you encounter any `File /usr/share/opensearch/config/opensearch.yml has insecure file permissions (should be 0600)` messages, you can use `chmod` to set file permissions before running `docker-compose up`. Docker Compose passes files to the container as-is.
{: .note }

Finally, you can reach OpenSearch Dashboards at http://localhost:5601, sign in, and use the **Security** panel to perform other management tasks.

## Using certificates with Docker

To use your own certificates in your configuration, add all of the necessary certificates to the volumes section of the Docker Compose file:

```yml
volumes:
- ./root-ca.pem:/full/path/to/certificate.pem
- ./admin.pem:/full/path/to/certificate.pem
- ./admin-key.pem:/full/path/to/certificate.pem
#Add other certificates
```

After replacing the demo certificates with your own, you must also include a custom `opensearch.yml` in your setup, which you need to specify in the volumes section.

```yml
volumes:
#Add certificates here
- ./custom-opensearch.yml: /full/path/to/custom-opensearch.yml
```

Remember that the certificates you specify in your Docker Compose file must be the same as the certificates listed in your custom `opensearch.yml` file. At a minimum, you should replace the root, admin, and node certificates with your own. For more information about adding and using certificates, see [Configure TLS certificates](../security/configuration/tls.md).

```yml
opensearch_security.ssl.transport.pemcert_filepath: new-node-cert.pem
opensearch_security.ssl.transport.pemkey_filepath: new-node-cert-key.pem
opensearch_security.ssl.transport.pemtrustedcas_filepath: new-root-ca.pem
opensearch_security.ssl.http.pemcert_filepath: new-node-cert.pem
opensearch_security.ssl.http.pemkey_filepath: new-node-cert-key.pem
opensearch_security.ssl.http.pemtrustedcas_filepath: new-root-ca.pem
opensearch_security.authcz.admin_dn:
  - CN=admin,OU=SSL,O=Test,L=Test,C=DE
```

To start the cluster, run `docker-compose up` as usual.
