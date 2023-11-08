# Console Package

## Introduction

The Console Package provides a straightforward solution for defining and executing console commands within `phpkg` applications.

## Installation

Install this package effortlessly by executing the following command:

```shell
phpkg add https://github.com/php-repos/console.git
```

## Usage

By default, Console looks for the `Source/Commands` directory in your project. Command files should be named with the suffix `Command.php` (e.g., `MakeCommand.php`) to be recognized.

### Defining Commands

In your command files, define commands by returning a closure. For instance, the following code in a `MakeCommand.php` file creates a `make` command in your application:

```php
<?php

return function () {
    // Specify actions for the 'make' command
};

```

With this code, when a user runs `php console` in your project root, they will see the following output:

```shell
Usage: console [-h | --help]
               <command> [<options>] [<args>]

Available commands:
    make
```

#### Command Description

To enhance the information presented when users interact with your command, you can provide a description by adding a docblock to the defined closure:

```php
<?php

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function () {
    // Specify actions for the 'make' command
};

```

With this code, when a user runs the `php console` command, they will encounter the following output:

```shell
Usage: console [-h | --help]
               <command> [<options>] [<args>]

Here you can see a list of available commands:
    make    This is the description for the command when the user runs help.
```

The first line of the description is visible when users execute `php console`. However, for a more detailed overview, you can define a multiline description that becomes visible when users request help for your command.

For example, using the same code as before, when a user runs `php console -h make`, they will see the following output:

```shell
Usage: console make

Description:
 This is the description for the command when the user runs help.
 You can also define a multiline description that will be shown when the user runs console with -h.

Arguments:
This command does not accept any arguments.

Options:
This command does not accept any options.
```

#### Arguments

If your command requires an argument, you can specify it using the `Argument` attribute. For instance, if your `make` command needs an email, you can define it as follows:

```php
<?php

use PhpRepos\Console\Attributes\Argument;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    string $email,
) {
    // Specify actions for the 'make' command
};

```

In the above definition, the `email` argument is mandatory, and the console ensures that users provide it. When users attempt to run the `make` command without supplying the `email` argument, they will encounter the following output:

```shell
$ php console make
Error: Argument `email` is required.
Usage: console make <email>

Description:
 This is the description for the command when the user runs help.
 You can also define a multiline description that will be shown when the user runs console with -h.

Arguments:
  <email>

Options:
This command does not accept any options.
```

However, it might not be clear why the `make` command requires an email. To address this, let's add a description for the `email` argument:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    string $email,
) {
    // Specify actions for the 'make' command
};

```

Now, if users fail to provide the `email`, they will see the following output:

```shell
$ php console make
Error: Argument `email` is required.
Usage: console make <email>

Description:
 This is the description for the command when the user runs help.
 You can also define a multiline description that will be shown when the user runs console with -h.

Arguments:
  <email> The email will be the new user email

Options:
This command does not accept any options.
```

Remember to define the type for your argument, which can be `bool`, `int`, `float`, `string`, or `array` when defining arguments.

Here's an example of a command with an array argument:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;

return function (
    #[Argument, Description('List of IDs to pass as arguments.')]
    array $ids,
);

```

Assuming an `update` command utilizes this code, users can pass IDs like so:

```shell
console update 1,2,3,4,5
```

To define an optional argument, simply make the type of the argument nullable:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    ?string $email,
) {
    // Specify actions for the 'make' command
};

```

You can also set a default value when defining an argument:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    ?string $email = 'admin@example.com',
) {
    // Specify actions for the 'make' command
};

```

Feel free to define as many arguments as needed. The order of defined arguments matters, so consider the following example:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    string $email,
    #[Argument]
    #[Description('The password for the user account')]
    string $password,
) {
    // Specify actions for the 'make' command
};

```

In this case, when users pass two arguments to the command, the first one will be the email, and the second one will be the password:

```shell
php console admin@example.com password123
```

#### Long Options

In certain commands, you may need to define options, and there are two types: long and short. We'll discuss short options later in this document.

To define a long option in your command, utilize the `LongOption` attribute for your command property. The following code illustrates defining a `--user` option for the `make` command:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;
use PhpRepos\Console\Attributes\LongOption;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    string $email,
    #[Argument]
    #[Description('The password for the user account')]
    string $password,
    #[LongOption('user')]
    string $username,
) {
    // Specify actions for the 'make' command
};

```

With this code, users will see the following output when they seek help for your command:

```shell
$ php console -h make
Usage: console make [<options>] <email>

Description:
 This is the description for the command when the user runs help.
 You can also define a multiline description that will be shown when the user runs console with -h.

Arguments:
  <email>    The email will be the new user email
  <password> The password for the user account

Options:
  --user <username>
```

To provide clarity, you can define a description for your long option using the `Description` attribute:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;
use PhpRepos\Console\Attributes\LongOption;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    string $email,
    #[Argument]
    #[Description('The password for the user account')]
    string $password,
    #[LongOption('user')]
    #[Description("This will be the new user's username")]
    string $username,
) {
    // Specify actions for the 'make' command
};

