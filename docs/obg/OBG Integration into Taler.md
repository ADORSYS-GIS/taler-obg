# GNU Taler and Open Banking Gateway (OBG) Integration

## Overview
The integration of Open Banking Gateway (OBG) into GNU Taler requires a well-structured infrastructure setup, configuration management, and dependency resolution. This document outlines the necessary components, deployment strategies, and considerations for integrating OBG into the existing Taler infrastructure using Ansible.

## Existing Infrastructure

### GNU Taler
- **Deployment Method:** Uses Ansible for infrastructure automation.
- **Containerization:** Podman.
- **Database:** PostgreSQL.
- **Hosting:** Applications are deployed via `deb.taler.net` for Debian-based distributions.
- **Ansible Repository:** `ansible-taler-exchange` public Git repository.
- **Main Playbook:** `setup.yml` orchestrates the deployment of Taler services.
- **Roles:** `common_packages`, `webserver`, `database`, `exchange`, `auditor`, etc.

### Open Banking Gateway (OBG)
- **Technology Stack:** Spring Boot and Maven.
- **Containerization:** Uses Docker Compose for orchestration.
- **Database:** PostgreSQL.
- **Hosting:** Deployed independently with its dependencies and configurations.

## Integration Strategy
To successfully integrate OBG into GNU Taler, the following steps are proposed:

### 1. Modify Ansible Playbooks
- Update `setup.yml` in `ansible-taler-exchange` to include a role for installing OBG.
- Create a new role specific to OBG, which will:
  - Install required dependencies (Java, Maven, etc.).
  - Deploy the OBG application.
  - Configure networking and service interactions.

### 2. Create a Separate OBG Playbook
- Instead of merging OBG dependencies with Taler, create a standalone playbook that provisions only OBG. This prevents unnecessary role execution from Taler that may not be needed by OBG.
- The playbook will include roles such as `common_packages`, `webserver`, `database`, `obg`.

### 3. Address Containerization Differences
- OBG uses Docker Compose, while Taler relies on Podman.
- Use Ansible to manage both Docker and Podman, ensuring that services are deployed correctly in their respective environments.

### 4. Define Infrastructure Requirements
- **Server:** Needs to accommodate both Taler and OBG services.
- **Database:** PostgreSQL instance for OBG should be validated for compatibility with Taler’s existing database setup.
- **Networking:** Ensure OBG services are properly exposed and can communicate with Taler services securely.
- **Security:** Maintain compliance with GNU Taler’s security policies.

### 5. Deployment Steps
- Add an OBG installation role to `setup.yml`.
- Create a separate `playbook-obg.yml` for standalone OBG deployment.
- Define `roles/obg` to manage:
  - Java and Maven installation.
  - Deployment of OBG services.
  - Configuration of PostgreSQL for OBG.
  - Integration with Taler services.
- Ensure correct environment variables and networking configurations.
- Test deployment using Ansible to verify successful integration.

## Conclusion
Effective use of Ansible roles and playbooks will ensure a smooth integration of OBG into GNU Taler while keeping the infrastructure lightweight and modular. Future improvements can be made as the integration evolves.
