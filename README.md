# Bonsai

A family wiki and photoalbum engine (in Russian).

## Features

* Pages with Markdown text
* Media files: photos, videos (PDF documents will be supported later)
* Person tags on photos
* Relations: validation, inferrence
* Fact storage (birthday, gender, blood type, languages, hobbies, etc.)
* Access control: editor, reader and guest roles
* Changesets: browse changes to any page/media, see diffs, easily revert if necessary

## Screenshots

#### Front-end:

<a href="https://user-images.githubusercontent.com/604496/46574247-037d4f00-c9a9-11e8-8585-0d574dda2600.png"><img src="https://user-images.githubusercontent.com/604496/46574252-1859e280-c9a9-11e8-821f-daeaaac7de3f.png" /></a>
<a href="https://user-images.githubusercontent.com/604496/46574259-2c054900-c9a9-11e8-8ecc-ca542053f665.png"><img src="https://user-images.githubusercontent.com/604496/46574288-9a4a0b80-c9a9-11e8-8373-2a7d3e00289c.png" /></a>
<a href="https://user-images.githubusercontent.com/604496/46574262-31629380-c9a9-11e8-9ea6-18fbe63f239f.png"><img src="https://user-images.githubusercontent.com/604496/46574291-9f0ebf80-c9a9-11e8-8656-8a54dd2f2be7.png" /></a>

#### Admin panel:

<a href="https://user-images.githubusercontent.com/604496/46574266-3f181900-c9a9-11e8-828d-9d9a5db25acb.png"><img src="https://user-images.githubusercontent.com/604496/46574292-a209b000-c9a9-11e8-8193-cd99fc1f5f91.png" /></a>
<a href="https://user-images.githubusercontent.com/604496/46574268-43443680-c9a9-11e8-974f-f8a60fbeaa74.png"><img src="https://user-images.githubusercontent.com/604496/46574297-a504a080-c9a9-11e8-8612-d3e5cd1592a4.png" /></a>

## Installation

For development, you will need the following:

* [.NET Core 2.1](https://dotnet.microsoft.com/download/dotnet-core/2.1): the main runtime for Bonsai
* [Java 8+](https://java.com/en/download/windows-64bit.jsp): required for ElasticSearch

### Windows
1. Install [NodeJS](https://nodejs.org/en/) (10+)
2. Install [PostgreSQL server](https://www.openscg.com/bigsql/postgresql/installers.jsp/) (9.6+)
3. Install [ElasticSearch 5.6.x](https://www.elastic.co/downloads/past-releases) (6.0 is not supported yet)
4. Install [Russian Morphology](https://github.com/imotov/elasticsearch-analysis-morphology) for ElasticSearch.
   
   If you're getting a "Syntax of the command is incorrect" error during this step, make sure you have a `JAVA_HOME` environment variable defined.
5. Download [ffmpeg shared binaries](https://ffmpeg.zeranoe.com/builds/) for your system and extract the archive's contents into `External/ffmpeg` folder in the solution root (must contain both `ffmpeg` and `ffprobe` executables).
6. Create a file called `appsettings.Development.json`, add the connection string:

    ```
    {
      "ConnectionStrings": {
        "Database": "Server=127.0.0.1;Port=5432;Database=bonsai;User Id=<login>;Password=<password>;Persist Security Info=true"
      },
      "Auth": {
	    "AllowPasswordAuth": true
      } 
    }
    ```

7. _Optional, but suggested_:

    Create a [Facebook Authorization App](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/social/facebook-logins?view=aspnetcore-2.1&tabs=aspnetcore2x) (or Google, Yandex or Vkontakte).

	Add the retrieved authorization credentials to the `appsettings.Development.json` and set `AllowPasswordAuth` to `false`:

	```
	{
	    "Auth": {
		    "AllowPasswordAuth": false,
		    "Facebook": {
			  "AppId": "<...>",
			  "AppSecret": "<...>" 
			},
			"Google": {
			  "ClientId": "<...>",
			  "ClientSecret": "<...>" 
			},
			"Yandex": {
			  "ClientId": "<...>",
			  "ClientSecret": "<...>" 
			},
			"Vkontakte": {
			  "ClientId": "<...>",
			  "ClientSecret": "<...>" 
			}
		}
	}
	```
    
8. Create the database:

    ```
    dotnet ef database update
    ```
9. Build the styles and scripts:

    ```
    npm install
    npm run build
    ```
10. Run the app (as Visual Studio project or using `dotnet run`).

### Linux + Docker
1. Modify your `vm.max_map_count` to at least 262,144 for ElasticSearch to start:

    ```
    sysctl -w vm.max_map_count=262144
    ```

2. Download the [docker-compose](docker-compose.yml). You will need to **replace placeholders** for HTTPS certificate issuing:

    * `@@YOUR_EMAIL@@` - your email address (for LetsEncrypt auto-SSL)
    * `@@YOUR_IP@@` - your external IP address (for xip.io)

2. _Optional, but suggested_:

    Create a [Facebook Authorization App](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/social/facebook-logins?view=aspnetcore-2.1&tabs=aspnetcore2x) (or Google, Yandex or Vkontakte).

	Save the retrieved authorization credentials in `docker-compose.yml` and set `Auth__AllowPasswordAuth` to `false`.
   
4. Bring everything up using `docker compose`:
   ```
   docker-compose up -d
   ```
5. After everything is brought up Bonsai will listen on ports 80 and 443.

## Security considerations

### Data backup

If you value the data that you store in Bonsai, make sure that you **SET UP BACKUPS**.

You will need to back up the following:

* Database (approximately tens of megabytes)
* Uploaded media in `wwwroot/media` (may contain gigabytes of data)

There are many options available, free and paid: uploading to a cloud storage, copying to external drives, etc. Please consider your usage/budget and select a combination that works best for you.

When restoring the database from a backup, set the `SeedData.ResetElastic` option to `true` in the config to rebuild full text search indices in sync with your current database.

### Authorization methods

Bonsai features two authorization methods: OAuth and password authorization.

OAuth is the preferred method: it's easier to use, more secure and versatile. **Please use the OAuth method if you can!**
For OAuth, you will need to create an authorization app on [Facebook](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/social/facebook-logins?view=aspnetcore-3.0), [Google](https://docs.microsoft.com/en-us/aspnet/core/security/authentication/social/google-logins?view=aspnetcore-3.0), [Vkontakte](https://vk.com/editapp?act=create) or [Yandex](https://oauth.yandex.ru/client/new) as described in the installation steps.
You can enable multiple authorization apps at the same time: users will pick the one they prefer.

As a fallback, you can also create an account with classic login/password authorization. It can be used for two purposes:

* Playing around with Bonsai (easier to set up: no auth app and https configuration required)
* Giving access to elder family members who don't have a social network account

Please keep the following facts in mind:

* Any user account can only have one authorization method: password, or Facebook, or Google, etc.
* It is not possible to change the authorization type for an account once it has been created.
* Password-based accounts can be locked out if there are too many consecutive failed login attempts.
* Account password can only be reset by an administrator manually. If you only have one admin account, it is password-based, and the password is lost - there's no way to regain access besides direct database manipulation!