```

The added description will be displayed when users run the help command or when they omit required arguments or options:

```shell
$ php console -h make
Usage: console make [<options>] <email>

Description:
 This is the description for the command when the user runs help.
 You can also define a multiline description that will be shown when the user runs console with -h.

Arguments:
  <email>    The email will be the new user email
  <password> The password for the user account

Options:
  --user <username> This will be the new user's username
```

When defining long options, you must specify the parameter type. You can use `bool`, `int`, `float`, or `array`. Except for array long options, other long options can be passed with or without the `=` sign:

```shell
$ php console make john@example.com --user JohnDoe
$ php console make john@example.com --user=JohnDoe
```

For array long options, users must use the equal sign for each item separately:

```shell
$ php console update --ids=1 --ids=2 --ids=3
```

You can make the option optional by defining the parameter type as nullable:

```php
<?php

use PhpRepos\Console\Attributes\LongOption;

return function (
    #[LongOption('team')]
    ?string $team
) {

};

```

You can also set default values for your optional long options during their definition:

```php
<?php

use PhpRepos\Console\Attributes\LongOption;

return function (
    #[LongOption('team')]
    ?string $team = 'dev'
) {

};

```

#### Short Options

Similar to long options, you can also define short options using the `ShortOption` attribute in your command.

The following code adds a `f` short option to the `make` command, allowing users to force the creation of a new user record:

```php
<?php

use PhpRepos\Console\Attributes\Argument;
use PhpRepos\Console\Attributes\Description;
use PhpRepos\Console\Attributes\LongOption;
use PhpRepos\Console\Attributes\ShortOption;

/**
 * This is the description for the command when the user runs help.
 * You can also define a multiline description that will be shown when the user runs console with -h.
 */
return function (
    #[Argument]
    #[Description('The email will be the new user email')]
    string $email,
    #[Argument]
    #[Description('The password for the user account')]
    string $password,
    #[LongOption('user')]
    #[Description("This will be the new user's username")]
    string $username,
    #[ShortOption('f')]
    #[Description('Whether or not force creating the user')]
    bool $force
) {
    // Specify actions for the 'make' command
};

```

When users run the help command, they will see the following output:

```shell
Usage: console make [<options>] <email> <password>

Description:
 This is the description for the command when the user runs help.
 You can also define a multiline description that will be shown when the user runs console with -h.

Arguments:
  <email>    The email will be the new user email
  <password> The password for the user account

Options:
  --user <username> This will be the new user's username
  -f                Whether or not to force creating the user
```

As observed, a description for the short option is already defined.

Type definition is crucial for short options as well. When you define a short option with `bool` type, it will be considered as `true` when present and `false` when absent.

```shell
php console command -f // f is true
php console command // f is false
```

For short options defined with other types like `int`, `float`, or `string`, the equal sign is optional:

```shell
php console command -p=password
php console command -p password
```

When defining a short option as `array`, users must use `=` for each item:

```shell
php console command -i=1 -i=2 -i=3
```

Of course, you can define short options as optional or set default values:

```php
<?php

use PhpRepos\Console\Attributes\Description;
use PhpRepos\Console\Attributes\ShortOption;

return function (
    #[ShortOption('t'), Description('Optional short option')]
    ?string $team,
    #[ShortOption('p'), Description('Optional with default value short option')]
    ?string $password = 'password',
) {

};

```

### Running Commands

It's crucial to remember that you need to `build` your application before running any commands. For more information, refer to the [build documentation](https://phpkg.com/documentations/build-command) on `phpkg`.

Once your application is built and you are in the build directory, you can run commands using either of the following methods:

```shell
php console your-command
./console your-command
```

Appending a short option `h` after `console` and before any command name will display the help:

```shell
php console -h // Shows the default help containing a list of commands and their descriptions

php console -h command-name // Displays detailed help for the command, including all defined arguments and options
```

### Advanced Customization

When utilizing the default `console` file for running your commands, Console searches for files ending with `Command.php` in the `Source/Commands` directory. However, you can customize this behavior.

Add a file to your project's root, for example, consider having a `cli` PHP file in your root. You can then modify the default settings:

```php
#!/usr/bin/env php
<?php

use PhpRepos\Console\Config;
use PhpRepos\FileManager\Path;
use PhpRepos\FileManager\Resolver;

$custom_console_config = new Config(
    commands_directory: Path::from_string(__DIR__ )->append('Src/MyCommands'),
    commands_file_suffix: '.php',
);

require Resolver\realpath(__DIR__ . '/Packages/php-repos/console/console');

```

With this code in your `cli` file, your commands will be loaded from the `Src/MyCommands` directory, and any `.php` file in that directory will be considered as a command.

> **Note**
> Ensure you add `cli` in the `entry-points` section for your application. See [Customization](https://phpkg.com/documentations/customization) on `phpkg`.
