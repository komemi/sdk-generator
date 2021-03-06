# Contributing

We would ❤️ for you to contribute to Appwrite and help make it better! As a contributor, here are the guidelines we would like you to follow:

## Code of Conduct

Help us keep Appwrite open and inclusive. Please read and follow our [Code of Conduct](/CODE_OF_CONDUCT.md).

## Installation

To install a working development environment follow this instruction:

1. Fork or clone the appwrite/sdk-generator repository.

2. Install Composer dependencies:

**CLI**
```bash
composer update --ignore-platform-reqs --optimize-autoloader
```

**Docker (UNIX)**

```bash
docker run --rm --interactive --tty --volume "$(pwd)":/app composer install
```

**Docker (Windows)**

```bash
docker run --rm --interactive --tty --volume "%cd%":/app composer install
```

1. Follow our contribution guide to learn how you can add support for more languages.

## Creating Language Class

First, create a new class for the new language in this directory: https://github.com/appwrite/sdk-generator/tree/master/src/SDK/Language

You can use the interface to know which methods are required to be implemented:
https://github.com/appwrite/sdk-generator/blob/master/src/SDK/Language.php

**getName**
SDK Language name (JS, PHP…)

**getKeywords**
An array with language keywords to avoid using as param or function names, template engine will solve conflicts

