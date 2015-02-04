# MULTI LANGUAGE

[![Build Status](https://travis-ci.org/micheleangioni/multi-language.svg)](https://travis-ci.org/micheleangioni/multi-language)
[![License](https://poser.pugx.org/michele-angioni/multi-language/license.svg)](https://packagist.org/packages/michele-angioni/multi-language)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/576a839e-7c84-4615-894a-465f30c8f881/small.png)](https://insight.sensiolabs.com/projects/576a839e-7c84-4615-894a-465f30c8f881)

## Introduction

Multi Language is a [Laravel](http://laravel.com) package which handles the localization. It acts as a wrapper for the laravel localization package and lets ease translation of your default lang files into new languages.

**WARNING** : The Laravel 4 version is not maintained anymore.

## Installation

Multi Language can be installed through Composer, just include `"michele-angioni/multi-language": "dev-master"` to your composer.json.

## Configuration

Add the following service providers under the providers array in your app.php configuration file

    'MicheleAngioni\MultiLanguage\MultiLanguageServiceProvider',
    'MicheleAngioni\MultiLanguage\MultiLanguageBindServiceProvider'

Multi Language can be highly customized by publishing the configuration file through the artisan command `php artisan config:publish michele-angioni/multi-language`. 
You can than edit the config.php file in your `app/config/packages/michele-angioni/multi-language` directory to customize the support behaviour:

- allowed_languages : it is the number of allowed languages. It can be used to prevent the creation of new supported languages
- allowed_nested_arrays : maximum number of nested arrays allowed in lang files
- language_files_path : path to the language files
- max_text_length : max text length allowed for the languages keys
- safe_mode : enables / disables safe mode. If safe mode is enabled, only files with the same name of files of default locale can be created. Furthermore, only keys already present in the default locale can be added to new languages.

## Usage

The `MicheleAngioni\MultiLanguage\LanguageManager` class is the class that accesses all Multi Language features.
By default it will uses the [Laravel file system manager](http://laravel.com/api/4.2/Illuminate/Filesystem/Filesystem.html) and the [Laravel localization feature](http://laravel.com/docs/4.2/localization).

You can inject it in the constructor of the one of your classes or directly instance it by using the Laravel Application facade `App::make('MicheleAngioni\MultiLanguage\LanguageManager')` and use its methods:

- getLanguagesPath() : get current path to languages files
- setLanguagesPath($path) : set the path to languages files
- setMaxTextLength($value) : set dynamically the max allowed length text
- getDefaultLocale() : get the default locale language. It will be used when a language key is not found in the selected language
- getLocale() : get the current used locale language
- setLocale($locale) : set the current used locale language
- setSafeMode($bool) : change dynamically the Safe Mode setting
- getAvailableLanguages() : get a list of the available languages, that is. the languages which have a directory under the lang directory
- getLanguageFiles($useDefaultLocale = false) : get a list of language files of input language
- getLanguageFile($fileName, $useDefaultLocale = false) : return all keys contained in the input file (without extension)
- getLanguageFileKey($fileName, $key) : return the value of input key in the set locale input file (without extension)
- createNewLanguage($newLocale) : create a new directory under the languages directory for input locale
- writeLanguageFile($fileName, array $inputs) : write the input array on a php file under the /lang/locale directory with input name
- buildArray(array $inputs) : take an array with dot notation and build the final array that will be written into the file
- convertArrayToDotNotation(array $array) : convert an array containing nested arrays to an array with dot notation

## (optional) Custom File System and Translator

By default the Language Manager uses the [Laravel file system manager](http://laravel.com/api/4.2/Illuminate/Filesystem/Filesystem.html) and the [Laravel localization feature](http://laravel.com/docs/4.2/localization).
You can override that by defining your own file system (which has to implement the `MicheleAngioni\MultiLanguage\FileSystemInterface`) and translator (which has to implement the `MicheleAngioni\MultiLanguage\TranslatorInterface`)
The two new files can be injected in the Language Manager constructor by commenint the 'MicheleAngioni\MultiLanguage\LanguageManagerBindServiceProvider' line in the app.php conf file and defining your custom binding in a new service provider.

## Example

Suppose we have a `users.php` file under the app/lang/en directory

    /app
    ├--/controllers
    ├--/lang
    |     └--/en
    |          └--users.php

which contains

    <?php

    return array(

        "password" => "Passwords must be at least six characters and match the confirmation.",

        "user" => "We can't find a user with that e-mail address.",

        "token" => "This password reset token is invalid.",

        "sent" => "Password reminder sent!",

    );

Let's suppose we want to create a Spanish version of the file. We can build a controller handling the language management

    <?php

    use MicheleAngioni\MultiLanguage\LanguageManager;

    class LanguagesController extends \BaseController {

        private $languageManager;

        function __construct(LanguageManager $languageManager)
        {
            $this->languageManager = $languageManager;
        }

    }

and write down some methods to handle the requests.

    public function index()
    {
        $languages = $this->languageManager->getAvailableLanguages(); // ['en']

        return View::make('languages')->with('languages', $languages)
    }

The above $languages variable would just be a single value array `['en']` since we only have the /en directory under /lang.

We now need a list of the English files:

    public function showFiles()
    {
        $files = $this->languageManager->getLanguageFiles(); // ['users']

        return View::make('languagesFiles')->with('files', $files)
    }

The showFiles() method would just return `['users']` as we have just one file in our /lang/en directory.

Let's examine the content of the file

    // $fileName = 'users';
    public function showFile($fileName)
    {
        $file = $this->languageManager->getLanguageFile($fileName)

        return View::make('languagesFile')->with('file', $file)
    }

The above method returns an array with the file content.

Let's now create a Spanish version. First of all we must create the /es directory under the /lang directory

    public function createNewLanguage()
    {
        // [...] Input validation

        $this->languageManager->createNewLanguage($input['locale']);

        return Redirect::route('[...]')->with('ok', 'New language successfully created.');
    }

We then obviously need a view to submit the Spanish sentences and we leave it up to you.
An associative array with key => sentence structure must be sent from the view to the following method

    public function saveFile($locale, $fileName)
    {
        // [...] Input validation

        $this->languageManager->setLocale($languageCode);

        $this->languageManager->writeLanguageFile($fileName, $input['all']);

        return Redirect::route('[...]')->with('ok', 'Lang file successfully saved.');
    }

## License

Support is free software distributed under the terms of the MIT license.

## Contacts

Feel free to contact me.
