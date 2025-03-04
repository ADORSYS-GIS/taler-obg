# Creating a Debian Package for Open Banking Gateway (OBG)

This document provides a step-by-step guide for packaging the Open Banking Gateway as a Debian (.deb) package for installation and management using `dpkg` and `systemd`.

## Prerequisites
Before you begin, ensure you have the following prerequisites:
1. **dpkg**: The Debian package management tool. You can install it using:
   ```bash
   sudo apt-get install dpkg
   ```
2. **Main JAR File**: The main JAR file of your Open Banking Gateway project -> from the `opba-embedded-starter` i.e `open-banking-gateway-1.0.1-SNAPSHOT-exec.jar`.
   
3. **PostgreSQL**: A running PostgreSQL database. You can install it using:
   ```bash
   sudo apt-get install postgresql
   ```
4. **Java**: Ensure Java is installed on your system. You can install it using:
   ```bash
   sudo apt-get install default-jdk
   ```
5. **Move the JAR File**: Place your main JAR file into the correct directory:
   ```bash
   cp path/to-your/jar-file/<your-main-jar-file>-exec.jar /var/lib/obg/
    ```
6. **Create the obg User**: The service runs under a dedicated `obg` user.
    ```bash
    sudo useradd -r -s /bin/false obg
    ```

## 1. Directory Structure
The Debian package must follow a specific directory structure to properly place files during installation.

```
obg-deb/
├── DEBIAN
│   └── control
├── lib
│   └── systemd
│       └── system
│           └── obg.service
├── usr
│   └── local
│       └── bin
│           └── obg
└── var
    └── lib
        └── obg
            └── <your-main-jar-file>-exec.jar #The main jar file of your obg project
```

## 2. Creating the Control File
The control file inside the `DEBIAN/` directory defines metadata about the package.

**File: `DEBIAN/control`**
```
Package: obg
Version: 1.0.0
Architecture: amd64
Maintainer: Your Name <your.email@example.com>
Description: Open Banking Gateway Service for Payment Processing
```

## 3. Creating the Systemd Service File
This file ensures that the Open Banking Gateway starts and stops as a systemd service.

**File: `lib/systemd/system/obg.service`**
```
[Unit]
Description=Open Banking Gateway (OBG) Service
After=network.target 

[Service]
User=obg
ExecStart=/usr/bin/java -jar /var/lib/obg/<your-main-jar-file>-exec.jar
SuccessExitStatus=143
Restart=on-failure
RestartSec=10
StandardOutput=append:/var/log/obg.log
StandardError=append:/var/log/obg.log

[Install]
WantedBy=multi-user.target
```

## 4. Creating the Executable Script
This script starts the Open Banking Gateway and logs output.

**File: `usr/local/bin/obg`**
```
#!/bin/bash
exec java -jar /var/lib/obg/<your-main-jar-file>-exec.jar
```

###### Note: Replace `/var/lib/obg/<your-main-jar-file>-exec.jar` with your mainJAR file i.e `/var/lib/obg/open-banking-gateway-1.0.1-SNAPSHOT-exec.jar`

Make the script executable:
```bash
chmod +x usr/local/bin/obg
```

Change ownership to the `obg` user:
```bash
sudo chown -R obg:obg usr/local/bin/obg
```

## 5. Building the Debian Package
Use the following command to build the .deb package:
```bash
dpkg-deb --build obg-deb
```

This will generate `obg-deb.deb`.

## 6. Installing the Package (`obg-deb.deb`)
To install the generated .deb package, run:
```bash
sudo dpkg -i obg-deb.deb
```

Fix any missing dependencies:
```bash
sudo apt-get install -f
```

## 7. Setting Up the Database
Now that the package is installed, set up the PostgreSQL database.

1. **Ensure PostgreSQL is running**:
   ```bash
   sudo systemctl start postgresql
   ```

2. **Export the required environment variables in the respective directory**:
   ```bash
   export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/<your_database>?schema=banking_protocol
   export SPRING_DATASOURCE_USERNAME=postgres
   export SPRING_DATASOURCE_PASSWORD=docker
   ```
    **To make these variables permanent, add them to /etc/environment or a systemd override file.**
3. **Connect to PostgreSQL using an IDE or CLI**:
   - Use any database management tool (DBeaver, pgAdmin, etc.) or connect via the CLI:
     ```bash
     psql -U postgres -d <your_database>
     ```

4. **Create the required schema and extension**:
   ```sql
   CREATE SCHEMA IF NOT EXISTS banking_protocol;
   CREATE EXTENSION IF NOT EXISTS pg_trgm WITH SCHEMA banking_protocol;
   ```
###### **- Note**: Replace `<your_database>` with your `databse name` e.g postgres, ...

## 8. Enabling and Starting the Package
You can start the package using any of the following methods;
### -> As a Package (Recommended)
Once installed, run the following:
```bash
obg
```

### -> As a Service
Once installed, enable and start the service using systemd:
```bash
sudo systemctl daemon-reload
sudo systemctl enable obg
sudo systemctl start obg
```

To check the service status:
```bash
sudo systemctl status obg
```

## 9. Uninstalling the Package
To remove the package, use:
```bash
sudo dpkg -r obg
```

## Conclusion
This guide covers all the steps needed to create a Debian package for Open Banking Gateway, ensuring proper installation, configuration, and service management. This method improves maintainability and simplifies deployment.
