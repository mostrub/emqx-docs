# Schema Validation

EMQX includes built-in schema validation capabilities to ensure that only messages conforming to predefined data formats are published to subscribers from specified topics. Schema validation supports multiple schema formats such as JSON Schema, Protobuf, and Avro and built-in SQL statement validation. This page describes the schema validation feature and how to use it.

## Why Validate Data

Clients may publish non-standard messages to the Broker, which could lead to exceptions in subscribers and data systems or pose security risks. EMQX can identify and block these non-compliant messages by validating data formats early, ensuring system stability and reliability. schema validation brings benefits in the following aspects:

- **Data Integrity**: Validates the structure and format of MQTT messages to ensure data consistency and correctness.
- **Data Quality**: Enforces data quality by checking for missing or invalid fields, data types, and formats, ensuring data consistency and quality.
- **Unified Data Model**: Ensures that the entire team and project use a unified data model, reducing inconsistencies and errors.
- **Reuse and Sharing**: Allows team members to reuse and share schemas, improving collaboration efficiency and reducing repetitive work and errors.
- **Security**: Prevents malicious or incorrectly formatted messages from being processed, reducing the risk of security vulnerabilities.
- **Interoperability**: Ensures messages conform to standardized formats, facilitating communication between different devices and systems.
- **Debugging**: Easily identify and debug invalid or incorrectly formatted messages.

## Workflow

When a message is published, it is validated against predefined rules. If validation succeeds, the process continues; otherwise, user-configured actions are executed, such as message discarding or disconnection.

1. When a message is published, the publish permissions are first checked through the [authorization mechanism](../access-control/authz/authz.md) in EMQX. After passing the permission check, validation rules are matched based on the publishing topic from a user-configured list of validators. A validator can be set for multiple topics or topic filters.

2. Once a validation rule is matched, the message is validated against the preset Schema or SQL.

   - Supports multiple types of Schema: JSON Schema, Protobuf, and Avro.
   - Supports SQL statements that comply with EMQX rule engine syntax.
   - A single policy can add multiple Schemas or SQLs and specify their relationships:
     - **All Pass**: Validation is considered successful only if all validations pass.
     - **Any Pass**: Validation stops and is considered successful if any validation passes.

3. Once validated successfully, the message continues to the next process, such as triggering the rule engine or dispatching to subscribers.

4. If validation fails, the following user-configured actions can be executed:

   - **Discard Message**: Terminate the publish and discard the message, returning a specific reason code (131 - Implementation Specific Error) for QoS 1 and QoS 2 messages via PUBACK.
   - **Disconnect and Discard Message**: Discard the message and disconnect the publishing client.
   - **Ignore**: No additional actions are taken.

   Regardless of the configured action, a log entry can be generated upon validation failure; users can configure the log's output level, which defaults to `warning`. A validation failure can also trigger a rule engine event `$events/schema_validation_failed`, allowing users to catch this event for custom handling, such as publishing the erroneous message to another topic or sending it to Kafka for analysis.

## User Guide

This section demonstrates how to configure the schema validation feature and how to test your setup. 

### Configure Schema Validation in Dashboard

This section demonstrates how to create and configure a schema validator in the Dashboard.

1. Click on **Integrations** -> **Schema Validation** in the left navigation of the Dashboard.
2. Click **Create** at the top right of the **Schema Validation** page.
3. On the Create Schema Validation page, configure the following information:
   - **Name**: Enter the name of the validator.
   - **Message Source Topic**: Set the topics whose messages need to be validated. Multiple topics or topic filters can be set.
   - **Note** (optional): Enter any notes.
   - **Validation Method**:
     - **Validation Strategy**: Specify the relationship between multiple validation strategies.
       - **All Pass** (by default): Considered passing only when all validation methods pass.
       - **Any Pass**: Stops further validation and is considered passing if any validation method passes.
     - **Validation List**: select the schema from the **Type** dropdown, and add schema or SQL. For how to create a schema, see [Create Validation Schema](#create-validation-schema).
   - **Validation Failure Operation**: 
     - **Action After Failure**: Select the actions to perform if validation fails:
       - **Drop Message**: Terminate the publish and discard the message, returning a specific reason code for QoS 1 and QoS 2 messages via PUBACK.
       - **Disconnect and Drop Message**: Discard the message and disconnect the publishing client.
       - **Ignore**: Perform no additional actions.
   - **Output Logs**: Select whether to log a message upon validation failure; default is enabled.
   - **Logs Level**: Set the log output level; the default is `warning`.
   
4. Click **Create** to complete the settings.

Now you can see an enabled new validator appears in the list on the Schema Validation page. You can disable it as you need. You can update the validator settings by clicking **Settings** in the **Actions** column. You can also delete the validator or move the position of the validator by clicking **More**.

### Configure Schema Validation in Configuration File

For configuration details, see [Configuration Manual](https://docs.emqx.com/en/enterprise/v@EE_VERSION@/hocon/).

### Create Validation Schema

This section demonstrates how to create a validation schema, using JSON Schema as an example. The JSON Schema must meet the following requirements:

- The JSON object must include a property named `temp`.
- The `temp` property must be an integer.
- The `temp` property must be at least 101.

```json
{
  "$schema": "http://json-schema.org/draft-06/schema#",
  "type": "object",
  "properties": {
    "temp": {
      "type": "integer",
      "minimum": 101
    }
  },
  "required": ["temp"]
}
```

### Test Schema Validation Setup

You can use the example schema created in [Create Validation Schema](#create-validation-schema) to test your schema validation setup.

Use [mqttx](https://mqttx.app/cli) to publish a message with a payload that conforms to the MQTT message rules:

```bash
mqttx pub -t t/1 -m '{"temp": 102}'
```

Use mqttx to publish a message with a payload that does not conform to the MQTT message rules:

```bash
mqttx pub -t t/1 -m '{"temp": 100}'
```

The log output should look like the following:

```bash
2024-05-16T06:24:10.733827+00:00 [warning] tag: SCHEMA_VALIDATION, clientid: mqttx_1db4547e, msg: validation_failed, peername: 127.0.0.1:40850, action: drop, validation: <<"check-json">>
```

### REST API

For detailed information on how to use schema validation through the REST API, see [EMQX Enterprise API](https://docs.emqx.com/en/enterprise/v@EE_MINOR_VERSION@/admin/api-docs.html).

## Statistics and Indicators

When enabled, the schema validation exposes statistics and indicators on the Dashboard. You can click the name of the validator on the Schema Validation page to see the following:

**Statistics**:

- **Total**: The total number of triggers since the system started.
- **Success**: Number of successful data validations.
- **Failed**: Number of failed data validations.

**Rate Indicators**:

- Current verification speed
- Seed in the last 5 minutes
- Historical maximum speed

Statistics are resettable and also added to Prometheus, accessible via the `/prometheus/schema_validation` path.