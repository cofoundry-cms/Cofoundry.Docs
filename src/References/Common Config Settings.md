## Admin Settings

- **Cofoundry:Admin:Disabled** Disables the admin panel, removing all routes from the routing table and disabling login.
- **Cofoundry:Admin:DirectoryName** The path to the admin panel. Defaults to "admin". Can only contain letters, numbers and dashes.

## Asset File Settings

- **Cofoundry:AssetFiles:FileExtensionValidation** Indicates the type of validation to perform against a file extension and is used in combination with the values in the  `FileExtensionValidationList` setting. The default is `UseBlacklist`, and other options are `UseWhitelist` or `Disable`.
- **Cofoundry:AssetFiles:FileExtensionValidationList** The list of file extensions to use when validating an uploaded file by it's file extension. By default this is a list of potentially harmful file extensions and is treated as a blacklist, but the `FileExtensionValidation` setting can be used to change this  behavior to interpret it as a whitelist, or disabled this validation entirely
- **Cofoundry:AssetFiles:MimeTypeValidation** Indicates the type of validation to perform against a mime type and is used in combination with the values in the `MimeTypeValidationList` setting. The default is `UseBlacklist`, and other options are `UseWhitelist` or `Disable`.
- **Cofoundry:AssetFiles:FileExtensionValidationList** The list of mime types to use when validating an uploaded file by it's mime type. By default this is a list of potentially  harmful mime types and is treated as a blacklist, but the `MimeTypeValidation` setting can be used to change this behavior to interpret it as a whitelist, or disabled this validation entirely.

## AuthenticationSettings

- **Cofoundry:Authentication:NumHoursPasswordResetLinkValid** The number of hours a password reset link is valid for. Defaults to 16 hours.
- **Cofoundry:Authentication:MaxIPAttempts** The maximum number of failed login attempts allowed per IP address during the time window described by the MaxIPAttemptsBoundaryInMinutes property. The default value is 60 minutes.
- **Cofoundry:Authentication:MaxUsernameAttempts** The maximum number of failed login attempts allowed per username during the time window described by the MaxUsernameAttemptsBoundaryInMinutes property. The default value is 40 minutes.
- **Cofoundry:Authentication:MaxIPAttemptsBoundaryInMinutes** The time window to measure login attempts when testing for blocking by IP address. The default value is 40 minutes.
- **Cofoundry:Authentication:MaxUsernameAttemptsBoundaryInMinutes** The time window to measure login attempts when testing for blocking by username. The default value is 20 minutes.
- **Cofoundry:Authentication:CookieNamepace** The text to use to namespace the auth cookie. The user area code will be appended to this to make the cookiename, e.g. "MyAppAuth_COF". By default the cookie namespace is created using characters from the entry assembly name of your application.

## AutoUpdateSettings

- **Cofoundry:AutoUpdate:Disabled** Disables the auto-update process entirely.
- **Cofoundry:AutoUpdate:ProcessLockTimeoutInSeconds** The amount of time before the process lock expires and allows another auto-update process to start. This is designed to prevent multiple auto-update processes running concurrently in multi-instance deployment scenarios. By default this is set to 10 minutes which should be more than enough time for the process to run, but you may wish to shorten/lengthen this depending on your needs.
- **Cofoundry:AutoUpdate:RequestWaitForCompletionTimeInSeconds** The amount of time (in seconds) that a request should pause and wait for the auto-update process to complete before returning a 503 "temporarily unavailable" response to the client. This defaults to 15 seconds. Setting this to 0 will cause the process not to wait. process entirely.

## ContentSettings

- **Cofoundry:Content:AlwaysShowUnpublishedData** A developer setting which can be used to view unpublished versions of content without being logged into the administrator site.

## DatabaseSettings

- **Cofoundry:Database:ConnectionString** The main connection string to the Cofoundry database.

## DebugSettings

- **Cofoundry:Debug:DisableRobotsTxt** Disables the dynamic robots.txt file and instead serves up a file that disallows all.
 
- **Cofoundry:Debug:DeveloperExceptionPageMode** Used to indicate whether the application should show the developer exception page with full exception details or not. By default this is set to "DevelopmentOnly"; other values include "On" or "Off".

*The following settings are intended to be used when working with the Cofoundry source code.*

