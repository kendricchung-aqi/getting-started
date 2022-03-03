# ProvisioningTool - `PicklistDisplayItem` tasks

- CREATE, UPDATE, and DELETE operations are supported.

# Configuring your AQTS picklists

The `PicklistDisplayItem` tasks are used to configure the dropdown menus visible in the AQTS system (typically in the browser web forms).

These tasks configure the `PicklistDisplayItem` database table, which stores a localizable list of values.

The AQUARIUS browser apps (Springboard, Field Visit, Location Manager) try to use the current language definition for a list item, falling back to the English item if no language-specific item exists.

### Database credentials are required for picklist configuration

The `PicklistDisplayItem` tasks require direct access to the AQTS database, since the Provisioning API doesn't yet expose the locale-specific information.

See the [[Database Credentials]] topic for more details.

The simplest thing to do is run ProvisioningTool directly on the AQTS app server, to allow automatic discovery of the correct database configuration settings.

### Fixed-size vs. Free-form picklists

Picklists fall into one of two categories:
- **Freeform lists**, where you are free to define as many or as few items as you'd like.
- **Fixed-sized lists**, where the number of items is fixed. You can change the display names, or add translated versions, but if AQTS is expecting a list of 10 items, your customization must still include 10 items.

Most picklists in the system are fixed-sized lists. The lists can change from release to release, so please contact our Support Team to help guide you through this configuration process.

### AQTS 2018.4 free-form lists

The 2018.4 free-form lists are these `PicklistKey` values:

```
ConditionType
ControlType
DriftCheckType
FlowOverControlType
IceAssemblyType
LevelSurveyMethod
ReadingQualifierType
SuspensionWeightType
ThresholdName
VelocityObservationMethodType
ViewModeType
```

All other picklists are fixed sized lists.

### CSV shape

```csv
PicklistKey, ItemKey, DisplayName, DisplayOrder, Locale

# The Locale column will always default to "en" if omitted.

ConditionType, Unspecified, Nobody knows, 1,
```