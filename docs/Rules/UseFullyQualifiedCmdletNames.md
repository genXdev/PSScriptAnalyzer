---
description: Use fully qualified module names when calling cmdlets and functions.
ms.date: 09/19/2026
ms.topic: reference
title: UseFullyQualifiedCmdletNames
---
# UseFullyQualifiedCmdletNames

**Severity Level: Warning**

**Default state: Disabled**

## Description

PowerShell resolves a command name against the commands that are available in the session. A script
that calls `Get-Process` instead of `Microsoft.PowerShell.Management\Get-Process` binds to whichever
alias, function, or cmdlet currently owns that name. A module that exports the same name, or an
alias or function defined in the session, can therefore change which command the script runs.

This rule flags calls to cmdlets, module functions, and aliases that aren't qualified with the
module that provides them, and suggests the fully qualified `ModuleName\CommandName` replacement.
Qualifying a call also tells PowerShell which module to load, so a script doesn't depend on the
order in which commands happen to be available.

## What the rule checks

The rule resolves each command name in the current session and reports a diagnostic when the
resolved command comes from a module and the call isn't already qualified.

The rule doesn't flag:

- Native commands and external applications, such as `cmd` or `where.exe`
- Functions declared in the analyzed script, since a call in their scope resolves to the function
  instead of a same-named cmdlet
- Commands that don't resolve to a module
- Calls that already use the `ModuleName\CommandName` form
- Variables and string literals that contain text that looks like a command name

## Examples

### Noncompliant

```powershell
# Unqualified cmdlet calls
Get-Command
Write-Host 'Hello World'
Get-ChildItem -Path C:\temp

# Aliases
gci C:\temp
ls -Force
```

### Compliant

```powershell
# Fully qualified cmdlet calls
Microsoft.PowerShell.Core\Get-Command
Microsoft.PowerShell.Utility\Write-Host 'Hello World'
Microsoft.PowerShell.Management\Get-ChildItem -Path C:\temp

# The cmdlets that the aliases resolve to
Microsoft.PowerShell.Management\Get-ChildItem C:\temp
Microsoft.PowerShell.Management\Get-ChildItem -Force
```

## Configure rule

This rule is disabled by default. Set `Enable` to `$true` to run it. Use `IgnoredModules` for the
modules whose commands you don't want to qualify.

```powershell
@{
    Rules = @{
        PSUseFullyQualifiedCmdletNames = @{
            Enable = $true
            IgnoredModules = @(
                'Microsoft.PowerShell.Management'
                'Microsoft.PowerShell.Utility'
            )
        }
    }
}
```

To suppress the rule for specific code, see the _Suppressing Rules_ section of
[Using PSScriptAnalyzer][03].

## Parameters

### Enable

Controls whether the rule runs during ScriptAnalyzer invocation. Accepted values are `$true` and
`$false`. The default value is `$false`.

### IgnoredModules

Specifies the modules whose commands the rule doesn't flag. Accepted values are an array of module
names, which are matched without regard to case. The default value is an empty array, which checks
every module.

## Further reading

For more information, see the following articles:

- [about_Command_Precedence][01]
- [about_Modules][02]
- [Using PSScriptAnalyzer][03]

<!-- Link references -->
[01]: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_command_precedence
[02]: https://learn.microsoft.com/powershell/module/microsoft.powershell.core/about/about_modules
[03]: ../using-scriptanalyzer.md
