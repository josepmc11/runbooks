#This runbook is intended for automate resume and pause of Power BI Embedded instances.
workflow PauseResumePowerBIEmbeddedCapacity
{
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

Param(
[Parameter(Mandatory=$true)]
[String]
$TagName,
[Parameter(Mandatory=$true)]
[String]
$TagValue,
[Parameter(Mandatory=$true)]
[Boolean]
$pause
)
$connectionName = "AzureRunAsConnection"
try
{
    # Get the connection "AzureRunAsConnection "
    $servicePrincipalConnection=Get-AutomationConnection -Name $connectionName         

    "Logging in to Azure..."
    Add-AzureRmAccount `
        -ServicePrincipal `
        -TenantId $servicePrincipalConnection.TenantId `
        -ApplicationId $servicePrincipalConnection.ApplicationId `
        -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint 
}
catch {
    if (!$servicePrincipalConnection)
    {
        $ErrorMessage = "Connection $connectionName not found."
        throw $ErrorMessage
    } else{
        Write-Error -Message $_.Exception
        throw $_.Exception
    }
}

    $instances = Get-AzureRmResource -TagName $TagName -TagValue $TagValue -ResourceType Microsoft.PowerBIDedicated/capacities

    if(!$instances){
        Write-Output "We haven't found any PowerBI instance with the specified parameters. Please verify tags are correct."
    }else{
        Foreach -Parallel ($instance in $instances){
            
            if($pause){
                Write-Output "Pausing $($instance.Name)";
                Write-Output "======================"
                Suspend-AzureRmPowerBIEmbeddedCapacity -Name $instance.Name -ResourceGroupName $instance.ResourceGroupName -PassThru
                Write-Output "======================"
            }
            else{
                Write-Output "Resuming $($instance.Name)";
                Write-Output "======================"        
                Resume-AzureRmPowerBIEmbeddedCapacity -Name $instance.Name -ResourceGroupName $instance.ResourceGroupName -PassThru
                Write-Output "======================"
            }
        }
    }
    Write-Output "Runbook execution has been completed"
}
