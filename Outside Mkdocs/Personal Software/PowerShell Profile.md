### Global
These global variables are mandatory for some of the functions to work.

```powershell
# Find out if the current user identity is elevated (has admin rights)
$identity = [Security.Principal.WindowsIdentity]::GetCurrent()
$principal = New-Object Security.Principal.WindowsPrincipal $identity
$isAdmin = $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)
```

After all the functions that use these variables are done add these lines at the bottom of the profile:
```powershell
# We don't need these any more; they were just temporary variables to get to $isAdmin. 
# Delete them to prevent cluttering up the user profile. 
Remove-Variable identity
Remove-Variable principal
```
### cd... and cd....
```powershell
# If so and the current host is a command line, then change to red color 
# as warning to user that they are operating in an elevated context
# Useful shortcuts for traversing directories
function cd... { Set-Location ..\.. }
function cd.... { Set-Location ..\..\.. }
```

### Compute file hashes
```powershell
# Compute file hashes - useful for checking successful downloads 
function md5 { Get-FileHash -Algorithm MD5 $args }
function sha1 { Get-FileHash -Algorithm SHA1 $args }
function sha256 { Get-FileHash -Algorithm SHA256 $args }
```

### Drive shortcuts
```powershell
# Drive shortcuts
function HKLM: { Set-Location HKLM: }
function HKCU: { Set-Location HKCU: }
function Env: { Set-Location Env: }
```

### Prompt

