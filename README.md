![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Rebuild All Indexes Across All SQL Databases And All Tables
**Post Date: March 31, 2015**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Here's some SQL logic I wrote up to rebuild ALL the indexes across ALL tables. Later I may include some extra code to identify those indexes that are fragmented above 70% to be REBUILD, and those indexes that are below 70% to be REORGANIZE. This script is not set to rebuild and leave tables online. For this you need to add the *WITH (ONLINE = ON); therefore you should run this as general maintenance after hours. It's just much faster to rebuild directly without using the ONLINE = ON. I'm not against doing it online, but I do advocate for the quickest approach normally. This process will target tables that are reflected as BASE TABLE from information_schema.tables. Additionally; it will not target the system databases so Master, Model, Msdb, and Tempdb are all excluded.
You will notice some replacements going on. This is so the logic responsible for producing the rebuild statement will create the appropriate variables adding the database name as the suffix to ensure uniqueness. Obviously; if the database name that it's adding to the variable name has any spaces, dots(.), etc. it will break the logical flow so to address this I simply replace the spaces with underscores, and the dots(.) also with underscores. Easy.
On with the logic.</p>      


## SQL-Logic
```SQL
use master;
set nocount on
 
declare @index_maintenance  varchar(max)
set @index_maintenance  = ''
select  @index_maintenance  = @index_maintenance +
'use [' + sd.name + '];' + char(10) +
'declare    @rebuild_indexes_in_' + replace(replace(sd.name, ' ', '_'), '.', '_') + '   varchar(max)'   + char(10) +
'set        @rebuild_indexes_in_' + replace(replace(sd.name, ' ', '_'), '.', '_') + '   = '''''         + char(10) +
'select @rebuild_indexes_in_' + replace(replace(sd.name, ' ', '_'), '.', '_') + '   = @rebuild_indexes_in_' + replace(replace(sd.name, ' ', '_'), '.', '_') + ' +
''alter index all on [dbo].['' + object_name(sddips.object_id) + ''] rebuild;'' + char(10)
from sys.dm_db_index_physical_stats (db_id(), null, null, null, null) sddips
join information_schema.TABLES ist on object_name(sddips.object_id) = ist.table_name
where ist.table_type = ''base table''
exec (@rebuild_indexes_in_' + replace(replace(sd.name, ' ', '_'), '.', '_') + ')' + CHAR(10) + CHAR(10)
from sys.databases sd where name not in ('master', 'model', 'msdb', 'tempdb') order by name asc
select  (@index_maintenance) for xml path(''), type
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

   
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

