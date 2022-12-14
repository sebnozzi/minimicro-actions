
# MiniMicro (GitHub) Actions

This repository aims to be a collection of GitHub workflows / actions useful for dealing with MiniMicro artifacts.

It consists of these workflows:

1. Deploying multi-platform builds to itch.io (including web)
2. Uploading a web-build to a web-server via FTP

## Deploying to itch.io (workflow)

This workflow packages a MiniMicro project for the available platforms (Windows, Mac, Linux) and deploys the generated deliverables to an itch.io project.

It does does so by automatizing the steps described here:

https://miniscript.org/wiki/How_to_package_a_Mini_Micro_game

It assumes that:

1. Your project is hosted on GitHub
2. You have an [itch.io](https://itch.io/dashboard) account

In order to use this workflow follow these instructions:

### 1. Obtain API key from itch.io.

Go to https://itch.io/user/settings/api-keys and generate an API key.

### 2. Create secret on repo

On your GitHub repository, go to:

* "Settings", then
* "Secrets", then
* "Actions", then
* "New repository secret"

Now you need to choose a name for the secret (e.g. `ITCH_IO_API_KEY`) and paste the value generated over at itch.io.

### 3. <a name="creating-project"></a>Create and setup project on itch.io

If you have not done so already, you need to create a new project at itch.io to deploy to.

In the [dashboard](https://itch.io/dashboard) click on "Create new project".

* Provide some "Title". This is only for displaying.
* Provide a "Project URL". **This is important**. This will be later referred to as the "game-id" when deploying.
* As a "Kind of project" choose "HTML".
* In "Embed options":
  * Leave "Embed in page" and "Manually set page size"
  * Set the "Viewport dimensions" to 1024x768.
  * In the "Frame options" you _might_ want to activate the "Fullscreen button"

For the time being you won't be uploading any files. You will return here later after the first deployment to adjust the "browser" channel deployment.

### 4. Create up a workflow file

Your GitHub project needs to have a ".github" folder, inside of it a "workflows" folder and inside it a workflow file.

The workflow file is a YAML file. You can choose the name you want. One example would be: `publish.yaml`.

Then it would look like this:

```
.github/workflows/publish.yaml
```

### 5. Configure the workflow file

Here is a sample content of the workflow file, for illustration purposes only:

```yaml
# Choose the name you want
name: Publish

on:
  push:
    branches:
      - master
  # Leave if you want to trigger builds manually
  workflow_dispatch:

jobs:

  publish:
    uses: sebnozzi/minimicro-actions/.github/workflows/itch-deploy.yaml@main
    with:
      minidisk_main_file: demo.ms
      minidisk_additional_entries: >-
        morning-mood-autumn.jpg
        window_1280.png
        fogmask.png
      boot_opts_path: bootOpts.grfon
      custom_executable_name: FoggyWindow
      itch_io_username: sebnozzi
      itch_io_game_id: foggy-window
      itchio_web_channel: html5
    secrets:
      itch_io_api_key: ${{ secrets.ITCH_IO_API_KEY }}
```

The relevant part is what happens in the "publish" job above (you can choose another name for the job). There it invokes the reusable workflow `itch-deploy.yaml` over at the `minimicro-actions` repository of user GitHub account `sebnozzi`.

The line in "uses" has to be written as it is.

The "with" sections contains the parameters that can be passed to the workflow, which are:

| Name    | Required | Description |
|---------|---------|----------------------------------------------------------------------|
| **minidisk_main_file** | Yes | The main file that will be loaded and run to start your game / app |
| minidisk_additional_entries | No | Additional files or top-level folders to be included as part of the game / app. Folder contents will be added recursively.<br/>The entries are specified as one string, separated by spaces (e.g. `file1.ms file2.ms someFolder`). To enter entries in different lines, you can make use of the YAML's `>-` [block style](https://yaml-multiline.info/) indicator, which will compact all lines in one string removing newlines. |
| boot_opts_path | No | Path to a `bootOpts.grfon` file to [configure how MiniMicro runs](https://miniscript.org/wiki/BootOpts.grfon). The name of the file MUST be "bootOpts.grfon". |
| custom_executable_name | No | Optionally the "MiniMicro" executable can be renamed to the name provided. E.g. for "My Game" the executables will result in "My Game.exe" (Windows) and "My Game.app" (Mac). |
| **itch_io_username** | Yes | Your username on itch.io |
| **itch_io_game_id** | Yes | The "game-id" chosen when [creating the project](#creating-project) at itch.io. <br/>Last part of the game's URL. |
| itchio_web_channel | No | Name to use for the "browser playable" version of the game. If not specified "browser" will be used. After the first deployment the file on that channel has to be manually set to "browser playable" on itch.io. |

Finally, the **GitHub secret** needs to be passed in the `secrets` section. The parameter is always "itch_io_api_key", but be sure to choose the name of the secreat as you configured it in your repository in the expression `${{ secrets.NAME_OF_YOUR_SECRET }}`

### 5. Make a first-time deployment

Initially, itch.io does not "know" that a deployment is meant to be run on the browser. One needs to make a first-time deployment and then specify deployments on that "channel" to be browser-playable. After that, subsequent deployments to that channel remain browser-playable.

In this step, deploy your game / application for the first time. In the next step we will "fix" the uploaded asset for the html / web / browser channel.

### 6. Fix the browser channel (first-time only)

In the project page, locate the "Uploads".

Locate the one which should be playable on the browser. If you did not specify a channel-name for this, it should have the "browser" channel.

Mark the checkbox for "This file will be played on the browser".

Save the project.

### 7. Deploy as needed

After this, everything should be setup.

Depending on the [triggers selected for the workflow](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) you would automatically deploy to itch.io each time the "main" branch changes, or when manually triggering a build.

## FTP Uploading

It assumes that:

1. You have a web-server running
2. Which you can upload files to via FTP

You must know suitable user / password to authenticate to the FTP server.

In order to use this workflow follow these instructions:

### 1. Create secrets on repo

On your GitHub repository, go to:

* "Settings", then
* "Secrets", then
* "Actions"

You will create secrets by clicking on "New repository secret".

Create these secrets:

1. One for your FTP-account user-name
2. Another your FTP-account password

Name these secrets however you want, but remember the names.

### 2. Create up a workflow file

Your GitHub project needs to have a ".github" folder, inside of it a "workflows" folder and inside it a workflow file.

The workflow file is a YAML file. You can choose the name you want. One example would be: `publish.yaml`.

Then it would look like this:

```
.github/workflows/publish.yaml
```

### 3. Configure the workflow file

Here is a sample content of the workflow file, for illustration purposes only:

```yaml
name: Publish

on:
  push:
    branches:
      - master

jobs:

  publish:
    uses: sebnozzi/minimicro-actions/.github/workflows/ftp-web-deploy.yaml@main
    with:
      page_title: "Feed the Wumpus! | MiniMicro"
      minidisk_main_file: game.ms
      minidisk_additional_entries: '"kitchen Background Lite.png"'
      ftp_host: "sebnozzi.com"
      ftp_target_path: "/public_html/demos/mini-micro/feed-the-wumpus/"
    secrets:
      ftp_username: ${{ secrets.FTP_USER }}
      ftp_password: ${{ secrets.FTP_PASS }}

```

The line in "uses" has to be written as it is.

The "with" sections contains the parameters that can be passed to the workflow, which are:

| Name    | Required | Description |
|---------|---------|----------------------------------------------------------------------|
|page_title|No|Title to give to the HTML page in which the project runs. If not specified the default title of the web-template will be used.|
| **minidisk_main_file** | Yes | The main file that will be loaded and run to start your game / app |
| minidisk_additional_entries | No | Additional files or top-level folders to be included as part of the game / app. Folder contents will be added recursively.<br/>The entries are specified as one string, separated by spaces (e.g. `file1.ms file2.ms someFolder`). To enter entries in different lines, you can make use of the YAML's `>-` [block style](https://yaml-multiline.info/) indicator, which will compact all lines in one string removing newlines. |
| boot_opts_path | No | Path to a `bootOpts.grfon` file to [configure how MiniMicro runs](https://miniscript.org/wiki/BootOpts.grfon). The name of the file MUST be "bootOpts.grfon". |
|ftp_host|Yes|Host-address of the FTP server. Note: host ONLY; you don't need to include the protocol part like "ftp://"|
|ftp_target_path|Yes|Path / directory in the server to which to upload the files. NOTE that it HAS to end with a trailing slash ("/path/" is OK, "/path" is NOT).|

Finally, the **GitHub secrets** (FTP user / password) need to be passed in the `secrets` section.

| Name    | Description |
|---------|----------------------------------------------------------------------|
|ftp_username|User-name of your FTP account|
|ftp_password|Password of your FTP account|

### 4. Deploy as needed

After this, everything should be setup.

Depending on the [triggers selected for the workflow](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows) you would automatically upload via FTP each time the "main" branch changes, or when manually triggering a build.

## End words

Hopefully these reusable workflows help in sharing your MiniMicro creations with the world.

Issues and suggestions welcome.

Happy coding!
