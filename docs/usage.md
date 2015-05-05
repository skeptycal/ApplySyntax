# User Guide {: .doctitle}
Configuring and using ApplySyntax.

---

# Overview
ApplySyntax is based on the idea of creating rules for applying a certain syntaxes to specific files. You define the rules, the plugin checks them. The first one to pass wins.

Create your own custom rules in `Packages/User/ApplySyntax.sublime-settings`. The easiest way to get started is to just copy the default settings file found in `Packages/ApplySyntax/ApplySyntax.sublime-settings` to your `Packages/User` directory and modify it to meet your needs. Make sure you remove the `default_syntaxes` key and create a new one called `syntaxes`. If you don't, you will overwrite the default syntaxes and they and you won't get updates to them.

# Creating Rules

Each rule is a dictionary within the syntax array.  Let's take a look at the top level parameters.

## Name
`name` is the syntax file that will be applied to a view which meets the criteria defined in the rule.

For syntax files you must specify the path to the syntax file. The plugin is capable of supporting multiple levels of folder nesting if you need it to. For example, if you had all of your tmLanguage files for Rails organized in a folder like this: `Packages/Rails/Language/*.tmLanguage`, and you were looking to use the `Ruby Haml.tmLanguage` file, the path to name translation would simply be: `Packages/Rails/Language/Ruby Haml.tmLanguage` -> `Rails/Language/Ruby Haml`.

```js
"name": "Rails/Language/Ruby Haml"
```

Notice that the paths are relative to the `Packages` folder.  Also, notice that we don't specify the extension.  Sublime Text in build 3084 added a new language syntax with the extension `sublime-syntax`.  In Sublime builds >= 3084, ApplySyntax will first default to `sublime-syntax` and fall back to `tmLanguage` if it cannot find the the other format.  If you want to force the syntax, just specify the extension; the extension must be either `sublime-syntax` or `tmlanguage`.

```js
"name": "Rails/Language/Ruby Haml.tmLanguage"
```

If it is desirable for the syntax rule to reference multiple tmLanguage files because it is not known which package will be on a machine, you can set the syntax as an array of names as shown in the following example.  The first one found will be used.

```js
"name": ["RSpec/RSpec", "RSpec (snippets and syntax)/Syntaxes/RSpec"]
```

Notice that each syntax file has a different path since they come from completely different plugins.

## Extensions
`extensions` is a convenience option to add a given set of extensions to your language settings file.  By adding the extension to the language settings file, sidebar icons in ST3 will display the proper icon, and files will load with the proper syntax via Sublime's default extension detection method.  Keep in mind though that other rules can override this.  As `extensions` isn't really a rule, but just a list which ApplySyntax uses to automatically add extension to the language settings file.  A hit (match) on this array won't currently stop ApplySyntax from processing more rules (this may change in the future).

Apply syntax will create a file `ApplySyntax.ext-list` in your `User` folder and track which extension it added so that if you remove a rule, ApplySyntax will only remove the extensions it added to the language file in question.

If you do not like this functionality, you can simply disable it by setting the following option in your settings file to `false`:

```js
    "add_exts_to_lang_settings": false,
```

