Get-Partition -DiskNumber 2 | Format-List PartitionNumber,DriveLetter,Type,Size,AccessPaths


Get-Volume | Format-Table DriveLetter,FileSystemLabel,FileSystem,Size,HealthStatus