OneDrive on Bash

Prerequisites
-------------

- `curl` is used for accessing the API.
- `grep` is used for filtering the parsed json output.
- `cut` is used for value extraction from the filtered json output.
- `xargs` is used for multi-threaded file uploads.
- `dd` is used for chunked file uploads.
- `stat` is used to determine the filesize.

Getting started (OneDrive Personal)
---------------

Before you can use this tool, create an application in the [Microsoft account Developer Center](https://apps.dev.microsoft.com/#/appList/create/sapi) to generate your custom Client ID and Client secret. Notice, that you need to add a new platform of type `Mobile application`.

Afterwards your overview should show your freshly created credentials in the form

    Application ID: 00000000A2B3C495
    Applications secrets: qOFCYaZKjm6e13aq3fdGiNz

Please ensure that your application is listed below `Live SDK applications` and that the Application ID (Client ID) does not contain any dashes. The newer applications are not (yet) supported by the `onedrive-authorize`.

Please insert these values in the matching variables in `onedrive.cfg`:

    export api_client_id="00000000A2B3C495"
    export api_client_secret="88qC3kX2Cbd0tXV2sBnqYbS321abcDEF"

After the initial configuration you must authorize the app to use your OneDrive account. Run

    $ onedrive -a

and follow the steps. You will need a web browser.

After the authorization process has successfully completed you can upload files.


Usage (OneDrive Personal)
-----

To upload a single file simply type

    $ onedrive file1

You can also upload multiple files, either by explicitly specifying each one

    $ onedrive file1 file2

or just use wildcards (globbing)

    $ onedrive file*.png

You can also specify a destination folder relative to the root folder configured in `onedrive.cfg`:

    $ onedrive -f "relative/path" file1

This command will automatically determine all of the needed folder ids and recursively create all subfolders that do not yet exist.

If you need your file to be uploaded with a different filename, you can activate the renaming mode:

    $ onedrive -r ./file1.txt renamed_file1.txt ./file2.txt renamed_file2.txt

Be aware that for each file you specify you must provide the remote filename as the subsequent parameter. This feature can lead to an unexpected behavior when combined with wildcards (globbing) because the pathname expansion is performed by bash before the execution of the script.


Configuration
-------------

### Specify an alternate root folder for uploads

If you want to use a folder other than the root folder of your OneDrive as your upload root folder, you need to retrieve its unique id. 

Open your the [OneDrive Web Interface](https://onedrive.live.com) and navigate to the folder, you want to configure as your new upload folder. The address bar should now show a URL like

    https://onedrive.live.com/?cid=123419ACDA5678AB&id=123419ACDA5678AB%2145321

The GET-parameter `id` is the unique folder id you need to place in your `onedrive.cfg`. You have to work with the decoded URL, e.g. `%21` decodes to `!`:

    export api_root_folder="items/123419ACDA5678AB!45321"

Unfortunately, you can't get item ID in this way for OneDrive for Business. So use another approach:

    Upload in debug mode any file to non-existent folder, which will be used as root folder.

    ./onedriveb -d -f Backup testfile
    2016-08-26 09:33:56 Searching for 'Backup' in ''
    2016-08-26 09:33:56 Creating folder 'Backup' in ''
    2016-08-26 09:33:58 api_folder_id is now '01KB37ZVWEMG6F4SS6TVGZAM7ACZKOEERY'
    2016-08-26 09:33:58 Size of testfile is less than or equal to 104857600 bytes, will use simple upload
    2016-08-26 09:34:01 Uploading 'testfile' as 'testfile' into 01KB37ZVWEMG6F4SS6TVGZAM7ACZKOEERY

    Copy `api_folder_id` and place it in your `onedriveb.cfg`

    export api_root_folder="items/01KB37ZVWEMG6F4SS6TVGZAM7ACZKOEERY"

You can also specify any of the [special folders](https://dev.onedrive.com/items/special_folders.htm) provided by the API by simply using

    export api_root_folder="special/approot"
    export api_root_folder="special/documents"
    export api_root_folder="special/photos"
    ...

The default is to use

    export api_root_folder="root"

which identifies the root folder of your account.

Keep in mind that this will be the new root folder of your OneDrive as seen by this script. The script does *not* support escaping the root by using `../`.

### Less permissions for uploads

If you don't want the script to have access to all your folders and files you can request less permissions during the authorization process. Simply change the value

    export api_permissions="onedrive.readwrite"

to

    export api_permissions="onedrive.approot"

and run the authorization process. You might need to remove the apps permissions prior to that ([Link](https://account.live.com/consent/Manage)) if you have already requested `onedrive.readwrite` permissions.

### Number of simultaneous uploads

If the script does not fully utilize your bandwidth, you can maybe speed things up a little bit by increasing the value of `max_upload_threads`.

When you start the upload of more than one file, the script will start up to `max_upload_threads` parallel uploads, but only one thread per file.

### Configure threshold for session based upload

For small files the script uses the [simple upload](https://dev.onedrive.com/items/upload_put.htm) of the api. For files that are larger than 100 MiB the script uses the [session based upload](https://dev.onedrive.com/items/upload_large_files.htm).

If you want to use the session based upload for files smaller than 100 MiB you can change the value of `max_simple_upload_size` to any positive value smaller than 104857600:

    export max_simple_upload_size=52428800

You can also change the value of `max_chunk_size` to any positive value smaller than 62914560, if you want to use smaller or larger chunks than the default:

    export max_chunk_size=62914560
