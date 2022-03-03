# As of ATQS 2021.4, you probably have no need to run the `ExtendedAttributeSchema` task

But for customers/integrators running earlier AQTS versions, the task does enable some workflows that are otherwise not possible.

# ProvisioningTool - `ExtendedAttributeSchema` tasks

- CREATE and UPDATE operations are supported and function identically. They just execute the SQL statements in the provided file.

### Database credentials are required for extended attribute schema configuration

The `ExtendedAttributeSchema` tasks require direct access to the AQTS database.

See the [[Database Credentials]] topic for more details.

The simplest thing to do is run ProvisioningTool directly on the AQTS app server, to allow automatic discovery of the correct database configuration settings.

## Why is this a dangerous operation?

This task allows you to execute arbitrary SQL statements against your AQTS database. That is a huge security and support risk. You can absolutely wipe out your entire organization's data with this task.

With great power comes great ... yada yada yada ... 

So if you mess up your DB with this task, that's on you, not us!