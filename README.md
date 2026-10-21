# Stremio GDrive Addon 

This is a simple addon for Stremio that allows you to watch videos from Google Drive.

By default, it searches the Google Drive folders and shared drives you have shared with your service account for videos and presents them in Stremio. You can optionally restrict results to specific folders via `CONFIG.driveFolderIds`.

If you combine it with some team drives, you have loads of content, all available to watch for free and without torrenting.

![showcase](/images/stremio_gdrive_showcase.png)

## Features

- Search your Google Drive and shared drives for videos
- Parses filenames for information accurately using regex and displays it in a appealing format.
- Catalog support - both search and full list on home page
- Kitsu support.
- TMDB Meta support if TMDB api key is provided.
- Optional filtering by specific Google Drive folders (non-recursive).
- Easily configurable using the `CONFIG` object at the top of the code. (See [Configuration](#configuration))
    - Change the addon name
    - Change the order of resolutions, qualities, visual tags, and filter them out if unwanted
    - Change the sorting criteria (resolution, quality, size, visual tags)
    - Prioritise specific languages and have them show up first in the results
- Only requires a single deployment with one file, making it easy to deploy and make changes. 

## Deployment

This addon is designed to be deployed as a worker on Cloudflare Workers.

Here is a guide to deploying this addon, taken from my site: [Viren070's guides](https://guides.viren070.me/stremio/addons/stremio-gdrive). This will most likely be easier to follow on my site, however. 


### Setting up our Google service account

This addon authenticates with a Google **service account** instead of an OAuth client. There is no consent screen, no app to publish and no refresh token to obtain. The trade-off: a service account is its own Google identity with an empty Drive, so it can only see what you **share with it** (see step 6).

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

3. Enable the Google Drive API. 

    1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
    2. In the search bar at the top, search for `Google Drive API` and click on the result:

        ![Google Drive API](/images/google_cloud_gdrive_api_search_results.png)

    3. Click on `Enable`:

        ![Google Drive API](/images/google_cloud_gdrive_api.png)

4. Create a service account.

    1. In the Google Cloud Console search bar, search for `Service Accounts` (under `IAM & Admin`) and open it.
    2. Click `+ Create service account`.
    3. Set a name such as `stremio-gdrive` and click `Create and continue`.
    4. Skip the optional `Grant access` and `Grant users access` steps and click `Done`.
    5. Copy the account's email from the list. It looks like `stremio-gdrive@your-project.iam.gserviceaccount.com`. You will need it in steps 5 and 6.

5. Create a key for the service account.

    1. Click on the service account you just created and open the `Keys` tab.
    2. Click `Add key` > `Create new key`, choose `JSON` and click `Create`.
    3. A `.json` file is downloaded. Open it in a text editor. You need two fields from it:
        - `client_email`
        - `private_key` (the long text from `-----BEGIN PRIVATE KEY-----` to `-----END PRIVATE KEY-----`)

    > Treat this file like a password. Anyone with it can read everything shared with the service account. Never commit it to Git or post it publicly.
    >
    > If your organisation blocks key creation (`iam.disableServiceAccountKeyCreation`), an admin has to lift that policy for your project first.

6. Share your Drive content with the service account.

    The addon can only list files the service account has access to.

    - **Folder or file in My Drive:** right-click it in Google Drive > `Share`, paste the service account email and give it the `Viewer` role. Sharing a folder gives access to everything inside it.
    - **Shared drive (team drive):** open the shared drive > `Manage members`, add the service account email and give it the `Viewer` (or `Content manager`) role.

    Uncheck `Notify people`, since the service account has no inbox.

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

7. Remove the existing code and paste the full contents of [`src/index.js`](src/index.js) from this repository.

8. Add your credentials. Pick **one** of these options:

    - **Option A: edit the code (simplest).** At the top of the pasted code, fill in the `CREDENTIALS` object with the two values from the JSON key:

        ```js
        const CREDENTIALS = {
            clientEmail: "stremio-gdrive@your-project.iam.gserviceaccount.com",
            privateKey: "-----BEGIN PRIVATE KEY-----\nMIIE...\n-----END PRIVATE KEY-----\n",
        };
        ```

        Paste `private_key` exactly as it appears in the JSON file, including the `\n` sequences and the BEGIN/END lines, inside double quotes. A real multi-line value (use backticks instead of quotes) also works.

    - **Option B: Cloudflare secrets (recommended if you share your code).** Leave `CREDENTIALS` empty. In the worker go to `Settings` > `Variables and Secrets` > `Add`, choose type `Secret` and create:

        | Name | Value |
        |------|-------|
        | `CLIENT_EMAIL` | the `client_email` from the JSON file |
        | `PRIVATE_KEY` | the `private_key` from the JSON file |

        Values written in the code take priority over the secrets.

9. Click `Deploy` in the top right to save your changes and deploy the worker.

10. Once deployed, you should see a green success message at the bottom. Click `Visit` next to the deploy button to go to the addon URL.

11. You should be redirected to the /manifest.json. if not, append `/manifest.json` to the URL in the address bar. 

12. Copy the URL and [add it to Stremio](https://guides.viren070.me/stremio/faq#how-do-i-install-an-addon-manually).  

Done! You have now set up your own addon which will allow you to stream videos from the Drive folders and shared drives you shared with the service account.

### Troubleshooting

- **Empty catalog or no streams:** the service account has nothing shared with it yet. Redo step 6 of the Google setup.
- **`Missing clientEmail` / `Missing privateKey`:** the credentials are not set in the code or in the Cloudflare secrets.
- **`privateKey must contain the full key...`:** the key was truncated or lost its BEGIN/END lines. Paste it again from the JSON file.
- **`invalid_grant` in the worker logs:** the key was deleted or disabled in Google Cloud, or the Google Drive API is not enabled for the project.

## Configuration 

Although you may modify the code as you wish, I have supplied a `CONFIG` object at the top of the code after `CREDENTIALS` that allows you to easily 
configure some aspects of the addon.

> [!NOTE]
> 
> Unless stated otherwise, all values are case sensitive

This table explains the configuration options:

|                  Name                 	|        Type        	| Values                                                                                                                                           	| Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 	|
|:-------------------------------------:	|:------------------:	|--------------------------------------------------------------------------------------------------------------------------------------------------	|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------	|
|              `resolution`             	|     `String[]`     	| `"2160p"`, `"1080p"`, `"720p"`, `"480p"`, `"Unknown"`                                                                                            	| This setting allows you to configure which resolutions are shown in your results. You may also change the order of this to change the order that the resolutions would show in the addon (if you sort by resolutions)<br><br>You can remove certain resolutions to not have them show up in your results. The Unknown resolution occurs when the resolution could not be determined from the filename.<br><br>You cannot add new resolutions to this list unless you also add the corresponding regex in the `REGEX_PATTERNS` object.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       	|
|              `qualities`              	|     `String[]`     	| `"BluRay REMUX"`, `"BluRay"`, `"WEB-DL"`, `"WEBRip"`, `"HDRip"`, `"HC HD-Rip"`, `"DVDRip"`, `"HDTV"`, `"CAM"`, `"TS"`, `"TC"`, `"SCR"`, `"CAM"`  	| This setting allows you to configure which qualities are shown in your results. You may also change the order of this list to change the priority of them when sorting by quality. (e.g. if you put CAM/TS at the top of the list and sorted by quality, then CAM/TS results would appear higher than other qualities)<br><br>Remove qualities from the list to remove them from your results. The Unknown quality occurs when one of the existing qualities could not be found in the filename.<br><br>You cannot add new qualities to this list unless you also add the corresponding regex in the `REGEX_PATTERNS` object.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               	|
|              `visualTags`             	|     `String[]`     	| `"HDR10+"`, `"HDR10"`, `"HDR"`, `"DV"`, `"IMAX"`, `"AI"`                                                                                         	| This setting allows you to configure which visualTags are shown in your results. You can also change the order of this to change the priority of each visual tag when sorting by `visualTag` (e.g. If you were sorting by visualTag and moved IMAX to the front, and removed AI, then any results with the AI tag will be removed and any results with the IMAX tag will be pushed to the front)<br><br>You cannot add new visual tags to this list unless you also add the corresponding regex in the `REGEX_PATTERNS` object.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             	|
|                `sortBy`               	|     `String[]`     	| `"resolution"`, `"quality"`, `"size"`, `"visualTag"`                                                                                             	| Change the order of this list to change the priority for which the results are sorted by.<br><br>`resolution` - sort by the resolution of the file e.g. 1080p, 720p. The better the resolution (determined by the resolution's position in the `resolution` list) the higher the result.<br>`quality` - sort by the quality of the file, e.g. BluRay, WEBRip. The better the quality (determined by the quality's position in the `qualities` list) the higher the result.<br>`size` - sort by the size of file e.g. 12GB. The higher the size the higher they show in the results.<br>`visualTag` - sort by the priority of the visual tags. Files with a visual tag are sorted higher and between files that both have visual tags, the order of the visual tag in the `visualTags` list will determine their order.<br><br>Examples:<br><br>If you want all 2160p results to show first with those results being sorted by size, then do resolution, then size<br>If you want results to be sorted by size regardless of resolution, only have size in the list.<br>If you want to see all HDR results first, and sort those results and the rest by size, then put `visualTag` and `size` in the list.  	|
| `considerHdrTagsAsEqual`              	| `boolean`          	| `true`, `false`                                                                                                                                  	| When sorting by visualTag, if this value is set to false, the HDR tags (HDR, HDR10, HDR10+) will be considered differently, and depending on their position in the visualTags list, they will be sorted accordingly. <br><br>For example, if HDR10+ was placed first, then all HDR10+ files will appear first, regardless of if there are HDR files that rank higher based on other factors.<br><br>With this value set to true, all HDR tags are considered equal and if you were sorting by `visualTag`, then `size`, a HDR file could appear above a HDR10+ file if the size of the HDR file was greater.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                	|
|              `addonName`              	|      `String`      	| any                                                                                                                                              	| Change the value contained in this string to change the name of the addon that appears in Stremio in the addon list<br>and in the stream results.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           	|
|          `prioritiseLanguage`         	| `String` \| `null` 	| See the languages object in the `REGEX_PATTERNS` object for a full list.<br><br>Set to `null` to disable prioritising specific language results. 	| By setting a prioritised language, you are pushing any results that have that language to the top.<br><br>However, parsed information about languages may be incorrect and filenames do not contain the language<br>that is contained within the file sometimes.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            	|
|           `proxiedPlayback`           	|      `boolean`     	| `true`, `false`                                                                                                                                  	| With `proxiedPlayback` enabled, the file will be streamed through the addon. If it is disabled, Stremio will stream directly from Google Drive. <br><br>If this option is disabled, streaming will not work on Stremio Web or through external players on iOS. You are also exposing your access token in the addon responses. <br><br>However, this option is experimental and may cause issues.<br><br>I recommend leaving it enabled, and only if you encounter issues, try disabling this option. <br><br>Note, that even with this enabled, anyone with your addon URL can still view your Google Drive files.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         	|
| `driveQueryTerms.`<br>`episodeFormat` 	|      `String`      	| `"name"`, `"fullText"` (see the Drive v3 API for more)                                                                                           	| This setting changes the object that we perform the queries upon for the episode formats (s01e03).<br><br>I recommend leaving this to fullText. However, if you are getting incorrect matches, try switching to name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        	|
|   `driveQueryTerms.`<br>`movieYear`   	|      `String`      	| `"name"`, `"fullText"`                                                                                                                           	| This setting changes the object that we perform queries upon for the release year of the movie.<br><br>I recommend leaving this to name. However, if you are getting incorrect matches, try switching to fullText.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          	|
|            `driveFolderIds`            |     `String[]`      | Google Drive folder IDs (e.g., `"1abc..."`, `"2def..."`)                                                  | Restricts both catalog and search to files **directly** inside any of the listed folders. Multiple IDs allowed. **Non‑recursive** (subfolders are ignored).                                                                                                             |
### Folder filtering

**Quick start**

```js
const CONFIG = {
  // ...
  driveFolderIds: [
    "<FOLDER_ID_1>",
    "<FOLDER_ID_2>"
  ]
};
```

#### How to get the folder ID

1) **Through the browser (URL bar)**
   - Open the folder in Google Drive (web).
   - Copy the part **after** `/folders/` in the URL.
   - Example: 
     `https://drive.google.com/drive/folders/1aBcD2E3fG4hiJ5k6LMnOpQRs7tuVXy-8` → **ID** = `1aBcD2E3fG4hiJ5k6LMnOpQRs7tuVXy-8`

2) **By using "Get link"**
   - Right-click on the folder → **Get link** → **Copy link**.
   - The ID is the long string **between** `/folders/` and the `?` (if present) in the copied link.

3) **Shared Drives**
   - Same process: open the desired folder in the shared drive and copy the ID from the URL.

> **Notes**
> - The filter is **non-recursive**: subfolders are **not** automatically included. Add each subfolder in `driveFolderIds` if you want to include it.
> - You can list **multiple** folder IDs in `driveFolderIds`.
> - Make sure the service account has access to these folders; otherwise, nothing will be returned.