**Prerequisite:** [Global Variables](#Global)

Set up command prompt and window title. Use UNIX-style convention for identifying whether user is elevated (root) or not.

Window title shows current version of PowerShell and appends [ADMIN] if appropriate for easy taskbar identify.

Also, if you're in a Git repository it will show you the current branch you in.

```powershell
# Set up command prompt and window title. Use UNIX-style convention for identifying 
# whether user is elevated (root) or not. Window title shows current version of PowerShell
# and appends [ADMIN] if appropriate for easy taskbar identification
function prompt { 
    $isAdmin = $false  # Replace this with your logic for determining admin status
    
    $gitBranch = $(git rev-parse --abbrev-ref HEAD 2>$null)
    $location = if ($isAdmin) { "#" } else { "$" }

    if ($gitBranch) {
        "[" + (Get-Location) + " ] ($gitBranch) $location "
    } else {
        "[" + (Get-Location) + "] $location "
    }
}

# Show Powershell with the current version of powershell
$Host.UI.RawUI.WindowTitle = "PowerShell {0}" -f $PSVersionTable.PSVersion.ToString()
if ($isAdmin) {
	# Add the Admin suffix to the window if running as admin
    $Host.UI.RawUI.WindowTitle += " [ADMIN]"
}
```

### dirs

Does the the rough equivalent of dir /s /b. For example, dirs *.png is dir /s /b *.png

```powershell
# Does the the rough equivalent of dir /s /b. For example, dirs *.png is dir /s /b *.png
function dirs {
    if ($args.Count -gt 0) {
        Get-ChildItem -Recurse -Include "$args" | Foreach-Object FullName
    } else {
        Get-ChildItem -Recurse | Foreach-Object FullName
    }
}
```

### Admin shell

Simple function to start a new elevated process. If arguments are supplied then a single command is started with admin rights; if not then a new admin instance of PowerShell is started.

Then, set UNIX-like aliases for the admin command, so sudo <command> will run the command with elevated rights. 

```powershell
# Simple function to start a new elevated process. If arguments are supplied then 
# a single command is started with admin rights; if not then a new admin instance
# of PowerShell is started.
function admin {
    if ($args.Count -gt 0) {   
        $argList = "& '" + $args + "'"
        Start-Process "$psHome\powershell.exe" -Verb runAs -ArgumentList $argList
    } else {
        Start-Process "$psHome\powershell.exe" -Verb runAs
    }
}

# Set UNIX-like aliases for the admin command, so sudo <command> will run the command
# with elevated rights. 
Set-Alias -Name su -Value admin
Set-Alias -Name sudo -Value admin
```

### ll

```powershell
function ll { Get-ChildItem -Path $pwd -File }
```

### Git Functions
```powershell
function gcom {
    git add .
    git commit -m "$args"
}

function lazyg {
    git add .
    git commit -m "$args"
    git push
}
```

### uptime
```powershell
function uptime {
    #Windows Powershell    
    Get-WmiObject win32_operatingsystem | Select-Object csname, @{
        LABEL      = 'LastBootUpTime';
        EXPRESSION = { $_.ConverttoDateTime($_.lastbootuptime) }
    }

    #Powershell Core / Powershell 7+ (Uncomment the below section and comment out the above portion)

    
	$bootUpTime = Get-WmiObject win32_operatingsystem | Select-Object lastbootuptime
	$plusMinus = $bootUpTime.lastbootuptime.SubString(21,1)
	$plusMinusMinutes = $bootUpTime.lastbootuptime.SubString(22, 3)
	$hourOffset = [int]$plusMinusMinutes/60
	$minuteOffset = 00
	if ($hourOffset -contains '.') { $minuteOffset = [int](60*[decimal]('.' + $hourOffset.ToString().Split('.')[1]))}       
	  if ([int]$hourOffset -lt 10 ) { $hourOffset = "0" + $hourOffset + $minuteOffset.ToString().PadLeft(2,'0') } else { $hourOffset = $hourOffset + $minuteOffset.ToString().PadLeft(2,'0') }
	$leftSplit = $bootUpTime.lastbootuptime.Split($plusMinus)[0]
	$upSince = [datetime]::ParseExact(($leftSplit + $plusMinus + $hourOffset), 'yyyyMMddHHmmss.ffffffzzz', $null)
	Get-WmiObject win32_operatingsystem | Select-Object @{LABEL='Machine Name'; EXPRESSION={$_.csname}}, @{LABEL='Last Boot Up Time'; EXPRESSION={$upsince}}
        


    #Works for Both (Just outputs the DateTime instead of that and the machine name)
    # net statistics workstation | Select-String "since" | foreach-object {$_.ToString().Replace('Statistics since ', '')}
        
}
```

### Get Public IP
```powershell
function Get-PubIP {
    (Invoke-WebRequest http://ifconfig.me/ip ).Content
}
```

### More Linux commands

```powershell
function reload-profile {
    & $profile
}
function find-file($name) {
    Get-ChildItem -recurse -filter "*${name}*" -ErrorAction SilentlyContinue | ForEach-Object {
        $place_path = $_.directory
        Write-Output "${place_path}\${_}"
    }
}
function unzip ($file) {
    Write-Output("Extracting", $file, "to", $pwd)
    $fullFile = Get-ChildItem -Path $pwd -Filter .\cove.zip | ForEach-Object { $_.FullName }
    Expand-Archive -Path $fullFile -DestinationPath $pwd
}
function grep($regex, $dir) {
    if ( $dir ) {
        Get-ChildItem $dir | select-string $regex
        return
    }
    $input | select-string $regex
}
function touch($file) {
    "" | Out-File $file -Encoding ASCII
}
function df {
    get-volume
}
function sed($file, $find, $replace) {
    (Get-Content $file).replace("$find", $replace) | Set-Content $file
}
function which($name) {
    Get-Command $name | Select-Object -ExpandProperty Definition
}
function export($name, $value) {
    set-item -force -path "env:$name" -value $value;
}
function pkill($name) {
    Get-Process $name -ErrorAction SilentlyContinue | Stop-Process
}
function pgrep($name) {
    Get-Process $name
}
```

### Auto complete using Tab
```powershell
# AUTOCOMPLETE Using Tab
# https://stackoverflow.com/questions/8264655/how-to-make-powershell-tab-completion-work-like-bash
Set-PSReadlineKeyHandler -Key Tab -Function Complete
# Alternative to make it even more like bash where you can use arrow-keys to navigate available options:
	# Set-PSReadlineKeyHandler -Key Tab -Function MenuComplete
```


### Backspace to work on VSCode
```powershell
if ($env:TERM_PROGRAM -eq "vscode") {
	# Backspace to delete full words in vscode
	Set-PSReadLineKeyHandler -Chord 'Ctrl+w' -Function BackwardKillWord
}
```

### Install Python Environment Functions

* Install requirements.txt using `VenvInstallRequirements`
* Activate existing environment using `VenvActivate`

```powershell
function VenvInstallRequirements{
    .\venv\Scripts\Activate.ps1
    python -m pip install -r requirements.txt --proxy http://proxy-iil.intel.com:911
}

function VenvActivate{
    .\venv\Scripts\Activate.ps1
}
```