**getFiles**
An array with a list of language template files in [twig format](https://twig.symfony.com/). 
Each file scope determines what template parameters will be available.

* default scope - Basic SDK and language-specific params (package name, language name, etc…)
* service scope - Generate x templates where x is the number of API services, adds service-specific params to the template (service name, methods, etc…)
* method scope - Generate x*y templates where x is the number of API services and y is the number of methods, adds service and method-specific params to the template (service name, method name, method params, etc…), good for generating MD files with examples for using each method

**getTypeName**
This method receives the API param type and should return the equivalent param in the implemented language.

**getParamDefault**
This method receives the API param and should return the equivalent default value of param in the implemented language, for example, a default array param in PHP is represented as [].

**getParamExample**
This method receives the API param and should return the equivalent example value of param in the implemented language, for example, if an example value is **some text** in PHP return value should be **'some text'** (with quotes).

Notice: The easiest way to get started is to copy an existing language class close to the new language about to be implemented and just edit it.

## Adding Templates

Add your new templates as listed in your language class **getFiles** method. Make sure to follow the [checklist](https://github.com/appwrite/sdk-generator#sdk-checklist) when building the language templates.

Make sure to follow the objects structure and service separation architecture. We aim to keep developer experience as consisted as possible across different SDKs to make the learning curve as small as possible.

> Appwrite SDK generator adds some filters to the TWIG templates to allow common code formatting options like converting text to camelCase, dash-case and other. Full list is available in the [SDK class](https://github.com/appwrite/sdk-generator/blob/master/src/SDK/SDK.php#L62)

When in need to test the API templates output, add your new language instance to the `example.php` file like this:

sdk-generator/blob/master/example.php:

```php
    use Appwrite\Spec\Swagger2;
    use Appwrite\SDK\SDK;
    use Appwrite\SDK\Language\NewLang;
    
    function getSSLPage($url) {
        $ch = curl_init();
        curl_setopt($ch, CURLOPT_HEADER, false);
        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        $result = curl_exec($ch);
        curl_close($ch);
        return $result;
    }

    $spec = getSSLPage('https://appwrite.io/v1/open-api-2.json?extensions=1');

    // NewLang
    $sdk  = new SDK(new NewLang(), new Swagger2($spec));
    $sdk
        ->setLogo('https://appwrite.io/v1/images/console.png')
        ->setLicenseContent('test test test')
        ->setWarning('**WORK IN PROGRESS - NOT READY FOR USAGE**')
    ;
    $sdk->generate(__DIR__ . '/examples/new-lang');
```

Run the following command (make sure you have an updated docker version on your machine):

```bash
docker run --rm -v $(pwd):/app -w /app php:7.4-cli php example.php
```

>Note: You can just add the new language next to the other languages in example.php file, no need to rewrite the file completely.

Check your output files at: /examples/new-lang and make sure the SDK works. When possible add some unit tests.

## SDK Checklist

It is very important for us to create consistent structure, architecture and native like feel for the SDKs we generate.
In order to accomplish that we have made a checklist of points to support while adding a new language to the SDK generator.

The checklist aims to balance consistency among languages, and follow each platform's best practices and coding standards.

* Proper Coding Standards and Conventions
* Proper Skeleton Structure
* Readme Doc
* HTTP Client class or object
    * Client Setters
        * Set Auth Keys Method
        * Set Basic Auth Method
        * Set OAuth Dialog Method
        * Set Endpoint Method
        * Set Self Signed Certificates
    * Default Headers
        * 'appwrite-sdk-version' header
        * Add 'User-Agent' header with server name and language version (ubuntu-18.02:php-7.0.1)
    * Methods
        * addHeader(key, value)
        * call(method, path = '', headers = [], params = [])
            * Concat GET params to path
            * Parse request params by content type header
            * Parse response params by content type header
            * Throw error on bad response
* Service Abstraction (optional)
    * Constructor receiving an instance of the client class 
* Service Class (extends the service abstraction if exists)
    * Headers Support (Content Type)
    * Parameters Support
        * Default Values Support
        * Required Values Support
        * String Support
        * Integer Support
        * Boolean Support
        * Files Support (+array file and multiple header support and params flatten)
        * Arrays / Dictionaries / Lists Support (+concatenation type)
* Usage Example Docs
* Definitions / Models Classes - with setters and getters

## Tests

Testing a single project that runs in multiple languages can be very hard. Managing dependencies with multiple package managers of different ecosystems can take the SDK Generator complexity to extreme levels.

To avoid that complexity, we have created a cross-platform mechanism that leverages Docker and a vanilla language file with no dependencies attached.

To add a new language test, you need to create a language file that initializes your SDK and call generated method in a predefined order. The test algorithm will evaluate your script output and determine whether the test passed or failed.

The test algorithm will generate your SDK from a small demo SDK JSON spec file and will run your test on different versions of your chosen language.

To get started, create a language file in this location:

`./tests/languages/tests-for-[MY-LANGUAGE].[MY-LANGUAGE-FILE-EXT]`

In your new language file, init your SDK from a relative path which will be generated here: `./tests/sdks/` from this spec file: `./tests/resources/spec.json`.

After you finish initializing, make a series of HTTP calls using your new generated SDKs method just like in one of these examples:

1. tests/languages/tests-for-php.js
2. tests/languages/tests-for-node.js

Once done, add a Docker command that can execute your test file to the SDK test algorithm `$containers` array in this location: `./tests/SDKTest.php:17`. Make sure to add one command for each language version you wish to support.

A good example is the PHP test for 5 different PHP versions:

```php
protected $containers = [
    'php-5.6' => 'docker run --rm -v $(pwd):/app -w /app php:5.6-cli php tests/languages/tests-for-php.php',
    'php-7.0' => 'docker run --rm -v $(pwd):/app -w /app php:7.0-cli php tests/languages/tests-for-php.php',
    'php-7.1' => 'docker run --rm -v $(pwd):/app -w /app php:7.1-cli php tests/languages/tests-for-php.php',
    'php-7.2' => 'docker run --rm -v $(pwd):/app -w /app php:7.2-cli php tests/languages/tests-for-php.php',
    'php-7.3' => 'docker run --rm -v $(pwd):/app -w /app php:7.3-cli php tests/languages/tests-for-php.php',
    'php-7.4' => 'docker run --rm -v $(pwd):/app -w /app php:7.4-cli php tests/languages/tests-for-php.php',
];
```
