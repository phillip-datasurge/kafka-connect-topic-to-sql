# Testing & Verification

1. Execute the command: `docker compose up --build` or `docker-compose up --build`
2. Navigate to the control center: http://localhost:9021
3. Upload the connector file (sqlserver-sink-connector.json)
4. Execute the following KSQLDB queries (note: AVRO schema will automatically be generated):

```
CREATE OR REPLACE STREAM sample_first_stream (
    ROWKEY VARCHAR KEY,
    ID VARCHAR,
    CODE INT,
    MESSAGE VARCHAR
) WITH (
    KAFKA_TOPIC = 'sample.first',
    VALUE_FORMAT = 'JSON'
);
```

```
CREATE OR REPLACE STREAM sample_first_stream_derived 
WITH (
    KAFKA_TOPIC = 'sample.second',
    VALUE_FORMAT = 'AVRO'
) AS
SELECT 
    ROWKEY,
    id,
    code,
    message
FROM sample_first_stream
WHERE code = 201;
```

5. Add a JSON message to the `sample.first` topic, for example:

```
{
  "id": "sdsiudhsi",
  "code": 201,
  "message": "test"
}
```

6. Verify the JSON message was converted to AVRO in the `sample.second` topic, for example:

**NOTE**: Code must be 201 to be forwarded to the `sample.second` topic (see KSQL `WHERE` clause)

```
{
  "ID": {
    "string": "sdsiudhsi"
  },
  "CODE": {
    "int": 201
  },
  "MESSAGE": {
    "string": "test"
  }
}
```

7. Verify the record was persisted in the SQL Server docker container:

| Property    | Value |
| -------- | ------- |
| JDBC URL  | jdbc:sqlserver://127.0.0.1:1433;databaseName=master;encrypt=false    |
| Username  | sa    |
| Password | Password1234!     |
| Database    | master    |
| Schema    | dbo    |
| Table    | SampleMessages    |

![Screenshot](https://i.ibb.co/D7PK6D3/Screenshot-2024-12-17-111644.png)