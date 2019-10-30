## GeoLite2MySQL
Powershell script to import MaxMinds GeoLite2 data into MySQL

## FUNCTIONALITY
1) If geoip table does not exist, it gets created
2) Deletes old files, renames previously "new" "old" in order to compare
3) Downloads MaxMinds geolite2 cvs data as zip file, uncompresses it, then renames the folder
4) Compares new and old data for incremental changes
5) Reads IPv4 cvs data, then calculates the lowest and highest IP from each network in the database
6) Deletes obsolete records
7) Inserts lowest and highest IP in range and geoname_id from IPv4 cvs file
8) Reads geo-name cvs file and updates each record with country code and country name based on the geoname_id
9) Creates scheduled task for weekly updates
10) Includes various error checking to keep from blowing up a working database on error
11) Email notification on error

## INSTRUCTIONS
1) Create folder to contain scripts and MaxMinds data
2) Modify user variables in GeoLite2MySQL.ps1
3) First time run from Powershell console

## NOTES
Run every Wednesday via task scheduler (MaxMinds releases updates on Tuesdays)

Initial loading of the database takes a LONG time, about 2 hours on my old hardware (338k+ records) - subsequent updates are incremental, so they only take a few minutes
	
## EXAMPLE QUERY
Returns countrycode and countryname from a given IP address:
	
```
SELECT countrycode, countryname FROM (SELECT * FROM geo_ip WHERE INET_ATON('125.64.94.220') <= maxipaton LIMIT 1) AS A WHERE minipaton <= INET_ATON('125.64.94.220')
```

## hMailServer VBS
Subroutine (credit to SorenR for error checking, among other things):
```
Sub GeoIPLookup(ByVal sIPAddress, ByRef m_CountryCode, ByRef m_CountryName)
    Dim oRecord, oConn : Set oConn = CreateObject("ADODB.Connection")
    oConn.Open "Driver={MariaDB ODBC 3.0 Driver}; Server=localhost; Database=geoip; User=geoip; Password=nnPCGiO3DhddUeJm;"

    If oConn.State <> 1 Then
'       EventLog.Write( "Sub GeoIPLookup - ERROR: Could not connect to database" )
        WScript.Echo( "Sub GeoIPLookup - ERROR: Could not connect to database" )
        m_CountryCode = "XX"
        m_CountryName = "ERROR"
        Exit Sub
    End If

    m_CountryCode = "NX"
    m_CountryName = "NOT FOUND"

    Set oRecord = oConn.Execute("SELECT countrycode, countryname FROM (SELECT * FROM geo_ip WHERE INET_ATON('" & sIPAddress & "') <= maxipaton LIMIT 1) AS A WHERE minipaton <= INET_ATON('" & sIPAddress & "')")
    Do Until oRecord.EOF
        m_CountryCode = oRecord("countrycode")
        m_CountryName = oRecord("countryname")
        oRecord.MoveNext
    Loop
    oConn.Close
    Set oRecord = Nothing
End Sub
```

Call Sub:
```
'	GeoIP Lookup
Dim m_CountryCode, m_CountryName
Call GeoIPLookup(oClient.IPAddress, m_CountryCode, m_CountryName)
```

## HISTORY
- v.07 added column width formatting to email notification plus more and useful information; email report is meaningful more than "success/fail"
- v.06 housekeeping
- v.05 added create scheduled task and email notification
- v.04 bug fixes
- v.03 bug fixes
- v.02 added incremental update and error checking
- v.01 first commit
