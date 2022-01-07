## Admin Settings

- **Cofoundry:Admin:Disabled** Disables the admin panel, removing all routes from the routing table and disabling login.
- **Cofoundry:Admin:AutoInjectVisualEditor**  Indicates whether to inject the visual editor into your content managed pages and other MVC results. Enabled by default.
- **Cofoundry:Admin:DirectoryName** The path to the admin panel. Defaults to "admin". Can only contain letters, numbers and dashes.

## Asset File Cleanup Settings

These settings control the background task that runs to clean up deleted asset files. Asset files are deleted as a background process to avoid file locking issues.

- **Cofoundry:AssetFileCleanup:Disabled** If set to true the cleanup background task is disabled.
- **Cofoundry:AssetFileCleanup:BackgroundTaskFrequencyInMinutes** How often the background task should run, measured in minutes.
- **Cofoundry:AssetFileCleanup:BatchSize** The number of queue items (files) to process each time the background task runs.
- **Cofoundry:AssetFileCleanup:CompletedItemRetentionTimeInMinutes** The number of minutes to stored data on completed items in the queue.
- **Cofoundry:AssetFileCleanup:DeadLetterRetentionTimeInMinutes** The number of minutes to store data on unprocessable items in the queue.
- **Cofoundry:AssetFileCleanup:MaxRetryWindowInDays** The total number of days to keep attempting to clean up an item in the queue.
- **Cofoundry:AssetFileCleanup:RetryOffsetInMinutes** The initial time to wait before trying again to clean up an item that previously errored. This is multiplied by the RetryOffsetMultiplier on each failed attempt.
- **Cofoundry:AssetFileCleanup:RetryOffsetMultiplier** The multiplier to use when incrementing the next attempt date when an item in the queue fails to be processed.

## Asset File Settings

- **Cofoundry:AssetFiles:FileExtensionValidation** Indicates the type of validation to perform against a file extension and is used in combination with the values in the  `FileExtensionValidationList` setting. The default is `UseBlocklist`, and other options are `UseAllowlist` or `Disable`.
- **Cofoundry:AssetFiles:FileExtensionValidationList** The list of file extensions to use when validating an uploaded file by it's file extension. By default this is a list of potentially harmful file extensions and is treated as a blocklist, but the `FileExtensionValidation` setting can be used to change this  behavior to interpret it as an allowlist, or disabled this validation entirely
- **Cofoundry:AssetFiles:MimeTypeValidation** Indicates the type of validation to perform against a mime type and is used in combination with the values in the `MimeTypeValidationList` setting. The default is `UseBlocklist`, and other options are `UseAllowlist` or `Disable`.
- **Cofoundry:AssetFiles:FileExtensionValidationList** The list of mime types to use when validating an uploaded file by it's mime type. By default this is a list of potentially  harmful mime types and is treated as a blocklist, but the `MimeTypeValidation` setting can be used to change this behavior to interpret it as an allowlist, or disabled this validation entirely.

## AuthenticationSettings

- **Cofoundry:Authentication:NumHoursPasswordResetLinkValid** The number of hours a password reset link is valid for. Defaults to 16 hours.
- **Cofoundry:Authentication:MaxIPAttempts** The maximum number of failed login attempts allowed per IP address during the time window described by the MaxIPAttemptsBoundaryInMinutes property. The default value is 60 minutes.
- **Cofoundry:Authentication:MaxUsernameAttempts** The maximum number of failed login attempts allowed per username during the time window described by the MaxUsernameAttemptsBoundaryInMinutes property. The default value is 40 minutes.
- **Cofoundry:Authentication:MaxIPAttemptsBoundaryInMinutes** The time window to measure login attempts when testing for blocking by IP address. The default value is 40 minutes.
- **Cofoundry:Authentication:MaxUsernameAttemptsBoundaryInMinutes** The time window to measure login attempts when testing for blocking by username. The default value is 20 minutes.
- **Cofoundry:Authentication:CookieNamepace** The text to use to namespace the auth cookie. The user area code will be appended to this to make the cookiename, e.g. "MyAppAuth_COF". By default the cookie namespace is created using characters from the entry assembly name of your application.

## AutoUpdateSettings

Settings that control the [auto-update process](/framework/auto-update) that runs at startup. 

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

## IdentitySettings

### Password

Controls the default password policy used for all user areas, including the Cofoundry admin user area.

- **Cofoundry:Identity:Password:MinLength:** The minimum length of a password. Defaults to 10 and anything less is not recommended. Must be between 6 and 2048 characters.
- **Cofoundry:Identity:Password:MaxLength:** The maximum length of a password. Defaults to 300 characters and must be between 6 and 2048 characters.
- **Cofoundry:Identity:Password:MinUniqueCharacters:** The number of unique characters required in a password. This is to prevent passwords like "aabbccdd". Defaults to 5 unique characters.

### EmailAddress

Controls the default email address validation rules used for all user areas, including the Cofoundry admin user area.

