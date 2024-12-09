# Stremio GDrive Addon 

This is a simple addon for Stremio that allows you to watch videos from Google Drive.

It searches your entire Google Drive and any shared drives you have access to for videos and presents them in Stremio.

If you combine it with some team drives, you have loads of content, all available to watch for free and without torrenting.

![image](https://github.com/user-attachments/assets/7975eb48-1705-4779-b808-c2d61f25a099)


## Deployment

This addon is designed to be deployed as a worker on Cloudflare Workers.

Here is a guide to deploying this addon, taken from my site: [Viren070's guides](https://guides.viren070.me/stremio/addons/stremio-gdrive). This will most likely be easier to follow on my site, however. 


### Setting up our Google App

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).

    If this is your first time using Google Cloud, you will be prompted to agree to the terms of service:

    ![Google Cloud Console](/images/google_cloud_onboarding.png)

2. Create a new project and select it:
    
    <details>
    <summary>How?</summary>

    1. Click on `Select a project` in the top left:

        ![Create a new project](/images/google_cloud_create_project_1.png)

    2. Click on `New Project`:

        ![Create a new project](/images/google_cloud_create_project_2.png)

    3. Enter a project name and click on `Create`:

        ![Create a new project](/images/google_cloud_create_project_3.png)

        - The project name can be anything you want, e.g., `Stremio-Gdrive`. Leave the `Organization` field blank.

    4. Once the project has been created, you will get a notification:

        ![Create a new project](/images/google_cloud_create_project_4.png)

        - Click `Select Project`. 

        :::note
        You may also use the same dropdown from step i to select the project.
        :::

    </details>

3.  Setup our Google Auth Platform 

    1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
    2. In the search bar at the top, search for `Google Auth Platform` and click on the result:

        ![Google OAuth Platform](/images/google_cloud_oauth_search_results.png)

    3. You should be met with a message telling you that `Google Auth Platform not configured yet`, click on `Get Started`: 
    
        ![Google OAuth Platform](/images/google_auth_platform_0.png)
        
    4. Fill in the form: 

        ![Google OAuth Platform](/images/google_auth_platform_1.png)

        1. `App Information`:
            - Set `App Name` to `Stremio GDrive`.
            - Set `User Support Email` to your email. It should appear in the dropdown. 
        2. `Audience`:
            - Set `User Type` to `External`.
        3. `Contact Information`
            - Add any email address, you can use the same one you used earlier for `User Support Email`.
        4. `Finish`
            - Check the box to agree to the `Google API Services: User Data Policy` 

    
    5. Once you have filled in the form, click on `Create`

4. Enable the Google Drive API. 

    1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
    2. In the search bar at the top, search for `Google Drive API` and click on the result:

        ![Google Drive API](/images/google_cloud_gdrive_api_search_results.png)

    3. Click on `Enable`:

        ![Google Drive API](/images/google_cloud_gdrive_api.png)

5. Create an OAuth client. 

    1. Go back to the `Google Auth Platform` page.
    2. Click on `Clients` in the sidebar and then click on `+ Create Client`:

        ![Google OAuth Platform](/images/google_cloud_auth_platform_clients.png)
    
    3. Fill in the form: 

        ![Google OAuth Platform](/images/google_cloud_auth_platform_clients_form.png)

        - `Application Type`: Set this to `Web application`.
        - `Name`: You can set this to anything such as `Stremio GDrive`.
        - `Authorized redirect URIs`: Set this to `https://guides.viren070.me/google/oauth/callback`


    4. Click on `Create`.

6. Publish the app. 

    1. Go back to the `Google Auth Platform` page.
    2. Click on `Audience` in the sidebar and then click on `Publish`:

        ![Google OAuth Platform](/images/google_cloud_auth_platform_publish.png)


### Setting up the Cloudflare Worker

1. Go to the [Cloudflare Workers](https://workers.cloudflare.com/) page and click `Log In` or `Sign Up` if you don't have an account.

2. Once logged in, you should be taken to the Cloudflare Workers & Pages dashboard. Click on `Create`: 

    ![Cloudflare Workers](/images/cloudflare_workers_pages_overview.png)

3. Once on the create page, make sure you're on the `Workers` tab and click `Create Worker`:

    ![Cloudflare Workers](/images/cloudflare_create_an_application_workers.png)

4. You'll be asked to give a name to your worker. You can name it anything, this will be the URL you enter into Stremio to access your addon. Click `Deploy` once named.
    
    ![Cloudflare Workers](/images/cloudflare_create_a_worker.png)

5. Once its done being deployed, you should be shown a success message. Click the `Edit code` button:

    ![Cloudflare Workers](/images/cloudflare_create_a_worker_success.png)

6. You should be taken to the Cloudflare Worker editor: 

    ![Cloudflare Workers](/images/cloudflare_worker_editor.png)

7. Now, we need to obtain the code for the addon that we will use specific to our Google Drive. 
    First, we need to obtain our `Client ID` and `Client Secret` from the Google Cloud Console.

    1. Go to the [Google Cloud Console](https://console.cloud.google.com/).

    2. In the search bar at the top, search for `Google Auth Platform` and click on the result:

        ![Google OAuth Platform](/images/google_cloud_oauth_search_results.png)

    3. Click on `Clients` and click on the download icon for the client you created earlier:

        ![Google OAuth Platform](/images/google_auth_platform_clients_download.png)

    4. A pop-up will appear with your `Client ID` and `Client Secret`. 

        ![Google OAuth Platform](/images/google_auth_platform_clients_download_menu.png)

    5. You can click the copy icons to copy the `Client ID` and `Client Secret` to your clipboard for the next step.

8. Now, we can get the code for the Cloudflare Worker.

    1. Go to the [OAuth Tool](https://guides.viren070.me/oauth/google)

    2. Fill in the form with the `Client ID` and `Client Secret` from the previous step.

    3. Click `Authorise`

    4. Sign in with your Google account and allow the app to access your Google Drive.

        > You may encounter a warning page saying `Google hasn't verified this app`, click on `Advanced` and then `Go to... (unsafe)`.
        >
        > This warning is because the app is not verified by Google. This is normal for self-hosted apps.

    5. You will be redirected back to the OAuth Tool with a success message. Click `Get Addon Code`.

    6. You should be shown another success message. Then, make sure you're on the `Viren070` tab and you should see a block of code. Copy this code.
    
    7. Go back to the Cloudflare Worker editor and paste the code into the editor.

    8. Your code should look something like this:

        ![Cloudflare Workers](/images/cloudflare_finished_worker_script.png)

10. Click `Deploy` in the top right to save your changes and deploy the worker.

11. Once deployed, you should see a green success message at the bottom. Click `Visit` next to the deploy button to go to the addon URL.

12. You should be redirected to the /manifest.json. if not, append `/manifest.json` to the URL in the address bar. 

13. Copy the URL and [add it to Stremio](https://guides.viren070.me/stremio/faq#how-do-i-install-an-addon-manually).  

Done! You have now set up your own addon which will allow you to stream videos from your drives and team drives.

## Configuration 

Although you may modify the code as you wish, I have supplied a `CONFIG` object at the top of the code after `CREDENTIALS` that allows you to easily 
configure some aspects of the addon.

> [!NOTE]
> 
> Unless stated otherwise, all values are case sensitive

This table explains the configuration options:

|                  Name                 	|      Type      	| Values                                                                                                                                             	| Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       	|
|:-------------------------------------:	|:--------------:	|----------------------------------------------------------------------------------------------------------------------------------------------------	|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
|              `resolution`             	|    String[]    	| "2160p", "1080p", "720p", "480p", "Unknown"                                                                                                        	| This setting allows you to configure which resolutions are shown in your results. You may also change the order of this to change the order that the resolutions would show in the addon (if you sort by resolutions) <br><br>You can remove certain resolutions to not have them show up in your results. The `Unknown` resolution occurs when the resolution could not be determined from the filename. <br><br>You cannot add new resolutions to this list unless you also add the corresponding regex in the `REGEX_PATTERNS` object.                                                                                         	|
|              `qualities`              	|    String[]    	| "BluRay Remux", "BDRip", "BluRay", "HDRip", "WEB-DL", "WebRip", "Web", "CAM/TS", "Unknown"                                                         	| This setting allows you to configure which qualities are shown in your results. You may also change the order of this list to change the priority of them when sorting by quality. (e.g. if you put CAM/TS at the top of the list and sorted by quality, then CAM/TS results would appear higher than other qualities) <br><br>Remove qualities from the list to remove them from your results. The `Unknown` quality occurs when one of the existing qualities could not be found in the filename. <br><br>You cannot add new qualities to this list unless you also add the corresponding regex in the `REGEX_PATTERNS` object. 	|
|                `sortBy`               	|    String[]    	| "resolution", "quality", "size"                                                                                                                    	| Change the order of this list to change the priority for which the results are sorted by. <br><br>Examples: <br><br>If you want all 2160p results to show first with those results being sorted by size, then do `resolution`, then `size`<br>If you want results to be sorted by size regardless of resolution, only have `size` in the list.                                                                                                                                                                                                                                                                                    	|
|              `addonName`              	|     String     	| any                                                                                                                                                	| Change the value contained in this string to change the name of the addon that appears in Stremio in the addon list<br>and in the stream results.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 	|
|          `prioritiseLanguage`         	| string \| null 	| See the `languages` object in the `REGEX_PATTERNS` object for a full list. <br><br>Set to null to disable prioritising specific language results.  	| By setting a prioritised language, you are pushing any results that have that language to the top. <br><br>However, parsed information about languages may be incorrect and filenames do not contain the language<br>that is contained within the file sometimes.                                                                                                                                                                                                                                                                                                                                                                 	|
| `driveQueryTerms.`<br>`episodeFormat` 	|     string     	| "name", "fullText" (see the Drive v3 API for more)                                                                                                 	| This setting changes the object that we perform the queries upon for the episode formats (s01e03). <br><br>I recommend leaving this to fullText. However, if you are getting incorrect matches, try switching to name                                                                                                                                                                                                                                                                                                                                                                                                             	|
|   `driveQueryTerms.`<br>`movieYear`   	|     string     	| "name", "fullText"                                                                                                                                 	| This setting changes the object that we perform queries upon for the release year of the movie. <br><br>I recommend leaving this to name. However, if you are getting incorrect matches, try switching to fullText.                                                                                                                                                                                                                                                                                                                                                                                                               	|
