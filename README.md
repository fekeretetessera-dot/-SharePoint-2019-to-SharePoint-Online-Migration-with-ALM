# -SharePoint-2019-to-SharePoint-Online-Migration-with-ALM
This project demonstrates migrating SharePoint 2019 content to SharePoint Online (SPO) using **ShareGate** and integrating **Application Lifecycle Management (ALM) practices**. 


---

<#
.SYNOPSIS
Pre-Migration Discovery & Assessment for SharePoint Migration

.DESCRIPTION
Performs inventory and risk analysis of SharePoint 2016/2019 environments.
Outputs reports used to define migration scope, approach, and ALM planning.

ALM Principles:
- Environment awareness
- Read-only discovery
- Repeatable and auditable outputs
- Separation of analysis from execution
#>

# ==============================
# ALM: Environment Declaration
# ==============================
$Environment = "Assessment"
$RunDate = Get-Date -Format "yyyy-MM-dd_HHmm"

Write-Host "Starting Pre-Migration Discovery | Environment: $Environment" -ForegroundColor Cyan

# ==============================
# Source Configuration
# ==============================
$SourceWebApp = "https://sp2016.contoso.com"
$OutputPath = ".\DiscoveryReports\$RunDate"

New-Item -ItemType Directory -Path $OutputPath -Force | Out-Null

# ==============================
# Discovery Thresholds
# ==============================
$LargeListThreshold = 5000
$InactiveDaysThreshold = 180

# ==============================
# Inventory Containers
# ==============================
$SiteInventory = @()
$PermissionFindings = @()
$RiskFindings = @()

# ==============================
# 1. Site & Content Inventory
# ==============================
Write-Host "Collecting Site Inventory..." -ForegroundColor Yellow

# (ShareGate or SP Management Shell would be used here)
# Placeholder logic for portfolio demonstration

$Sites = @(
    @{ Url="https://sp2016.contoso.com/sites/Claims"; LastModified=(Get-Date).AddDays(-30); Owner="Claims Admin" },
    @{ Url="https://sp2016.contoso.com/sites/OldProjects"; LastModified=(Get-Date).AddDays(-400); Owner="Unknown" }
)

foreach ($site in $Sites) {

    $Inactive = ((Get-Date) - $site.LastModified).Days -gt $InactiveDaysThreshold

    $SiteInventory += [PSCustomObject]@{
        SiteUrl        = $site.Url
        Owner          = $site.Owner
        LastModified   = $site.LastModified
        InactiveSite   = $Inactive
        MigrationScope = if ($Inactive) { "Leave Behind / Archive" } else { "Migrate" }
    }
}

# ==============================
# 2. Permission & Security Analysis
# ==============================
Write-Host "Analyzing Permissions..." -ForegroundColor Yellow

$PermissionFindings += [PSCustomObject]@{
    SiteUrl        = "https://sp2016.contoso.com/sites/Claims"
    IssueType      = "Direct User Assignment"
    RiskLevel      = "High"
    Recommendation = "Replace with Azure AD Security Group"
}

$PermissionFindings += [PSCustomObject]@{
    SiteUrl        = "https://sp2016.contoso.com/sites/OldProjects"
    IssueType      = "Orphaned User"
    RiskLevel      = "Medium"
    Recommendation = "Remove or map to group"
}

# ==============================
# 3. Large List & Performance Risk
# ==============================
Write-Host "Identifying Performance Risks..." -ForegroundColor Yellow

$RiskFindings += [PSCustomObject]@{
    SiteUrl        = "https://sp2016.contoso.com/sites/Claims"
    RiskCategory   = "Large List"
    Details        = "List exceeds 5,000 items"
    Impact         = "Migration throttling / SPO limits"
    Recommendation = "Filter, archive, or split list"
}

# ==============================
# 4. Modernization Candidates
# ==============================
Write-Host "Identifying Modernization Opportunities..." -ForegroundColor Yellow

$RiskFindings += [PSCustomObject]@{
    SiteUrl        = "https://sp2016.contoso.com/sites/Claims"
    RiskCategory   = "Legacy Customization"
    Details        = "InfoPath form detected"
    Recommendation = "Rebuild using Power Apps"
}

# ==============================
# Export Reports
# ==============================
$SiteInventory | Export-Csv "$OutputPath\SiteInventory.csv" -NoTypeInformation
$PermissionFindings | Export-Csv "$OutputPath\PermissionFindings.csv" -NoTypeInformation
$RiskFindings | Export-Csv "$OutputPath\MigrationRisks.csv" -NoTypeInformation

Write-Host "Discovery Completed Successfully" -ForegroundColor Green
Write-Host "Reports generated in: $OutputPath" -ForegroundColor Cyan

