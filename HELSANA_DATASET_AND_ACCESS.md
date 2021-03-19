# How to access the Helsana data

We prepared a decent data-set with more then 190'000 activities from 1900 users. The data is stored in one single Azure SQL table.

We privide you two ways of accessing this data to address various use-cases.

1. Direct SQL data-access (read-only) for data-analytics or backend access.
1. REST API to retrieve the data from within apps or backend.

## The data structure

To keep things simple we provide one single datbase table. Note that this is a small part of data we have for Helsana+ and its build specially for this Hackaton. It's not our real world data-structure but it should give you a usable dataset to work with without introduce a huge complexity. Its up to you what of this data you like to use in your Hack.

Fields and their meaning:

- ``Id`` (GUID) &mdash; Unique Id for each activitiy.
- ``UserId`` (string) &mdash; User-Id who did the activity.
- ``Gender`` (string) &mdash; The gender of the user as a string.
- ``Age`` (integer) &mdash; The age of the user as number of years (integer).
- ``ActivityTime`` (string, date-time) &mdash; The time when the activity happend.
- ``BasicActivity`` (string) -- The Helsana+ activity this activity represent.
- ``ImageClass`` (string) -- The kind of image/photo/screenshot that was submitted with this activity.
- ``RecognizedActivity`` (string) &mdash; The kind of sport/movement done as part of this activity.
- ``LabelsJson`` (string, JSON) &mdash; A JSON structure with information what labels/tags we found on the image/photo/screenshot sent with this activity. It should give you in idea what was visible on the photo what the user sent.
- ``ScreenshotFindingsJson`` (string, JSON) &mdash; Basic information in JSON what Helsana recognized in the screenshots of the apps for number of steps, calories and distance.
- ``ScreenshotFindingsDetailsJson`` (string, JSON) &mdash; Detailed information about the screenshot image with bounding boxes and texts what we found using AI.
- ``ActivityDetails`` (string, JSON) &mdash; Additonal data (payload) in JSON depending on the *BasicActivity* field.

## Direct SQL read-only access

Use the database tools or libraries of your choice to connect to our Azure SQL database (mostly compatible with MS-SQL). You may use MS-SQL data-drivers to access this database.

- Server: hhack-sqlserver.database.windows.net
- Port: 1433
- Catalog/Database: hhack_database
- User: hhackreader
- Passwort: GetMeData!

Here are some connection strings you may like to use:

**ADO .NET:**
```
Server=tcp:hhack-sqlserver.database.windows.net,1433;Initial Catalog=hhack_database;Persist Security Info=False;User ID=hhackreader;Password=GetMeData!;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

**JDBC:**
```
jdbc:sqlserver://hhack-sqlserver.database.windows.net:1433;database=hhack_database;user=hhackreader@hhack-sqlserver;password=GetMeData!;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;
```

**ODBC:**
```
Driver={ODBC Driver 13 for SQL Server};Server=tcp:hhack-sqlserver.database.windows.net,1433;Database=hhackreader;Uid=hhackadmin;Pwd=GetMeData!;Encrypt=yes;TrustServerCertificate=no;Connection Timeout=30;
```

**PHP:**
```
// PHP Data Objects(PDO) Sample Code:
try {
    $conn = new PDO("sqlsrv:server = tcp:hhack-sqlserver.database.windows.net,1433; Database = hhack_database", "hhackreader", "GetMeData!");
    $conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
}
catch (PDOException $e) {
    print("Error connecting to SQL Server.");
    die(print_r($e));
}

// SQL Server Extension Sample Code:
$connectionInfo = array("UID" => "hhackreader", "pwd" => "GetMeData!", "Database" => "hhack_database", "LoginTimeout" => 30, "Encrypt" => 1, "TrustServerCertificate" => 0);
$serverName = "tcp:hhack-sqlserver.database.windows.net,1433";
$conn = sqlsrv_connect($serverName, $connectionInfo);
?>
```

**GO:**
```
// Go connection Sample Code:
package main
import (
	github.com/denisenkom/go-mssqldb
	database/sql
	context
	log
	fmt
	errors
)

