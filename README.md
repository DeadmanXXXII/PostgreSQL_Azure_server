# **Azure PostgreSQL Security Setup**

#### **1. Provisioning an Azure PostgreSQL Database Instance**

**1.1 Create a Resource Group:**
```bash
az group create --name MyPostgreSQLResourceGroup --location eastus
```

**1.2 Create an Azure PostgreSQL Server:**
```bash
az postgres server create --resource-group MyPostgreSQLResourceGroup --name mypostgresqlserver --location eastus --admin-user myadmin --admin-password MyP@ssword123 --sku-name B_Gen5_2
```

**1.3 Create a PostgreSQL Database:**
```bash
az postgres db create --resource-group MyPostgreSQLResourceGroup --server-name mypostgresqlserver --name mydatabase
```

#### **2. Security Configurations**

**2.1 Configure Firewall Rules:**
```bash
az postgres server firewall-rule create --resource-group MyPostgreSQLResourceGroup --server-name mypostgresqlserver --name AllowAllIps --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

**2.2 Configure Virtual Network Service Endpoints:**
```bash
az network vnet subnet update --resource-group MyPostgreSQLResourceGroup --vnet-name myVNet --name default --service-endpoints Microsoft.DBforPostgreSQL
```

**2.3 Enable Advanced Threat Protection:**
```bash
az postgres server threat-policy update --resource-group MyPostgreSQLResourceGroup --server-name mypostgresqlserver --state Enabled --email-addresses your-email@example.com
```

**2.4 Configure Auditing and Logging:**
- **Auditing:** Enable auditing to log database activities. You can configure this in the Azure Portal under the PostgreSQL server settings. Logs are available in Azure Monitor.
- **Logging:** Enable logging for monitoring and troubleshooting. This can also be configured in the Azure Portal under PostgreSQL settings.

**2.5 Enable SSL/TLS Connections:**
- Enforce SSL/TLS for connections by setting `sslmode=require` in your PostgreSQL client connection string.

**2.6 Enable Multi-Factor Authentication (MFA):**
- Configure MFA for the Azure Active Directory used to manage your PostgreSQL server.
  - Go to Azure Active Directory > Users > Select user > Multi-Factor Authentication.

**2.7 Use Managed Identities:**
- Assign a managed identity to Azure resources needing access to Azure PostgreSQL for secure authentication.
  - In the Azure Portal, go to the resource (e.g., Azure VM) > Identity > Enable system-assigned managed identity.

**2.8 Implement Network Security Groups (NSGs):**
- Create and configure NSGs to control inbound and outbound traffic to your PostgreSQL server.
  ```bash
  az network nsg create --resource-group MyPostgreSQLResourceGroup --name myNSG
  az network nsg rule create --resource-group MyPostgreSQLResourceGroup --nsg-name myNSG --name AllowPostgreSQL --protocol Tcp --direction Inbound --priority 1000 --source-address-prefixes VirtualNetwork --source-port-ranges '*' --destination-address-prefixes '*' --destination-port-ranges 5432 --access Allow
  ```

**2.9 Use Private Link:**
- Configure Private Link to ensure secure connectivity to your PostgreSQL server from your virtual network.
  ```bash
  az network private-endpoint create --resource-group MyPostgreSQLResourceGroup --name myPrivateEndpoint --vnet-name myVNet --subnet default --private-connection-resource-id /subscriptions/{subscription-id}/resourceGroups/MyPostgreSQLResourceGroup/providers/Microsoft.DBforPostgreSQL/servers/mypostgresqlserver --group-id sqlServer
  ```

**2.10 Enable Encryption at Rest:**
- Azure PostgreSQL uses Transparent Data Encryption (TDE) by default, but you can configure customer-managed keys for additional control.
  - Configure Key Vault to manage encryption keys and link it with your PostgreSQL server for enhanced security.

**2.11 Regularly Review and Rotate Credentials:**
- Use Azure Key Vault to manage and periodically rotate administrative and user passwords.

**2.12 Use PostgreSQL Internal Firewalls:**
- Restrict access by configuring `pg_hba.conf` file within PostgreSQL to allow only specific IP addresses.
  ```bash
  # Example entry in pg_hba.conf
  host all all                            192.168.1.0/24 md5
  ```

### **Summary**

This setup guide for Azure PostgreSQL includes provisioning a server, configuring firewall and network security, enabling advanced threat protection and auditing, enforcing SSL/TLS, using managed identities, and implementing private links. Additionally, it covers encryption, credential management, and internal PostgreSQL firewalls to enhance the overall security of your database instance. Regular updates and audits are essential to maintaining this security posture.
