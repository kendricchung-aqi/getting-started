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

### Fixed-size vs. Configurable-size picklists

The System Config app has two tabs for picklist configuration:
- The `Drop-down Lists, Configurable` tab, where you are free to define as many or as few items as you'd like for each list.
- The `Drop-down Lists, Fixed` tab, where the number of items is fixed. You can change the display names, or add translated versions, but if AQTS is expecting a list of 10 items, your customization must still include 10 items.

Most picklists in the system are fixed-sized lists. The lists can change from release to release, so please contact our Support Team to help guide you through this configuration process.

### AQTS 2021.4 picklists

The 2021.4 picklists are:

| PickListKey | PickListType | DisplayName |
|---|---|---|
| BottomEstimateMethodType | Configurable | Bottom Estimate Method |
| ConditionType | Configurable | Condition |
| ControlType | Configurable | Control Type |
| DepthReferenceType | Configurable | Depth Reference |
| FlowOverControlType | Configurable | Flow Over Control Type |
| IceAssemblyType | Configurable | Ice Assembly Type |
| LevelSurveyMethod | Configurable | Level Survey Method |
| NavigationMethodType | Configurable | Navigation Method |
| ReadingQualifierType | Configurable | Reading Qualifier Type |
| SuspensionWeightType | Configurable | Suspension Weight |
| TopEstimateMethodType | Configurable | Top Estimate Method |
| VelocityObservationMethodType | Configurable | Velocity Observation Method |
| AdjustmentType | Fixed | Adjustment Type |
| BaseFlowType | Fixed | Base Flow Type |
| CalibrationCheckType | Fixed | Calibration Check Type |
| ChannelEvennessType | Fixed | Channel Evenness |
| ChannelMaterialType | Fixed | Channel Material |
| ChannelStabilityType | Fixed | Channel Stability |
| ControlCleanedType | Fixed | Control Cleaned Type |
| CrossSectionStartPointType | Fixed | Cross Section Start Point |
| DatumPolarity | Fixed | Datum Polarity |
| DeploymentMethodType | Fixed | Deployment Method |
| DischargeMeasurementReasonType | Fixed | Discharge Measurement Reason |
| DischargeMethodType | Fixed | Discharge Method Type |
| EngineeredStructureType | Fixed | Engineered Structure Type |
| GageHeightCalculationType | Fixed | Gage Height Calculation Type |
| HorizontalFlowType | Fixed | Horizontal Flow Type |
| InspectionType | Fixed | Inspection Type |
| InterpolationType | Fixed | Interpolation Type |
| MeasurementLocationToGageType | Fixed | Measurement Location To Gage |
| MeterSuspensionType | Fixed | Meter Suspension |
| PointVelocityMethodType | Fixed | Point Velocity Method |
| PointVelocityObservationType | Fixed | Point Velocity Observation |
| QualitativeUncertaintyType | Fixed | Qualitative Uncertainty |
| ReadingType | Fixed | Reading Type |
| ReasonForAdjustmentType | Fixed | Reason For Adjustment |
| StartPointType | Fixed | Start Point |
| ThresholdType | Fixed | Threshold Type |
| VelocityMethodType | Fixed | Velocity Method |
| VelocityVariationType | Fixed | Velocity Variation |
| VerticalType | Fixed | Vertical Type |
| VerticalVelocityDistributionType | Fixed | Vertical Velocity Distribution |
| ViewModeType | Fixed | View Mode |

### CSV shape

```csv
PicklistKey, ItemKey, DisplayName, DisplayOrder, Locale

# The Locale column will always default to "en" if omitted.

ConditionType, Unspecified, Nobody knows, 1,
```