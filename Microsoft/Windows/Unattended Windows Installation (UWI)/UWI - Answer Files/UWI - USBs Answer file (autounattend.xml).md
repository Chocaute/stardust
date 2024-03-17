---
created: 2024-01-05T16:34
updated: 2024-01-05T17:02
tags:
  - Windows
  - Automation
  - Deployment
---
#### Useful links if you can't be bothered to research: 

- https://www.windowsafg.com/
- https://schneegans.de/windows/unattend-generator/

#### REMINDERS : 
> [!NOTE] #1 
> if you don't have a `windowsPE` section, the installer "forgets" to copy the `Autounattend.xml` from the root of the PE image to the installation drive's `\Windows\Panther\unattend.xml`, and without `unattend.xml` there, you can forget about anything being applied.
> 
> Source:
> https://github.com/pbatard/rufus/issues/1981

> [!NOTE] #2
> You need to provide a ProductKey section in the windowsPE pass, even if it's just to provide a blank/invalid key. OR ELSE.
> 
> Source:
> https://github.com/pbatard/rufus/issues/1981

> [!NOTE] #3
> This method of autounattend doesn't have system rights, even before the user log in for the first time, making post-install scripts unreliable through SetupComplete unreliable. 
> 
> Think about this when handling secrets!
> Need to this test this further.

---
#### An Exemple :

Here's the contents of an autounattend.xml at the root of an USB key. 
- Set the language at all relevant stages to French.
- This particular version work only with an UEFI boot.
- The installation setup is seamless, no user input needed.
- It setup the first local disk available. 
- It automatically setup the wifi to use the local network.
- It hides Windows OOBE, and turn off the privacy-invasive features.
- It creates an user and log to it automatically.

```
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <settings pass="windowsPE">
        <component name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <SetupUILanguage>
                <UILanguage>fr-FR</UILanguage>
            </SetupUILanguage>
            <InputLocale>fr-FR</InputLocale>
            <SystemLocale>fr-FR</SystemLocale>
            <UILanguage>fr-FR</UILanguage>
            <UserLocale>fr-FR</UserLocale>
        </component>
        <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <ImageInstall>
				        <OSImage>
					          <InstallTo>
						            <DiskID>0</DiskID>
						            <PartitionID>3</PartitionID>
					          </InstallTo>
				        </OSImage>
			      </ImageInstall>
            <UserData>
                <AcceptEula>true</AcceptEula>
                <ProductKey>
                    <Key>----</Key>
                </ProductKey>
            </UserData>
            <RunSynchronous>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>1</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo SELECT DISK=0"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>2</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo CLEAN"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>3</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo CONVERT GPT"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>4</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo CREATE PARTITION EFI SIZE=300"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>5</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo FORMAT QUICK FS=FAT32 LABEL="System""</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>6</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo CREATE PARTITION MSR SIZE=16"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>7</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo CREATE PARTITION PRIMARY"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>8</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo SHRINK MINIMUM=1000"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>9</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo FORMAT QUICK FS=NTFS LABEL="Windows""</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>10</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo CREATE PARTITION PRIMARY"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>11</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo FORMAT QUICK FS=NTFS LABEL="Recovery""</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>12</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo SET ID="de94bba4-06d1-4d40-a16a-bfd50179d6ac""</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>13</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.txt" echo GPT ATTRIBUTES=0x8000000000000001"</Path>
				        </RunSynchronousCommand>
				        <RunSynchronousCommand wcm:action="add">
					        <Order>14</Order>
					        <Path>cmd.exe /c "&gt;&gt;"X:\diskpart.log" diskpart.exe /s "X:\diskpart.txt""</Path>
				        </RunSynchronousCommand>
			        </RunSynchronous>
        </component>
    </settings>
    <settings pass="specialize">
		    <component name="Microsoft-Windows-Deployment" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
			    <RunSynchronous>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>1</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;WLANProfile xmlns="http://www.microsoft.com/networking/WLAN/profile/v1"^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>2</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;name^&gt;[REDACTED]^&lt;/name^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>3</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;SSIDConfig^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>4</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;SSID^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>5</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;hex^&gt;4c756e64694d6174696e^&lt;/hex^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>6</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;name^&gt;[REDACTED]^&lt;/name^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>7</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/SSID^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>8</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/SSIDConfig^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>9</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;connectionType^&gt;ESS^&lt;/connectionType^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>10</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;connectionMode^&gt;auto^&lt;/connectionMode^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>11</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;MSM^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>12</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;security^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>13</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;authEncryption^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>14</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;authentication^&gt;WPA2PSK^&lt;/authentication^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>15</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;encryption^&gt;AES^&lt;/encryption^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>16</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;useOneX^&gt;false^&lt;/useOneX^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>17</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/authEncryption^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>18</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;sharedKey^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>19</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;keyType^&gt;passPhrase^&lt;/keyType^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>20</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;protected^&gt;false^&lt;/protected^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>21</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;keyMaterial^&gt;[REDACTED]^&lt;/keyMaterial^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>22</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/sharedKey^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>23</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/security^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>24</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/MSM^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>25</Order>
					    <Path>cmd.exe /c "&gt;&gt;"%TEMP%\wifi.xml" echo ^&lt;/WLANProfile^&gt;"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>26</Order>
					    <Path>netsh.exe wlan add profile filename="%TEMP%\wifi.xml" user=all</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>27</Order>
					    <Path>netsh.exe wlan connect name="[REDACTED]" ssid="[REDACTED]"</Path>
				    </RunSynchronousCommand>
				    <RunSynchronousCommand wcm:action="add">
					    <Order>28</Order>
					    <Path>cmd.exe /c "del "%TEMP%\wifi.xml""</Path>
				    </RunSynchronousCommand>
			    </RunSynchronous>
		    </component>
	    </settings>
    <settings pass="oobeSystem">
        <component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            <InputLocale>fr-FR</InputLocale>
            <SystemLocale>fr-FR</SystemLocale>
            <UILanguage>fr-FR</UILanguage>
            <UserLocale>fr-FR</UserLocale>
        </component>
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">        
            <AutoLogon>
                <Password>
                    <Value>[REDACTED]</Value>
                    <PlainText>true</PlainText>
                </Password>
                <Enabled>true</Enabled>
                <Username>admin</Username>
            </AutoLogon>
            <OOBE>
                <HideEULAPage>true</HideEULAPage>
                <HideOEMRegistrationScreen>true</HideOEMRegistrationScreen>
                <HideOnlineAccountScreens>true</HideOnlineAccountScreens>
                <HideWirelessSetupInOOBE>true</HideWirelessSetupInOOBE>
                <HideLocalAccountScreen>true</HideLocalAccountScreen>
                <ProtectYourPC>3</ProtectYourPC>
            </OOBE>
            <UserAccounts>
                <LocalAccounts>
                    <LocalAccount wcm:action="add">
                        <Password>
                            <Value>[REDACTED]</Value>
                            <PlainText>true</PlainText>
                        </Password>
                        <DisplayName>Administrateur</DisplayName>
                        <Group>Administrateurs</Group>
                        <Name>admin</Name>
                    </LocalAccount>
                </LocalAccounts>
            </UserAccounts>
        </component>
    </settings>
</unattend>
```
