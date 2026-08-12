# Aws-workshop-template

## Repo structure

```bash
.
├── contentspec.yaml                  <-- Specifies the version of the content
├── README.md                         <-- This instructions file
├── static                            <-- Directory for static assets to be hosted alongside the workshop (ie. images, scripts, documents, etc) 
└── content                           <-- Directory for workshop content markdown
    └── index.en.md                   <-- At the root of each directory, there must be at least one markdown file
    └── introduction                  <-- Directory for workshop content markdown
        └── index.en.md               <-- Markdown file that would be render 
```

## What's Included

This project contains the following folders:
* `static`: This folder contains static assets to be hosted alongside the workshop (ie. images, scripts, documents, etc) 
* `content`: This is the core workshop folder. This is generated as HTML and hosted for presentation for customers.

## How to create content

Under the `content` folder, Each folder requires at least one `index.<lang>.md` file. The file will have a header

```aidl
+++
title = "AWS Workshop Template"
weight = 0
+++
```

The title will be the title on navigation panel on the left. The weight determines the order the page appears in the navigation panel.

## Local Development

OS X
Download the appropriate local content preview application for your operating system:

https://artifacts.us-east-1.prod.workshops.aws/v2/cli/osx/preview_build 

Open Terminal and change to the location of the preview_build app with cd YourDirectory
Make the binary executable by running chmod +x preview_build. Reference 
In the Finder on your Mac, locate the downloaded app
Don’t use Launchpad to do this. Launchpad doesn’t allow you to access the shortcut menu.
Control-click the app icon, then choose Open from the shortcut menu
Click Open
Launch the preview application from your CLI by providing the relative or absolute path, such as running ~/Downloads/preview_build in Terminal

The preview will now be available at http://localhost:8080
http://localhost:8080 

Notes:
Make changes to your content and view them in your local browser. Your browser will refresh automatically when file changes are detected in the content repository.
Watch the terminal window where you're running the preview utility to see information about build failures, warnings, and other tips.

Tips:
You can specify a custom port with the -port flag, such as ./preview_build -port 8081
You can run the preview server from a different location by providing a relative or full path to your content repository such as ./preview_build -input Path/To/MyContentRepo or .\preview_build.exe -input C:\Users\YourAlias\Desktop\MyContentRepo
