## Connecting Cloudflare R2 to Databricks

To connect Cloudflare R2 as external storage in Databricks:

1. Go to **Catalog**.
2. Open **Settings**.
3. Select **External Locations**.
4. Click **Create external location**.
5. Enter an **External Location Name**.
6. Select **R2** as the storage type.
7. Create a new **Storage Credential**.
8. Enter the required R2 credentials:

   * Account ID
   * Access Key
   * Secret Key
9. Test the connection.

### Connection Test

If the configuration is correct, Databricks displays:

```text
Test connection

Location Type: Directory

Success
 - Read

Success
 - List

Success
 - Write

Success
 - Delete

Success
 - Path Exists

All Permissions Confirmed

The associated Storage Credential grants permission
to perform all necessary operations.
```

This confirms that the storage credential has the required permissions to interact with the external location.

### Workflow

```text
Cloudflare R2
     ↓
Storage Credential
     ↓
External Location
     ↓
Databricks
```
