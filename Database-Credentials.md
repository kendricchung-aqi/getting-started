# ProvisioningTool - Database Credentials

Most of the ProvisioningTool tasks use the Provisioning API to configure the specified items.

But some tasks also need direct access to the AQTS database, in order to make their changes, since the APIs don't directly support all the required functionality.

## Tasks requiring direct database access

- [[PicklistDisplayItem]]
- [[ExtendedAttributeSchema]]
- [[RepairTimeSeries]]
- [[PanelCount]]

## AQCloud customers cannot run tasks requiring direct database access

Customers running an AQUARIUS application server hosted at aquaticinformatics.dev will not be able to run any of the ProvisioningTool tasks which require direct database access. Please raise a ticket in our Support portal to have the Aquatic Informatics Support or Professional Services teams run these tasks on your behalf.

## Run the ProvisioningTool directly on the AQTS application server for simplicity

When you run the ProvisioningTool directly on the AQTS app server, any task which requires direct DB access can automatically figure out the correct database connection settings, and you won't be required to specify any extra command line options.

## Or add your Windows domain account as an Administrator of the AQTS app server

If your Windows domain account is configured in the Adminstrators group of the AQTS app server, then the ProvisioningTool will be able to read the `\\server\C$\ProgramData\Aquatic Informatics\AQUARIUS\AquariusDataSource.xml` file over the network, and figure out the the database connection settings automatically.

### Database command line options

The `/DbFilename=`, `/DbType=`, and `/DbConnectionString=` command-line options allow you to explicitly provide database credentials when they cannot be inferred from the `/Server=` context.

If you are running `ProvisioningTool.exe` directly on the AQTS app server, or if the `\\server\C$\ProgramData\Aquatic Informatics\AQUARIUS\AquariusDataSource.xml` file is readable over a network share by your Windows account, then the tool should be able to automatically infer the DB credentials for you.

Try to use the automatically inferred DB credentials before manually setting the `/DbFilename=`, `/DbType=`, or `/DbConnectionString=` options.
