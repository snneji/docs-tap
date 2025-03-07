# Log configuration and usage

This topic covers configuring the Supply Chain Security Tools - Store to output detailed log
information and interpret them. re-boot

## <a id='log-lev'></a> Log levels

There are six log levels that the Supply Chain Security Tools - Store supports.

| Level   | Description                                 |
|---------|---------------------------------------------|
| Trace   | Output extended debugging logs              |
| Debug   | Output standard debugging log               |
| More    | Output more verbose informational logs      |
| Default | Output standard informational logs          |
| Less    | Outputs less verbose informational logs     |
| Minimum | Outputs a minimal set of informational logs |

When the Store is deployed at a specific log level, all logs of that level and lower are outputted
to the console. For example, setting the log level to `More` outputs logs from `Minimal` to `More`,
while `Debug` and `Trace` logs are muted.

Currently, the application logs output at these levels:

* **Minimum** does not output any logs.
* **Less** outputs a single log line indicating the current log level the Metadata Store is
configured to when the application starts.
* **Default** outputs API endpoint access information.
* **Debug** outputs API endpoint payload information, both for requests and responses.
* **Trace** outputs verbose debug information about the actual SQL queries for the database.

Other log levels do not output any additional log information and are present for future
extensibility.

If no log level is specified when the Store is installed, the log level is set to `default`.

### <a id='error-logs'></a> Error Logs

Error logs are always outputted regardless of the log level, even when set to `minimum`.

## <a id='obtain-logs'></a> Obtaining logs

Kubernetes pods emit logs. The deployment has two pods: one for the database and one
for the API back end. 

Use `kubectl get pods` to obtain the names of the pods by running:

```console
kubectl get pods -n metadata-store
```

For example:

```console
$ kubectl get pods -n metadata-store
NAME                                  READY   STATUS    RESTARTS   AGE
metadata-store-app-67659bbc66-2rc6k   2/2     Running   0          4d3h
metadata-store-db-64d5b88587-8dns7    1/1     Running   0          4d3h
```

The database pod has prefix `metadata-store-db-` and the API backend pod has the prefix
`metadata-store-app-`. Use `kubectl logs` to get the logs from the pod you're interested in.
For example, to see the logs of the database pod, run:

```console
$ kubectl logs metadata-store-db-64d5b88587-8dns7 -n metadata-store
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.
...
```

The API backend pod has two containers, one for `kube-rbac-proxy`, and the other for the
API server. Use the `--all-containers` flag to see logs from both containers. For example:

```console
$ kubectl logs metadata-store-app-67659bbc66-2rc6k --all-containers -n metadata-store
I1206 18:34:17.686135       1 main.go:150] Reading config file: /etc/kube-rbac-proxy/config-file.yaml
I1206 18:34:17.784900       1 main.go:180] Valid token audiences:
...
```

##  <a id='api-endptlog-out'></a> API endpoint log output

When an API endpoint handles a request, the Store generates two and five log lines. They are:

1. When the endpoint receives a request, it outputs a `Processing request` line. This logline is
shown at the `default` log level.
1. If the endpoint includes query or path parameters, it outputs a `Request parameters` line.
This line logs the parameters passed in the request. This line is shown at the `default` log level.
1. If the endpoint takes in a request body, it outputs a `Request body` line.
This line outputs the entire request body as a string. This line is shown at the `debug` log level.
1. When the endpoint returns a response, it outputs a `Request response` line.
This line is shown at the `default` log level.
1. If the endpoint returns a response body, it outputs a second `Request response` line with an extra
key `payload`, and its value is set to the entire response body. This line is shown at the `debug`
log level.

### <a id='api-endptlog-out-format'></a>  Format

When the Store handles a request, it outputs some API endpoint access information in the following
format:

```console
I1122 20:30:21.869528       1 images.go:26] MetadataStore "msg"="Processing request" "endpoint"="/api/images?digest=sha256%3A20521f76ff3d27f436e03dc666cc97a511bbe71e8e8495f851d0f4bf57b0bab6" "hostname"="metadata-store-app-564f8995c8-r8d6n" "method"="GET"
```

The log is broken down into three sections: The header, name, and key/value pairs.

#### <a id='log-head'></a>  Log header

