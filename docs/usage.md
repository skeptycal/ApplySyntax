# User Guide {: .doctitle}
Configuring and using ApplySyntax.
{: .doctitle-info}

---

# Overview
ApplySyntax is based on the idea of creating rules for selecting a certain syntax. You define the rules, the plugin checks them. The first one to pass wins.

Create your own custom rules in `Packages/User/ApplySyntax.sublime-settings`. The easiest way to get started is to just copy the default settings file found in `Packages/ApplySyntax/ApplySyntax.sublime-settings` to your `Packages/User` directory and modify it to meet your needs. Make sure you rename the `default_syntaxes` key to just syntaxes. If you don't, you will overwrite the default syntaxes and they will not work.

See the default settings file for examples.

# Creating Rules

Each rule is a dictionary within the syntax array.  Let's take a look at the top level parameters.

## Name
`name` is the syntax file that will be applied to a view which meets the criteria defined in the rule.

For syntax files you must specify the path to the syntax file. The plugin is capable of supporting multiple levels of nesting if you need it to. For example, if you had all of your tmLanguage files for Rails organized like this: `Packages/Rails/Language/*.tmLanguage`, and you were looking to use the `Ruby Haml.tmLanguage` file, you would define the syntax to be:

```js
"name": "Rails/Language/Ruby Haml"
```

If it is desirable for the syntax rule to reference multiple tmLanguage files because it is not known which package will be on a machine, you can set the syntax as an array of names like:

```js
"name": ["RSpec", "RSpec (snippets and syntax)/Syntaxes/RSpec"]
```

## Extensions
`extensions` is a convenience option to add a given set of extensions to your language settings file.  By adding the extension to the language settings file, sidebar icons in ST3 will display the proper icon and files will load with the proper syntax via Sublime's default extension detection method.  Keep in mind though that other rules can override this.  As `extensions` isn't really a rule, but just a list that automatically adds the extension to the language settings file, a hit on this array won't currently stop ApplySyntax from processing more rules (this may change in the future).

Apply syntax will create a file `ApplySyntax.ext-list` in your `User` folder and track which extension it added so that if you remove a rule, ApplySyntax will only remove the extensions it added to the language file in question.

If you do not like this functionality, you can simply disable it by setting the following option in your settings file to `false`:

```js
    "add_exts_to_lang_settings": true,
```

## Match
`match` is a setting that you either include or do not.  When included, you set it to `all`.  When set, all rules defined must be met for a match to be considered successful.

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

In this case, any of the rules can match:

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
`rules` is an array of rules that can be used to target specific files with your defined syntax file.  The rules are processed until the first True result, so order your rules in a way that makes sense to you.

### Filename Rule
Filename rule defines a filename with regex.

```js
{"file_name": ".*\\.xml(\\.dist)?$"},
```

### First Line Rule
First line rule allows you to check if the first line matches a given regex.

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
This is an example of using a custom function to decide whether or not to apply this syntax. The source file should be in a plugin folder. `name` is the function name and `source` is the file in which the function is contained; you must include the package it resides in, all sub-folders leading to the file, and the actual file name (extension not needed).

When this function is called, the filename will be passed to it as the only argument. You are free to do whatever you want in your function, just return `True` or `False`.  But please be conscious of keeping it quick and light if possible.

```js
{"function": {"name": "is_rails_file", "source": "ApplySyntax/is_rails_file"}}
```

### Content Rule
Sometimes a filename or first line search is just not enough and maybe a function rule is overkill.  In this case, maybe searching content of a file can be enough.  You can search a file's content with regex for a specific token via the `contains` rule.

```js
{"contains": "<script [^>]*type=\"text\\/x-handlebars\"[^>]*>"}
```

!!! tip "Tip"
    It is recommended to pair `contains` rules with other rules via the `#!js "match": "all"` option to ensure you don't search every file (which can significantly slow down the editor), and you get good match results. If pairing with other rules, it is advised to pair it after the other required rule(s) to ensure you search the content of as few files as possible.

    Also, try to use very specific regex to ensure you don't get false positives.

### Project Specific Rules
To define project specific syntaxes just add `project_syntaxes` to your project file.  `project_syntaxes` is an array; just add your syntax rules to it, and ApplySyntax will append the rules to the end of your globally defined rules.

## Settings Options
There are a couple of general settings found in `ApplySyntax.sublime-settings.

### Re-Raise Exceptions
If an exception occurs when processing a function, should it be re-raised so the user gets feedback? This is really only useful to those writing functions. The average user just wants the plugin to work.  By default, this will be set to `false`.

```js
    "reraise_exceptions": false,
```

### New File Syntax
If you want to have a syntax applied when new files are created, set `new_file_syntax` to the name of the syntax to use. The format is exactly the same as the `name` parameter in the syntax rules mentioned earlier. For example, if you want to have a new file use JavaScript syntax, set `new_file_syntax` to `JavaScript/JavaScript`.  The default is `false`.

```js
    "new_file_syntax": "JavaScript/JavaScript",
```

### Add Extensions to Language Settings
To enable adding defined extensions to language settings, just set `add_exts_to_lang_settings` to `true`.  See [Extensions](#extensions) for more info.

```js
    "add_exts_to_lang_settings": true,
```
