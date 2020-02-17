
<!-- Start Document Outline -->

* [Introduction](#introduction)
* [DocFx installation](#docfx-installation)
* [A test solution in Visual Studio](#a-test-solution-in-visual-studio)
* [Setting up DocFx with docfx init](#setting-up-docfx-with-docfx-init)
* [Setting up DocFx manually](#setting-up-docfx-manually)
* [The anatomy of the docfx.json configuration file](#the-anatomy-of-the-docfxjson-configuration-file)
	* [The metadata section](#the-metadata-section)
	* [The build section](#the-build-section)
	* [The docs folder](#the-docs-folder)
* [The filterConfig.yml file](#the-filterconfigyml-file)
* [Conceptual documentation and the TOC (Table Of Content) files](#conceptual-documentation-and-the-toc-table-of-content-files)
* [The .gitignore file](#the-gitignore-file)
* [Generate the documentation site](#generate-the-documentation-site)
* [Publish the generated documentation site to GitHub Pages](#publish-the-generated-documentation-site-to-github-pages)
* [The PDF adventure](#the-pdf-adventure)

<!-- End Document Outline -->

## Introduction

[DocFx](https://dotnet.github.io/docfx/) is a command line tool that generates documentation. 

DocFx builds a documentation web site combining two sources: 
- **reference** documentation it collects from comments found in source code files
- **conceptual** documentation provided to DocFx as [Markdown](https://en.wikipedia.org/wiki/Markdown) files, by the user.

According to [DocFx](https://dotnet.github.io/docfx/) web site 
> "DocFX can produce documentation from source code (including .NET, REST, JavaScript, Java, Python and TypeScript) as well as raw Markdown files."

and also

> "DocFX can run on Linux, macOS, and Windows. The generated static website can be deployed to any host such as GitHub Pages or Azure Websites with no additional configuration."

DocFx is an [open source](https://github.com/dotnet/docfx) tool developed by Microsoft and, [as the company says](https://docs.microsoft.com/en-us/teamblog/announcing-unified-dotnet-experience-on-docs#built-with-open-source-in-mind), is a tool used in building Microsoft's documentation [web site](https://docs.microsoft.com). 

In this post we'll use DocFx to produce documentation for a [Visual Studio](https://visualstudio.microsoft.com/) [C#](https://en.wikipedia.org/wiki/C_Sharp_(programming_language)) solution in a Windows machine.

## DocFx installation
The easiest way to download and install DocFx is to use the [Chocolatey](https://teonotebook.wordpress.com/2019/03/16/chocolatey-a-package-manager-for-windows/) package manager for Windows. Open a terminal as administrator and execute the following

```
choco install docfx -y
```
The above adds the DocFx to the `PATH` environment variable too.

## A test solution in Visual Studio
Open Visual Studio and create a solution with two projects: a Library project and a Windows Forms project. 

Here is the folder structure

```
Solution
    + App
    + Lib
    Solution.sln
```

For each project go to `Properties | Build` and check the `XML documentation file` check box.

Add some classes to both projects and document those classes. This is done by adding triple-slash comments to classes, methods and properties.

Build the solution.

## Setting up DocFx with `docfx init`

The DocFx documentation provides [two walkthroughs](https://dotnet.github.io/docfx/tutorial/walkthrough/walkthrough_overview.html). 

Those walkthroughs say that we `init` the DocFx by opening a terminal, `cd` to solution folder and then typing `docfx init -q` to initialize the project.

```
cd C:\Solution
docfx init -q
```

The `-q` parameter is for `quiet`. Otherwise it goes through a series of questions the user has to answer. 

The above creates a `docfx_project` folder inside the root solution folder and adds a number of sub-folders and files to it. The most important file inside that `docfx_project` is the `docfx.json` file which is the configuration file for the DocFx. 

> "docfx.json is the configuration file docfx uses to generate documentation"

All folder references inside that generated `docfx.json` are wrong. And some of the created folders are not needed at all.

Therefore delete the `docfx_project` folder. We are going to use are own way.

## Setting up DocFx manually

We need a sub-folder inside the root solution folder for the DocFx files and generated documentation. Create the new folder and name it `DocFx`.

```
Solution
    + App
    + DocFx
    + Lib
    Solution.sln
```

Inside `DocFx` folder create an empty `docfx.json`, open it with Visual Studio and add the following content.

```
{
    "metadata": [
        {
            "src": [
                {
                    "files": [ "**/**.csproj" ],
                    "src": ".."
                }
            ],
            "dest": "reference",
            "disableGitFeatures": false,
            "disableDefaultFilter": false,
            "filter": "filterConfig.yml"
        }
    ],
    "build": {
        "content": [
            {
                "files": [
                    "reference/**.yml",
                    "reference/index.md"
                ]
            },
            {
                "files": [
                    "Concepts/toc.yml",
                    "Concepts/**.md",
                    "Concepts/**/toc.yml",
                    "toc.yml",
                    "*.md"
                ]
            }
        ],
        "resource": [
            {
                "files": [ "images/**" ]
            }
        ],
        "dest": "../docs",
        "globalMetadataFiles": [],
        "fileMetadataFiles": [],
        "template": [ "default" ],
        "postProcessors": [],
        "markdownEngineName": "markdig",
        "noLangKeyword": false,
        "keepFileLink": false,
        "cleanupCacheHistory": false,
        "disableGitFeatures": false
    }
}
```

## The anatomy of the `docfx.json` configuration file

The `docfx.json` contains two sections: `metadata` and `build`. There can be a third section, `pdf`, but we leave that ...adventure for a later time.

### The `metadata` section

The `metadata` section says to DocFx 
- where to find source code files, to gather comments from
- where to place the gathered material
- and how to filter inderited members of types found in source files.

Our `metadata` section says to DocFx
- to look for source code files in all `csproj` files (`"files": [ "**/**.csproj" ]`) starting from the rool solution folder (`"src": ".."`)
- place the gathered material to a folder named `reference` (`"dest": "reference",`)
- and filter inherited members according to a provided `yml` file (`"filter": "filterConfig.yml"`)

The `reference` folder will be created by DocFx if not there. 

### The `build` section

The `build` section says to DocFx
- where to find the content to build, for both types, the content DocFx gathered from source files (**reference**) and the content provided by the user as Markdown files (**conseptual**)
- where to find the images used in Markdown files
- where to place the "compiled" output, meaning the web site it generates
- and what template to use

Our `build` section says to DocFx
- the **reference** content is found in the `reference` folder while the **conseptual** is found in the `Concepts` folder
- the images are in the `images` folder
- place the generated site to the `../docs` folder `"dest": "../docs"`
- and use the `default` template

### The `docs` folder

We instruct DocFx to place the generated web site in a `docs` folder, under the root solution folder. This will result in the following folder structure.

```
Solution
    + App
    + DocFx
    + docs
    + Lib
    Solution.sln
```

You can instruct DocFx to place the generated site in any folder you like. 

We name that folder `docs` and place it under the root solution folder because [GitHub Pages](https://pages.github.com/) wants it like that. 

If you use [github](https://github.com/) to host your open source project and you want to provide a nice documentation site for your project, you can achive that easily by simply placing the documentation site inside the `docs` folder under the root and setting that `docs` folder as the [publishing source](https://help.github.com/en/github/working-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) for the GitHub Pages site.


## The `filterConfig.yml` file

As the [DocFx documentation](https://dotnet.github.io/docfx/tutorial/howto_filter_out_unwanted_apis_attributes.html) says

> "A filter configuration file is in YAML format. You may filter out unwanted APIs or attributes by providing a filter configuration file and specifying its path."

Place a `filterConfig.yml` with the following content

```
apiRules:
- exclude:
    # inherited members from Form
    uidRegex: ^System\.Windows\.Forms\.Form\..*$
    type: Member
- exclude:
    # inherited members from Control
    uidRegex: ^System\.Windows\.Forms\.Control\..*$
    type: Member	
- exclude:
    # mentioning types from System.* namespace
    uidRegex: ^System\..*$
    type: Type	
- exclude:
    # mentioning types from Microsoft.* namespace
    uidRegex: ^Microsoft\..*$
    type: Type
```

> **CAUTION**: the lines containg the `uidRegex` and `type` entries should start with **two spaces**. [YAML](https://en.wikipedia.org/wiki/YAML) language uses **white space identation**.

## Conceptual documentation and the TOC (Table Of Content) files

DocFx accepts Markdown files containing conceptual documentation, written by the user. Organizes those Markdown files using TOC (Table Of Content) YAML files.

Under the `DocFx` folder add an `index.md` file with whatever content you like.

Here is my `DocFx` folder
```
DocFx
    docfx.json
    index.md
    .gitignore
    filterConfig.yml
    toc.yml
```

Inside the `DocFx` folder create a `Concepts` sub-folder and add the following folders and files

```
Concepts
    + Advanced
        Abstract.md
        Advanced.md
        toc.yml
    + Easy
        Abstract.md
        Easy.md
        toc.yml
    Abstract.md
    toc.yml
```

You may place whatever content you like in these Markdown files. Usually those files contain **conceptual** overviews regarding the use of the **referenced** [API](https://en.wikipedia.org/wiki/Application_programming_interface).

Regarding `TOC` files you may consult the relevant [DocFx documentation](https://dotnet.github.io/docfx/tutorial/intro_toc.html) which says that

> "DocFX supports processing Markdown files or Conceptual Files, as well as structured data model in YAML or JSON format or Metadata Files. Besides that, DocFX introduces a way to organize these files using Table-Of-Content Files or TOC Files, so that users can navigate through Metadata Files and Conceptual Files."

Here is the three `toc.yml` files used

1. `Concepts/toc.yml`

```
- name: Easy
  href: Easy/toc.yml
  topicHref: Easy/Abstract.md
- name: Advanced
  href: Advanced/toc.yml
  topicHref: Advanced/Abstract.md
```

2. `Concepts/Advanced/toc.yml`

```
- name: Advanced Overview Title
  href: Advanced.md
```

3. `Concepts/Easy/toc.yml`

```
- name: Easy Overview Title
  href: Easy.md
```

- The `name` entry is the clickable title, i.e. link, to be displayed by the TOC of the generated site.

- The [`href`](https://dotnet.github.io/docfx/tutorial/intro_toc.html#href-in-detail) entry says where to navigate when that title is clicked. 

- The optional `topicHref` says what content file to display. Used when the `href` links to another `toc.yml`, that is another Table Of Contents, but the user wants to provide some kind of abstration to the visitor, as to what is going to find in that link.

## The .gitignore file

The `docfx init -q` command adds a `.gitignore` file inside the `DocFx` folder. Create a `.gitignore` file with the following content and place it into the `DocFx` folder.

```
/**/DROP/
/**/TEMP/
/**/packages/
/**/bin/
/**/obj/
reference

```

## Generate the documentation site

In order to generate the web site, open a terminal, `cd` to `DocFx` folder and just type
```
docfx
```
The web site is generated. 

It's time to preview the site. According to [documentation](https://dotnet.github.io/docfx/tutorial/walkthrough/walkthrough_create_a_docfx_project.html#step4-preview-our-website)

> "If port 8080 is not in use, docfx will host _site under http://localhost:8080. If 8080 is in use, you can use docfx serve _site -p <port> to change the port to be used by docfx."

To preview the site, `cd` to `docs` 
```
cd ../docs
```
and then type
```
docfx serve
```
or if the port `8080` is taken by another application, just use another port
```
docfx serve -p 8081
```
Now the documentation site is running.

Open a browser and navigate to `http://localhost:8080`.

Alternatively, if you place the folder of the generated site under the `DocFx` folder, you may build and run the site with just a signle line.
```
docfx --serve
```

You may delete the generated folders, `reference` and `docs`. They are recreated in each build.

## Publish the generated documentation site to GitHub Pages

- `push` your local git repository to your github remote repository
- in github repository go to `Settings` (it's far right with the gear icon)
- scroll down to `GitHub Pages` section
- select `master brach /docs folder` as Source
- do **NOT** select theme

That's all. Your documentation site will be available soon, if not immediately, at

    `https://USER_NAME.github.io/PROJECT_NAME/`

## The PDF adventure

DocFx can generate a single PDF file, for the whole generated documentation. Not without [problems](https://github.com/dotnet/docfx/issues/5490).

DocFx generates the PDF output using the [wkhtmltopdf](https://wkhtmltopdf.org/) tool.

To download and install wkhtmltopdf open a terminal as administrator and type
```
choco install wkhtmltopdf -y
```

In order to generated the coveted PDF file, the user has to read and ...understand the relevant documentation provided by DocFx. 

One piece of that information can be found in the [user manual](https://dotnet.github.io/docfx/tutorial/docfx.exe_user_manual.html#24-generate-pdf-documentation-command-docfx-pdf) while the other can be found in the [third walkthrough](https://dotnet.github.io/docfx/tutorial/walkthrough/walkthrough_generate_pdf.html).
 
Here is my way, after a lot of digging and experimenting.

- add a `PDF` folder inside `DocFx` folder.
- add the following `toc.yml` inside `PDF` folder
```
- name: Concepts
  href: ../Concepts/toc.yml
- name: Reference
  href: ../reference/toc.yml 
```
- add a `pdf` section in the `docfx.json` with the following content
```
    "pdf": {
        "content": [
            {
                "files": [ "PDF/toc.yml" ]
            },
            {
                "files": [
                    "reference/**.yml",
                    "reference/index.md"
                ],
                "exclude": [
                    "**/toc.yml",
                    "**/toc.md"
                ]
            },
            {
                "files": [
                    "Concepts/**.md",
                    "Concepts/**/toc.yml",
                    "toc.yml",
                    "*.md"
                ],
                "exclude": [
                    "**/bin/**",
                    "**/obj/**",
                    "PDF/**",
                    "**/toc.yml",
                    "**/toc.md"
                ]
            }
        ],
        "resource": [
            {
                "files": [ "images/**" ],
                "exclude": [
                    "**/bin/**",
                    "**/obj/**",
                    "PDF/**"
                ]
            }
        ],
        "dest": "_pdf",
        "outline": "NoOutline"
    }
```
- open a terminal, `cd` to `DocFx` folder and type
```
docfx pdf
```

This will generate a PDF file, without outline, meaning PDF TOC. Obviously there is a problem and the outline is not correctly generated. So I deactivated using the `"outline": "NoOutline"`.

**Tested on:**
Windows 10
docfx 2.48.1.0
wkhtmltopdf 0.12.5 (with patched qt)





 

















 