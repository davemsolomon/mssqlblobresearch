######################################SQL TO CREATE TABLE STRUCTURE##################
1> CREATE TABLE mytable (
2>     ID INT PRIMARY KEY,
3>     BinaryData VARBINARY(MAX)
4> );
5> go
#######################################################################################

#####################################POWERSHELL TO PUSH TIF INTO DB#############################
$filePath = "C:\tmp\smile.tif"
$connectionString = "Server=localhost\SQLEXPRESS;Database=master;Integrated Security=True;"

$fileBytes = [System.IO.File]::ReadAllBytes($filePath)

$connection = New-Object System.Data.SqlClient.SqlConnection($connectionString)
$connection.Open()

$command = $connection.CreateCommand()
$command.CommandText = "INSERT INTO mytable (ID, BinaryData) VALUES (@ID, @BinaryData)"
$command.Parameters.Add("@ID", [System.Data.SqlDbType]::Int).Value = 1
$command.Parameters.Add("@BinaryData", [System.Data.SqlDbType]::VarBinary, -1).Value = $fileBytes

$command.ExecuteNonQuery()
$connection.Close()
#################################################################################################

######################################SQL TO PROVE TIF IN DB SUCCESSFULLY#######################
1> select * from mytable;
2> go
ID          BinaryData
----------- ----------
          1 0x49492A000x0ECBC533

(1 rows affected)
###############################################################################################

#########################################POWERSHELL TO DOwNLOAD TIF###########################
$outputPath = "C:\tmp\smile_retrieved.tif"
$connectionString = "Server=localhost\SQLEXPRESS;Database=YourDatabaseName;Integrated Security=True;"

$connection = New-Object System.Data.SqlClient.SqlConnection($connectionString)
$connection.Open()

$command = $connection.CreateCommand()
$command.CommandText = "SELECT BinaryData FROM mytable WHERE ID = 1"

$reader = $command.ExecuteReader()
$reader.Read()

$fileBytes = $reader.GetSqlBytes(0).Value
[System.IO.File]::WriteAllBytes($outputPath, $fileBytes)

$reader.Close()
$connection.Close()

Write-Host "Done! File saved to $outputPath"
#############################################################################################
