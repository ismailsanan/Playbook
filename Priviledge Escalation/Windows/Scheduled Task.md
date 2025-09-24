
basically cronjob for windows can be time based , event based and custom

```sh
Get-ScheduledTask

#cmd
schtasks /query

#details about the Task

Get-ScheduledTaks -TaskName "Simple" | Get-ScheduledTaskInfo

schtasks /query /FO LIST /V

# More detials 
Get-ScheduledTaks -TaskName "Simple"  | Format-List *

#extract bin path and arguments 
(Get-ScheduledTask -TaskName "Simple").Actions

#Details of the task
Export-ScheduledTaks -TaskName "Simple"  -TaskPath "C:\XXX"
```