var db *sql.DB
var server = "hhack-sqlserver.database.windows.net"
var port = 1433
var user = "hhackreader"
var password = "GetMeData!"
var database = "hhack_database"

func main() {
	// Build connection string
	connString := fmt.Sprintf("server=%s;user id=%s;password=%s;port=%d;database=%s;",
		server, user, password, port, database)
	var err error
	// Create connection pool
	db, err = sql.Open("sqlserver", connString)
	if err != nil {
		log.Fatal("Error creating connection pool: ", err.Error())
	}
	ctx := context.Background()
	err = db.PingContext(ctx)
	if err != nil {
		log.Fatal(err.Error())
	}
	fmt.Printf("Connected!")
}
```


## REST API access

We provide you with a simple REST API to query the above data. It contains two GET endpoints. One to query user-id's using various criterias and one to query the activities.

Both endpoints needs the following API key present in header of the request:

- API-Key Header name: ``x-functions-key: WJpDAQqpIbZNa7ANLrlZIzShYYUszrfRNMbdjQv6g66RdW1JLaVAaQ==``
- Base Url: https://hackapi.azurewebsites.net

Here is a sample to test the connectivity by getting all user-id's present in the dataset:

**cURL:**
```
curl --location --request GET 'https://hackapi.azurewebsites.net/api/users' \
--header 'x-functions-key: WJpDAQqpIbZNa7ANLrlZIzShYYUszrfRNMbdjQv6g66RdW1JLaVAaQ=='
```

**Power-Shell:**
```
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("x-functions-key", "WJpDAQqpIbZNa7ANLrlZIzShYYUszrfRNMbdjQv6g66RdW1JLaVAaQ==")

$response = Invoke-RestMethod 'https://hackapi.azurewebsites.net/api/users' -Method 'GET' -Headers $headers
$response | ConvertTo-Json
```

### Query users

Allows you to query user-id's with various optinoal filters.

**Url:** https://hackapi.azurewebsites.net/api/users
Verb: GET

Filters are provided using query parameters:

- ``fromAge`` (integer) &mdash; Range from age in years
- ``toAge`` (integer) &mdash; Range to age in years
- ``gender`` (string) &mdash; The content of the gender field (mostly ``female`` or ``male``).

**Example: All females in the age of 30**

```
curl --location --request GET 'https://hackapi.azurewebsites.net/api/users?fromAge=30&toAge=30&gender=female' \
--header 'x-functions-key: WJpDAQqpIbZNa7ANLrlZIzShYYUszrfRNMbdjQv6g66RdW1JLaVAaQ=='
```

### Query activities

Retrieve a set of activities using this endpoint. Query all activitiy at once is not possible due to the among of data that this would result in.

**Url:** https://hackapi.azurewebsites.net/api/activities

Filters are provided as query parameters:

- ``userId`` (string) &mdash; Id of the user to get the activities for.
- ``fromActivityTime`` (string/datetime) &mdash; Range from date/time of activitiy. Eg ``2021-01-13``.
- ``toActivityTime`` (string/dateimte) &mdash; Range to data/time of activity.
- ``imageClass`` (string) &mdash; Content of field ImageClass which specifies the kind of activity-image this activity was submitted with. It's only available for activities which where sent as photo or screenshot.
- ``basicActivity`` (string) &mdash; The Helsana+ type of activity done. Eg "Personal exercise", "Session mindfulness Coach", ...
- ``recognizedActivity`` (string) &mdash; The classification of the work/sport/activity done as part of this activity. Eg. "Walk or Jog", "HomeWorkout", ...

**Example to get all activities done by a certain user-id**

```
curl --location --request GET 'https://hackapi.azurewebsites.net/api/activities?userId=01E8091B7D4F004EFB77FA332F662C20' \
--header 'x-functions-key: WJpDAQqpIbZNa7ANLrlZIzShYYUszrfRNMbdjQv6g66RdW1JLaVAaQ=='
```