`I1122 20:30:21.869528       1 images.go:26]` is the logging header.
The [Logging header formats](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/logging.md#logging-header-formats)
section in GitHub explains each part in more detail.

####  <a id='name'></a> Name

The string that follows the header is a name that helps identify what produced the log entry.
For Stores, the name always starts with `MetadataStore`.

For log entries that display the raw SQL queries, the name is `MetadataStore/gorm`.

####  <a id='key-val'></a> Key-value pairs

Key-value pairs compose the rest of the log output. The tables in the following sections list each
key and the meaning of their values.

#####  <a id='common-all'></a> Common to all logs

The following key-value pairs are common for all logs.

| Key      | Type    | Log Level | Description                                                                                                                                                       |
|----------|---------|-----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| msg      | string  | default   | A short description of the logged event                                                                                                                           |
| endpoint | string  | default   | The API endpoint the Metadata Store attempts to handle the request. This also includes any query and path parameters passed in.                              |
| host name | string  | default  | The Kubernetes hostname of the pod handling the request. This helps identify the specific instance of the Store when you deploy multiple instances on a cluster. |
| function | string  | debug     | The function name that handles the request                                                                                                                        |
| method   | string  | default   | The HTTP verb to access the endpoint. For example, "GET" or "POST."                                                                                               |
| code     | integer | default   | The HTTP response code                                                                                                                                            |
| response | string  | default   | The HTTP response in human-readable format. For example, "OK", "Bad Request", or "Internal Server Error."                                                         |
| error    | string  | all       | The error message which is only available in error log entries                                                                                                    |

#####  <a id='log-query'></a> Logging query and path parameter values

Those endpoints that use query or path parameters are logged on the `Request parameters` logline as
key-value pairs. Afterward, they are appended to all other log lines of the same request as
key-value pairs.

The key names are the query or path parameter's name, while the value is set to the value of those
parameters in string format.

For example, the following log line contains the `digest` and `id` key, which represents the
respective `digest` and `id` query parameters, as well as their values:

```console
I1122 20:30:21.869791       1 images.go:34] MetadataStore "msg"="Request parameters" "endpoint"="/api/images?digest=sha256%3A20521f76ff3d27f436e03dc666cc97a511bbe71e8e8495f851d0f4bf57b0bab6" "hostname"="metadata-store-app-564f8995c8-r8d6n" "method"="GET" "digest"="sha256:20521f76ff3d27f436e03dc666cc97a511bbe71e8e8495f851d0f4bf57b0bab6" "id"=0
```

These key/value pairs show up in all subsequent log lines of the same call. For example:

```console
I1122 20:30:21.878749       1 images.go:56] MetadataStore "msg"="Request response" "digest"="sha256:20521f76ff3d27f436e03dc666cc97a511bbe71e8e8495f851d0f4bf57b0bab6" "endpoint"="/api/images?digest=sha256%3A20521f76ff3d27f436e03dc666cc97a511bbe71e8e8495f851d0f4bf57b0bab6" "hostname"="metadata-store-app-564f8995c8-r8d6n" "id"=0 "method"="GET" "code"=200 "response"="OK"
```

This is done to ensure:

* The application interprets the values of the query or path parameters correctly.
* Help figure out which log lines are associated with a particular API request.
Since there can be several simultaneous endpoint calls, this is a first attempt at grouping
logs by specific calls.

#####  <a id='api-payload-out'></a> API payload log output

As mentioned at the start of this section, by setting the log level to `debug`, the Store logs the
body payload data for both the request and response of an API call.

The `debug` log level, instead of the `default`, is used to display this information instead of `default`
because:

* Body payloads can be huge, containing full CycloneDX and SBOM information.
Moving the payload information at this level helps keep the production log output to a reasonable size.
* Some information in these payloads may be sensitive, and the user may not want them exposed in
production environment logs.

##  <a id='sql_query-out'></a> SQL Query log output

Some Store logs display the executed SQL query commands when you set the log level to `trace` or a
failed SQL call occurs.

>**Note:** Some information in these SQL Query trace logs might be sensitive, and the user might not
want them exposed in production environment logs.

###  <a id='sql_query-out-format'></a> Format

When the Store display SQL query logs, it uses the following format:

```console
I0111 20:14:30.816833       1 connection.go:40] MetadataStore/gorm "msg"="Sql Call" "hostname"="metadata-store-app-56799fc4f9-phlv7" "rows"=1 "sql"="SELECT count(*) FROM information_schema.tables WHERE table_schema = CURRENT_SCHEMA() AND table_name = 'images' AND table_type = 'BASE TABLE'"
```

It is similar to the [API endpoint log output](#api-endpoint-log-output) format, but also uses the
following key-value pairs:

| Key   | Type    | Log Level | Description                                                  |
| ----- | ------- | --------- | ------------------------------------------------------------ |
| rows  | integer | trace     | Indicates the number of rows affected by the SQL query       |
| sql   | string  | trace     | Displays the raw SQL query for the database                  |
| data# | string  | all       | Used in error log entries. You can replace `#` with an integer because multiples of these keys can appear in the same log entry. These keys contain extra information related to the error. |
