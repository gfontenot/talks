# Liftoff
#### https://github.com/thoughtbot/liftoff

^ I wanted to talk to you a bit about a project I've been working on called Liftoff.

---

## We create a lot of projects

^ At thoughtbot, we create projects, a lot.
- Client apps
- Internal apps
- Test apps
- Open source projects

---

## Unfortunately, Xcode

^ Xcode's project generation doesn't work the way we want it to. We need to do a ton of extra configuration after creating a project.
- Common build settings
- setting indentation at the project level
- CocoaPods setup
- The big one: Directories vs Groups

---

> But you can create Xcode template projects!
-- you, probably, right now

^ you _can_, but do you really want to?

---

## Xcode template projects suck

^ It's hard to do common things with Xcode template projects
- Create them
- Customize them
- Share them

---

## You are a beautiful snowflake

^ project setup is a personal process. Decisions need to be made at multiple levels (global defaults, one-time decisions).

^ Liftoff's goal is to make it faster and easier to set up projects while also improving your ability to customize your projects to your exact needs.

---

## Installation

```
❯ brew tap thoughtbot/formulae
❯ brew install liftoff
```

Without xcproj:

```
❯ brew tap thoughtbot/formulae
❯ brew install liftoff --without-xcproj
```

^ xcproj is a command line utility for modifying Xcode projects. It's not required, but without it, Xcode might convert your project to a different format after opening it, resulting in a dirty repo

---

## Usage

```
❯ liftoff
Project name? Liftoff
Company name? thoughtbot
Company identifier? |com.thoughtbot|
Prefix? LIFT
```

^ Running liftoff in a directory:
- Creates a directory structure with the project name
- Installs template files
- Creates an Xcode project file
- Sets the company name, identifier, prefix
- Runs pod install against the default Podfile
- Sets project indentation
- Adds more warnings
- turns on warnings as errors for the release build
- Adds a shell script for TODO and FIXME comments
- enables the static analyzer

---

## Specifying options on the command line

```
usage: liftoff [-v | --version] [-h | --help] [config options]
    -v, --version                    Display the version and exit
    -h, --help                       Display this help message and exit
        --[no-]strict-prompts        Enable/Disable strict prompts
        --[no-]cocoapods             Enable/Disable Cocoapods
        --[no-]git                   Enable/Disable git
        --no-open                    Don't open Xcode after generation
        --template [TEMPLATE NAME]   Use the specified project template
    -t, --indentation N              Set indentation level
    -n, --name [PROJECT_NAME]        Set project name
    -c, --company [COMPANY]          Set project company
    -a, --author [AUTHOR]            Set project author
    -p, --prefix [PREFIX]            Set project prefix
    -i, --identifier [IDENTIFIER]    Set project company ID (com.example)
```

^ Obviously, some of the configuration might not be what you want. We can set a number of options on the command line by passing them to `liftoff`

---

## Command line options (kind of) suck

^ 
- Can't add everything to that interface (there's already too much)
- I don't want to pass them every time.

---

## Customization

Configuration through the `liftoffrc` file:

```
❯ vim ~/.liftoffrc
❯ vim ./.liftoffrc
```

View available options:

```
❯ man liftoffrc
```

^ creating a liftoffrc lets you define a huge number of settings
- default project template
- define warnings
- add template files
- create and override project templates
- author, company, identifier info
- arbitrary per-build config settings

^ Can be implemented at the user level, or the current directory level.  Fallback order = Local -> Global -> Default

---

## Custom Templates:

```
❯ tree .liftoff/templates
.liftoff/templates
└── AppDelegate.swift
└── MySuperCoolTemplate.swift
└── Model.xcdatamodeld
    └── Model.xcdatamodel
        └── contents
```

^ Template files override the default, follow the same rules for fallback. You can also add your own templates in order to completely customize your project templates

---

## Sharing configurations is simple

^
- For most shared configurations, all you have to share is the liftoffrc
- Even for complex configurations, you can easily share templates
- Since local templates trump global ones, company policies don't need to affect your personal development

---

## The Future

^
- OS X
- even more customization
  - Doesn't play well with all arbitrary configuration options (Crashlytics needs to be hardcoded)
  - Want to open up dependency management to be easier to deal with
  - fewer hard coded options, more exposed hooks for arbitrary code execution
- Ability to create project inside a specific directory
- Swift?
  - Ruby doesn't feel like the right language for this. I'd rather make it easy for people using the tool to be able to contribute back to it

---

### Gordon Fontenot
#### https://github.com/thoughtbot/liftoff
#### @gfontenot (basically everywhere)
#### http://buildphase.fm
#### @thoughtbot