- **Cofoundry:Debug:UseUncompressedResources** By default Cofoundry will try and load minified css/js files, but this can be overridden for debugging purposes and an uncompressed version will try and be located first.
- **Cofoundry:Debug:BypassEmbeddedContent** Use this to bypass resources embedded in assemblies and instead load them straight from the  file system. This is intended to be used when debugging the Cofoundry project to avoid having to re-start the project when embedded resources have been updated. False by default.
- **Cofoundry:Debug:EmbeddedContentPhysicalPathRootOverride** If bypassing embedded content, `MapPath` will be used to determine the folder root unless this override is specified. The assembly name is added to the path to make the folder root of the project with the resource in.

## DocumentAssetSettings

- **Cofoundry:DocumentAssets:Disabled** Disables document asset functionality, removing it from the admin panel and skipping registration of document asset routes. Access to document is still possible from code if you choose to use those APIs from a user account with permissions.
- **Cofoundry:DocumentAssets:EnableCompatibilityRoutesFor0_4** Enables document asset routes that work for URLs generated prior to v0.4 of Cofoundry. It isn't recommended to enable these unless you really need to because the old routes were vulnerable to enumeration.
- **Cofoundry:DocumentAssets:CacheMaxAge**  The default max-age to use for the cache control header, measured in seconds. Document asset file URLs are designed to be permanently cacheable so the default value is 1 year.

## FileSystemFileStorageSettings

- **Cofoundry:FileSystemFileStorage:FileRoot** The directory root in which to store files such as images, documents and file caches. The default value is "~/App_Data/Files/". `IPathResolver` is used to resolve this path so by default you should be able to use application relative and absolute file paths.

## ImageAssetSettings

- **Cofoundry:ImageAssets:Disabled** Disables image asset functionality, removing it from the admin panel and skipping registration of image asset routes. Access to images is still possible from code if you choose to use those APIs from a user account with permissions.
- **Cofoundry:ImageAssets:EnableCompatibilityRoutesFor0_4** Enables image asset routes that work for URLs generated prior to v0.4 of Cofoundry. It isn't recommended to enable these unless you really need to because the old routes were vulnerable to enumeration.
- **Cofoundry:ImageAssets:DisableResizing** Indicates whether dynamic image resizing should be disabled. Defaults to false. An exception will be thrown if image resizing is requested but not enabled.
- **Cofoundry:ImageAssets:MaxUploadWidth** The maximum size in pixels of the image that can be uploaded. Defaults to 3200.
- **Cofoundry:ImageAssets:MaxUploadHeight** The maximum height in pixels of the image that can be uploaded. Defaults to 3200.
- **Cofoundry:ImageAssets:MaxUploadWidth** The maximum width in pixels of that an image is permitted to be resized to. Defaults to 3200.
- **Cofoundry:ImageAssets:MaxResizeHeight** The maximum height in pixels of that an image is permitted to be resized to. Defaults to 3200.
- **Cofoundry:ImageAssets:CacheMaxAge**  The default max-age to use for the cache control header, measured in seconds. Image asset file URLs are designed to be permanently cacheable so the default value is 1 year.


## MailSettings

- **Cofoundry:Mail:SendMode** Indicates whether emails should be sent and how. Uses the `MailSendMode` enum (LocalDrop, Send, SendToDebugAddress, DoNotSend)
- **Cofoundry:Mail:DebugEmailAddress** An email address to redirect all mail to when using MailSendMode.SendToDebugAddress
- **Cofoundry:Mail:DefaultFromAddress** The default address to send emails
- **Cofoundry:Mail:DefaultFromAddressDisplayName** Optionally the name to display with the default From Address
- **Cofoundry:Mail:MailDropDirectory** The path to the folder to save mail to when using SendMode.LocalDrop. Defaults to ~/App_Data/Emails


## PagesSettings

- **Cofoundry:Pages:Disabled** Disables the pages functionality, removing page, directories and page templates from the admin panel and skipping registration of the dynamic page route and visual editor. Access to pages is still possible from code if you choose to use those APIs from a user account with permissions.

## SiteUrlResolverSettings

- **Cofoundry:SiteUrlResolver:SiteUrlRoot** The root url to use when resolving a relative to absolute URL, e.g. 'http://www.cofoundry.org'. If this value is not defined then the default implementation will fall back to using URL url from the request.

## StaticFilesSettings

- **Cofoundry:StaticFiles:MaxAge** The default max-age to use for the cache control header, measured in seconds. The default value is 1 year. General advice here for a maximum is 1 year.
- **Cofoundry:StaticFiles:CacheMode** The type of caching rule to use when adding caching headers. This defaults to StaticFileCacheMode.OnlyVersionedFiles which only sets caching headers for files using the "v" querystring parameter convention.
