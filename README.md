#This runbook is intended for automate resume and pause of Power BI Embedded instances.
 <#
    .DESCRIPTION
        This runbook is intended to pause and resume PowerBI instances using tags.
    .LOGIC
        if ($pause){
            The runbook pauses the PowerBI instance
        } 
        else{
            The runbook resumes the PowerBI instance
        }
    .NOTES
        AUTHOR: Jose Martinez
        LASTEDIT: Oct 4, 2018
#>
