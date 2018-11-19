![D365O Cheatsheet](https://github.com/anderson-joyle/D365O-Cheatsheet/blob/master/D365O_logo_cheatsheet.png)

[![Donate Crypto](https://img.shields.io/badge/Donate-Crypto-805AFF.svg)](https://github.com/anderson-joyle/D365O-Cheatsheet#donate)
[![Say Thanks!](https://img.shields.io/badge/Say%20Thanks-!-1EAEDB.svg)](https://saythanks.io/to/joyle)

This document is a cheatsheet for Dynamics 365 AX/Op/FinOp you will frequently encounter in projects.

This guide is not intended to teach you X++ or D365 AX/Op/FinOp general development from the ground up, but to help developers with basic knowledge who may struggle to get familiar with some concepts.

Besides, I will sometimes provide personal tips that may be debatable but will take care to mention that it's a personal recommendation when I do so.

## Complementary Resources
When you struggle to understand a notion, I suggest you look for answers on the following resources:

- [Microsoft Docs](https://docs.microsoft.com/en-gb/dynamics365/unified-operations/fin-and-ops/) - Official Dynamics 365 AX/Op/FinOp documentation.
- [StackOverflow tag](https://stackoverflow.com/questions/tagged/dynamics-365-operations) - "dynamics-365-operations" tag on StackOverflow. Currently there is not that much content there, but this is a attempt to encourage you to use it when needed.
- [AX Community](https://community.dynamics.com/ax) - AX community where thousands of questions has been asked and answered.
- [#MSDyn365FO on Linkedin](https://www.linkedin.com/search/results/content/?facetSortBy=date_posted&keywords=%23MSDyn365FO&origin=SORT_RESULTS) - Official D365FO hashtag on Linkedin.
- [#MSDyn365FO on Twitter](https://twitter.com/search?f=tweets&vertical=default&q=%23MSDyn365FO&src=typd) - Official D365FO hashtag on Twitter.

## Table of Contents
- [General development](#general-development)
  * [Tables](#tables)
    + [Relationships](#relationships)
  * [Data entities](#data-entities)
    + [Copying from staging to target](#copying-from-staging-to-target)
    + [Methods](#entity-methods)
      - [mapEntityToDataSource](#mapentitytodatasource)
  * [Reports](#reports)
    + [Logging](#logging)
- [X++](#x++)

## General development
### Tables
#### Relationships
> Links in this section refers to AX 2012, but it does apply to 365 versions.
Kindly take a look at the [official documentation](https://msdn.microsoft.com/en-us/library/hh803131.aspx).

##### RelationType property
See the [official documentation](https://msdn.microsoft.com/en-us/library/hh803131.aspx)

What relation type should I choose?
Coming soon.

### [Data entities](https://docs.microsoft.com/en-us/dynamics365/unified-operations/dev-itpro/data-entities/build-consuming-data-entities?toc=/fin-and-ops/toc.json)
#### Copying from staging to target

When importing data into AX using data entities, sometimes there is no way to match data structure between data source (xml file, excel spredsheet, etc) and AX table. For instance:
  * Single line from a spredsheel source needs to be split amoung table header and table line in D365.
  * Records creation is assisted by some class and cannot be directly created by DMF (Data Management Framework).

From your data entity, create a new static field following the below template:
> Kindly note this is a personal quick recommendation. Obviously this code can be improved.
```csharp
public static container copyCustomStagingToTarget(DMFDefinitionGroupExecution _dmfDefinitionGroupExecution)
{
    CustCustomerStaging staging;
    CustCustomerStaging stagingUpd;
    
    // Iterate through all records with have not been processed
    while select forupdate staging
        where staging.ExecutionId    == _dmfDefinitionGroupExecution.ExecutionId
        &&   (staging.TransferStatus == DMFTransferStatus::NotStarted || staging.TransferStatus == DMFTransferStatus::Validated)
    {
        try
        {
            ttsbegin;
            // Do your stuff

            staging.TransferStatus = DMFTransferStatus::Completed;
            staging.update();
            ttscommit;
        }
        catch (Exception::Error)
        {
            error("Something wrong has happened.");
        }
    }    

    ttsbegin;
    update_recordset staging
        setting TransferStatus = DMFTransferStatus::Error
        where staging.DefinitionGroup == _dmfDefinitionGroupExecution.DefinitionGroup
        &&    staging.ExecutionId     == _dmfDefinitionGroupExecution.ExecutionId
        &&   (staging.TransferStatus == DMFTransferStatus::NotStarted || staging.TransferStatus == DMFTransferStatus::Validated);
    ttscommit;

    // Method returns a container containing the quantities of inserted and updated records.
    select count(RecId) from staging
        where staging.DefinitionGroup == _dmfDefinitionGroupExecution.DefinitionGroup
            && staging.ExecutionId == _dmfDefinitionGroupExecution.ExecutionId
            && staging.TransferStatus == DMFTransferStatus::Completed;

    return [staging.RecId, 0];
}
```
In order to *copyCustomStagingToTarget* be executed, you need to set field *Set-based processing* as **TRUE**.  
*Data management workspace > Data entities button*  
![set-based](https://github.com/anderson-joyle/D365O-Cheatsheet/blob/master/prints/set_base_field.PNG)

#### Handling errors messages
Basically there are two types of data entities errors messages: from *View excecution log* message and *View staging data* message. *View excecution log* displays messages in macro way e.g. "Could not connect into system X", while *View staging data* displays messages to each distinct staging table record.

##### Creating logs in *View excecution log*
Any message printed during DMF execcution (info, warning and error) will end up being displayed in *View excecution log* area.

##### Creating logs in *View staging data*
To display custom log message to specifics staging records, use the following snippet:  
```csharp
DMFStagingValidationLog::insertLogs(_dmfDefinitionGroupExecution.DefinitionGroup,
                                    _dmfDefinitionGroupExecution.ExecutionId,
                                    DMFEntity::find(_dmfDefinitionGroupExecution.Entity),
                                    staging.RecId,
                                    "",
                                    "My custom log message",
                                    DMFSourceTarget::Target);
```

#### Entity methods
##### mapEntityToDataSource
* **Direction**: Importing  
* **Purpose**: When importing, use it to fill either datasource or entity fields based on entity fields.  
* **Example**: In *CustCustomerEntity.mapEntityToDataSource()*, *EmployeeResponsibleNumber* field value is used to retrive worker record id and set it into *MainContactWorker* field from entity itself.

### Reports
#### Logging
Sometimes we get very generic erros while rendering reports from browser and Event Viewer doesnt help much. Checking reporting service log files is the best way to find out the reason:

<i>C:\Program Files\Microsoft SQL Server\ <i>SQL_INSTANCE</i> \Reporting Services\LogFiles</i>
> You may find this [addin](https://github.com/anderson-joyle/D365FO-ReportLatestLogFile) useful.

## X++
Coming soon.

## Donate
If this project helped you in any way and you feel like supporting me:

###### BTC: 1G3rHge15Kt1G8Amh2ZfNeH5pp1L3CFGmm
###### ETH: 0x9c8A747C13536b1de9Ce932E0f79FA4eB6E309b6
###### LTC: LMdh66L6tv19YLxbVgP1t47eocitSHhw5S