- **Cofoundry:Identity:EmailAddress:AllowAnyCharacter:** Allows any character in an email, effectively bypassing characters validation. Defaults to `true`, to ensure maximum compatibility to the widest range of email addresses. When `true` any settings for `AllowAnyLetters`, `AllowAnyDigit` and `AdditionalAllowedCharacters` are ignored.
- **Cofoundry:Identity:EmailAddress:AllowAnyLetter:** Allows an email to contain any character classed as a unicode letter as determined by `Char.IsLetter`. This setting is ignored when `AllowAnyCharacter` is set to `true`, which is the default behavior.
- **Cofoundry:Identity:EmailAddress:AllowAnyDigit:** Allows an email to contain any character classed as a decimal digit as determined by `Char.IsDigit` i.e 0-9. This setting is ignored when `AllowAnyCharacter` is set to `true`, which is the default behavior.
- **Cofoundry:Identity:EmailAddress:AdditionalAllowedCharacters:** Allows any of the specified characters in addition to the letters or digit characters permitted by the `AllowAnyLetters` and `AllowAnyDigit` settings. This setting is ignored when `AllowAnyCharacter` is set to `true`, which is the default behavior. The @ symbol is always permitted. The default settings specifies the range of special characters permitted in unquoted email addresses, excluding comment parentheses "()", and the square brackets "[]" that are used to denote an IP address instead of a domain i.e "!#$%&'*+-/=?^_`{|}~.@". When enabling or altering these settings please be aware of the [full extent of acceptable email formats](https://en.wikipedia.org/wiki/Email_address#Syntax).
- **Cofoundry:Identity:EmailAddress:MinLength:** The minimum length of an email address. Defaults to 3. Must be between 3 and 150 characters. 
- **Cofoundry:Identity:EmailAddress:MaxLength:** The maximum length of an email address. Defaults to 150 characters and must be between 3 and 150 characters.
- **Cofoundry:Identity:EmailAddress:RequireUnique:** Set this to `true` to ensure that an email cannot be allocated to more than one user per user area. Note that if `IUserAreaDefinition.UseEmailAsUsername` is set to `true` then this setting is ignored because usernames have to be unique. This defaults to `false` because a uniqueness check during registration can expose whether an email is registered or not, which may be sensitive information depending on the nature of the application.

### EmailAddress

Controls the default username validation rules used for all user areas.

- **Cofoundry:Identity:Username:AllowAnyCharacter:** Allows any character in a username, effectively bypassing characters validation. Defaults to `true`, to ensure maximum compatibility to the widest range of usernames when integrating with external systems. When `true` any settings for `AllowAnyLetters`, `AllowAnyDigit` and `AdditionalAllowedCharacters` are ignored. Note that username character validation is ignored when `IUserAreaDefinition.UseEmailAsUsername` is set to `true`, because the format is already validated against the configured `EmailAddressOptions`.
- **Cofoundry:Identity:Username:AllowAnyLetter:** Allows a username to contain any character classed as a unicode letter as determined by `Char.IsLetter`. This setting is ignored when `AllowAnyCharacter` is set to `true`, which is the default behavior.
- **Cofoundry:Identity:Username:AllowAnyDigit:** Allows a username to contain any character classed as a decimal digit as determined by `Char.IsDigit` i.e 0-9. This setting is ignored when `AllowAnyCharacter` is set to `true`, which is the default behavior.
- **Cofoundry:Identity:Username:AdditionalAllowedCharacters:** Allows any of the specified characters in addition to the letters or digit characters permitted by the `AllowAnyLetter` and `AllowAnyDigit` settings. This setting is ignored when `AllowAnyCharacter` is set to `true`, which is the default behavior. The default settings specifies a handful of special characters commonly found in usernames: "-._' ".
- **Cofoundry:Identity:Username:MinLength:** The minimum length of a username. Defaults to 1. Must be between 1 and 150 characters. 
- **Cofoundry:Identity:Username:MaxLength:** The maximum length of a username. Defaults to 150 characters and must be between 1 and 150 characters.

## ImageAssetSettings

- **Cofoundry:ImageAssets:Disabled** Disables image asset functionality, removing it from the admin panel and skipping registration of image asset routes. Access to images is still possible from code if you choose to use those APIs from a user account with permissions.
- **Cofoundry:ImageAssets:EnableCompatibilityRoutesFor0_4** Enables image asset routes that work for URLs generated prior to v0.4 of Cofoundry. It isn't recommended to enable these unless you really need to because the old routes were vulnerable to enumeration.
- **Cofoundry:ImageAssets:DisableResizing** Indicates whether dynamic image resizing should be disabled. Defaults to false. An exception will be thrown if image resizing is requested but not enabled.
- **Cofoundry:ImageAssets:MaxUploadWidth** The maximum size in pixels of the image that can be uploaded. Defaults to 3200. If uploading via the admin panel, oversized images will be resized automatically before uploading.
- **Cofoundry:ImageAssets:MaxUploadHeight** The maximum height in pixels of the image that can be uploaded. Defaults to 3200. Defaults to 3200. If uploading via the admin panel, oversized images will be resized automatically before uploading.
- **Cofoundry:ImageAssets:MaxResizeWidth** The maximum width in pixels of that an image is permitted to be resized to. Defaults to 3200.
- **Cofoundry:ImageAssets:MaxResizeHeight** The maximum height in pixels of that an image is permitted to be resized to. Defaults to 3200.
- **Cofoundry:ImageAssets:CacheMaxAge**  The default max-age to use for the cache control header, measured in seconds. Image asset file URLs are designed to be permanently cacheable so the default value is 1 year.

## InMemoryObjectCacheSettings

These settings are specificaly for the default in-memory object cache implementation.

- **Cofoundry:InMemoryObjectCache:CacheMode**  The cache mode that should be used to determine the lifetime of data stored in the cache. Defaults to InMemoryObjectCacheMode.Persitent, which is the preferred mode for a single-server deployment. InMemoryObjectCacheMode.PerScope can be used to enable a simple multi-server deployment.

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