!!! note "Note":
    `extensions` will not work in a [project specific rule](#project-specific-rules).

## Match
`match` is a setting that you either include or omit.  When included, you set it to `all`.  When set, all rules defined must be met for a match to be considered successful.  `match` ignores the `extensions` key as it is not actually a rule.

```js
    "match": "all"
```

So in this case, all the rules must match for the syntax to be applied:

```js
     "name": "Handlebars/Handlebars",
     "match": "all",
     "rules": [
         {"file_name": ".*\\.html$"},
         {"contains": "<script [^>]*type=\"text\\/x-handlebars\"[^>]*>"}
     ]
```

In this case, there is no `match` key, so only one rule needs to match:

```js
    {
        "name": "Ruby/Ruby",
        "extensions": ["thor", "rake", "simplecov", "jbuilder", "rb", "podspec", "rabl"],
        "rules": [
            {"file_name": ".*(\\\\|/)Gemfile$"},
            {"file_name": ".*(\\\\|/)Capfile$"},
            {"file_name": ".*(\\\\|/)Guardfile$"},
            {"file_name": ".*(\\\\|/)[Rr]akefile$"},
            {"file_name": ".*(\\\\|/)Berksfile$"},
            {"file_name": ".*(\\\\|/)[Cc]heffile$"},
            {"file_name": ".*(\\\\|/)Thorfile$"},
            {"file_name": ".*(\\\\|/)Podfile$"},
            {"file_name": ".*(\\\\|/)config.ru$"},
            {"file_name": ".*\\\\Vagrantfile(\\\\..*)?$"},
            {"file_name": ".*/Vagrantfile(/..*)?$"},
            {"file_name": ".*\\.thor$"},
            {"file_name": ".*\\.rake$"},
            {"file_name": ".*\\.simplecov$"},
            {"file_name": ".*\\.jbuilder$"},
            {"file_name": ".*\\.rb$"},
            {"file_name": ".*\\.podspec$"},
            {"file_name": ".*\\.rabl$"},
            {"binary": "ruby"}
        ]
    },
```

## Rules
`rules` is an array of rules that can be used to target specific files with your defined syntax file.  The rules are processed until the first rule matches, so order your rules in a way that makes sense to you.

### Filename Rule
Filename rule defines a filename with regex.  The pattern is matched against the beginning of the full file path (there is an implicit `^`).

```js
{"file_name": ".*\\.xml(\\.dist)?$"},
```

### First Line Rule
First line rule allows you to check if the first line of the files content matches a given regex.

```js
{"first_line": "^<\\?xml"},
```

### Binary (Shebang)
A `binary` rule does the same thing as a `first_line` rule that uses a regex to match a shebang.  The difference being that ApplySyntax will construct the regex for you.

So a `first_line` rule:

```js
{"first_line": "^#\\!(?:.+)ruby"}
```

Can be simplified as:

```js
{"binary": "ruby"}
```

### Function Rule
This is an example of using a custom function to decide whether or not to apply a syntax. The source file should be in a plugin folder. `name` is the function name and `source` is the file in which the function is contained; you must include the package it resides in, all sub-folders leading to the file, and the actual file name (extension not needed as it is assumed to be a python file).

When this function is called, the full file path of the given file will be passed to it as the only argument. You are free to do whatever you want in your function, just return `True` or `False` to indicate whether a match was made.  But please be conscious of keeping it quick and light if possible.

```js
{"function": {"name": "is_rails_file", "source": "ApplySyntax/is_rails_file"}}
```

!!! tip "Tip"
    When placing a function rule module in a package, it is advised to put it in a sub-folder.  The sub-folder does not need an `__init__.py`, it just needs your module(s).

### Content Rule
Sometimes a filename or first line search is just not enough and maybe a function rule is overkill.  In this case, maybe searching the content of a file can be enough.  You can search a file's content with regex for a specific token via the `contains` rule.

```js
{"contains": "<script [^>]*type=\"text\\/x-handlebars\"[^>]*>"}
```

!!! tip "Tip"
    It is recommended to pair `contains` rules with other rules via the `#!js "match": "all"` option to ensure you don't search every file (which can significantly slow down the editor); this will also help ensure get more reliable matches. If pairing with other rules as dependencies, it is advised to pair the `contains` rule after the other required rule(s) to ensure you search the content of as few files as possible.

    Also, try to use very specific regex to ensure you don't get false positives.

## Project Specific Rules
To define project specific syntaxes, just add `project_syntaxes` to your project file.  `project_syntaxes` is an array; just add your syntax rules to `project_syntaxes` just like you would add them to `syntaxes` in your user settings file, and ApplySyntax will append the rules to the end of your globally defined rules.

There is one difference between project specific rules and global rules.  In project rules, the `extensions` key will be ignored, as the extension feature adds extensions globally, and project specific rules are meant to be confined to the project scope.

```js
    "project_syntaxes": [
        {
            "name": "XML/XML",
            "rules": [
                {"file_name": ".*\\.xml(\\.dist)?$"},
                {"first_line": "^<\\?xml"}
            ]
        }
    ]
```

## Settings Options
There are a couple of general settings found in `ApplySyntax.sublime-settings.

### Re-Raise Exceptions
If an exception occurs when processing a function, this will re-raised the captured exception in Sublime's console so the user get feedback. This is really only useful to those writing functions. The average user shouldn't need this.  By default, the setting will be set to `false`.

```js
    "reraise_exceptions": false,
```

### New File Syntax
If you want to have a syntax applied when new files are created, set `new_file_syntax` to the name of the syntax to use. The format is exactly the same as the [`name`](#name) parameter in the syntax rules mentioned earlier. For example, if you want to have a new file use JavaScript syntax, set `new_file_syntax` to `JavaScript/JavaScript`.  The default is `false`.

```js
    "new_file_syntax": "JavaScript/JavaScript",
```

### Add Extensions to Language Settings
To enable adding defined extensions to language settings, just set `add_exts_to_lang_settings` to `true`.  See [Extensions](#extensions) for more info.

```js
    "add_exts_to_lang_settings": true,
```
