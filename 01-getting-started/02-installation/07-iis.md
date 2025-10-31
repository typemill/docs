#  Run Typemill on IIS

This installation guide below has been provided by [sohamakl on GitHub](https://github.com/typemill/typemill/discussions/518).

## Prerequisite

Make sure that the following tools and extensions are installed on your IIS server:

* **Composer for Windows**: Required to update and install the external libraries for typemill if you use the GitHub-version). 
* **curl**: curl is required for several requests, e.g. the version check and the license activation.  If you run into errors, then read this article about the handling of [ssl certificates with curl](https://php.watch/articles/php-curl-windows-cainfo-fix) on Windows. Make also sure that **cainfo.pem** is downloaded and php.ini is correctly configured if you run into troubles. 
* **OpenSSL**: OpenSSL is not available for IIS by default. There are workarounds in Typemill to handle verifications on a remote server, but if there is a possibility to install OpenSSL on the IIS server, then this is the better option.

You can also configure an own IIS Application Pool for Typemill.

## Set IIS Folder Permissions

In the folder security configuration of each folder with write access, add the Windows Server local user groups IUSR and IIS_IUSRS as shown below.

![Screenshot: Defining usergroups in IIS](media/live/iis-permissions.webp){.center loading="lazy" width="766" height="519"}

![Screenshot: Setting permissions in IIS](media/live/iis-permissions-2.webp){.center loading="lazy" width="820" height="530"}

## web.config

First download and install [URL Rewrite module for IIS](https://www.iis.net/downloads/microsoft/url-rewrite) if not already available. The easiest way to add the URL Rewrite configuration to the actual web.config is as follows:

* Select the root folder of typemill in the IIS Configuration Manager.
* Open the URL Rewriter configuration.
* Select import, insert and import the content of the .htaccess file.
* Remove the unneeded Flag NE and apply the configuration change.

![Screenshot: Select URL rewrite module in IIS](media/live/iis-rewrite-rule-1.webp){.center loading="lazy" width="820" height="535"}

![Screenshot: Import rewrite rules in IIS](media/live/iis-rewrite-rule-2.webp){.center loading="lazy" width="820" height="535"}

![Screenshot: Transform the apache rewrite roles in IIS](media/live/iis-rewrite-rule-3.webp){.center loading="lazy" width="820" height="682"}

The web.config in the root directory of Typemill should look like this now:

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="Imported Rule 1" stopProcessing="true">
                    <match url="^" ignoreCase="false" />
                    <conditions logicalGrouping="MatchAll">
                        <add input="{URL}" pattern="^/index\.php" ignoreCase="false" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
                        <add input="{REQUEST_FILENAME}" matchType="IsDirectory" ignoreCase="false" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" appendQueryString="true" />
                </rule>
                <rule name="Imported Rule 2" stopProcessing="true">
                    <match url="^(.*)?\.yml$" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 3" stopProcessing="true">
                    <match url="^(.*)?\.yaml$" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 4" stopProcessing="true">
                    <match url="^(.*)?\.txt$" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 5" stopProcessing="true">
                    <match url="^(.*)?\.example$" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 6" stopProcessing="true">
                    <match url="^(.*)?\.git+" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 7" stopProcessing="true">
                    <match url="^(.*)?\.md" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 8" stopProcessing="true">
                    <match url="^(.*)?\.ph" ignoreCase="false" />
                    <conditions logicalGrouping="MatchAll">
                        <add input="{URL}" pattern="/index\.php" ignoreCase="false" negate="true" />
                    </conditions>
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 9" stopProcessing="true">
                    <match url="^(.*)?\.twig" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 10" stopProcessing="true">
                    <match url="^(media\/tmp\/)" ignoreCase="false" />
                    <action type="CustomResponse" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 11" stopProcessing="true">
                    <match url="^(composer\.lock|composer\.json|\.htaccess)$" ignoreCase="false" />
                    <action type="CustomResponse" url="error" statusCode="403" statusReason="Forbidden" statusDescription="Forbidden" />
                </rule>
                <rule name="Imported Rule 12" stopProcessing="true">
                    <match url="(^|/)\.(?!well-known\/)" ignoreCase="false" />
                    <action type="Rewrite" url="index.php" />
                </rule>
                <rule name="Imported Rule 13" stopProcessing="true">
                    <match url="^(system\/typemill\/author\/css\/)" ignoreCase="false" />
                    <action type="None" />
                </rule>
                <rule name="Imported Rule 14" stopProcessing="true">
                    <match url="^(system\/typemill\/author\/img\/)" ignoreCase="false" />
                    <action type="None" />
                </rule>
                <rule name="Imported Rule 15" stopProcessing="true">
                    <match url="^(system\/typemill\/author\/js\/)" ignoreCase="false" />
                    <action type="None" />
                </rule>
                <rule name="Imported Rule 16" stopProcessing="true">
                    <match url="^(system|content|data|settings|(media\/files\/))" ignoreCase="false" />
                    <action type="Rewrite" url="index.php" appendQueryString="true" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

## More Information

* Typemill is built on the Slim Framework, which provides information about [configuration for IIS](https://www.slimframework.com/docs/v3/start/web-servers.html#iis).
* If you encounter a 404 response on IIS, this discussion on [Stack Overflow](https://stackoverflow.com/questions/13902696/getting-404-errors-using-slim-php-on-iis7) may offer some guidance.
* Information about handling of [ssl-certificates with curl](https://php.watch/articles/php-curl-windows-cainfo-fix) on IIS.

