# Locale for Javascript

# Features
---
* Multiple target languages (translate to)
* Any base language (translate from)
* Plural forms for any language
* Simultaneous support of multiple languages
* Flexible placeholders
* Deferred translation

## It's simple
---
```javascript
// Translate
var who = 'world';
__('Hello, %s!', who);

// Translate plural
__([
  'There is %n item in your cart (total $%s).',
  'There are %n items in your cart (total $%s).'
  ], cart.items.length, cart.total
);
```

# API
---
- [Library import](#library-import)
- [Configuration](#configuration)
- [Creating i18n object](#creating-i18n-object)
- [Translation target](#translation-target)
- [Translation](#translation)
- [Strings format](#strings-format)
- [Deferred translation](#deferred-translation)

## Library import
---

### For node.js:
```javascript
var locale = require('locale-node-js');
```

## Configuration
---

### Library initialization (browser):
```javascript
locale.init(base, rule)
```
, where:

- *base* (string) - id of application's base language;
- *rule* (string) - plural rule for base language.

You should use locale.add() to add translation for each used language.

### Library initialization (node.js):
```javascript
locale.init(path)
```
, where *path* is the path to the translation files.

This function searches for and loads all translations automatically.

### Example:
```javascript
// In node.js app:
locale.init('./i18n/');
```
### Add translation:
```javascript
locale.add(lang, translation)
```
, where:

- *lang* (string) - language id;
- *translation* (object) - translation JSON.

## Creating i18n object
---

### Create translation object for the given target:
```javascript
new locale.i18n(target)
```
, where *target* (string or null) - 1) id of the target language, 2) empty string or 3) null.

Sets target language to base language if *target* is *''* (empty string).  
Enables deferred translation mode if *target* is *null*.  

### Example:
```javascript
// Create translation object for Russian language
var i18n = new locale.i18n('ru');
// Create translation object for base language (no translation)
var i18n = new locale.i18n('');
// Create translation object for deferred translation
var i18n = new locale.i18n(null);
// And pull up __() function to the local scope
var __ = i18n.__;
```
## Translation target
---

This methods are accessible for *locale* and *i18n* objects.

### Get current target:
```javascript
to()
```
Returns (string or null) current translation target.

### Set target:
```javascript
to(target)
```
, where *target* (string or null) - 1) id of target language, 2) empty string or 3) null.

Sets target language to base language if *target* is *''* (empty string).  
Switches i18n object to deferred translation mode if *target* is *null*.  

### Example:
```javascript
// Set target language for library to German
locale.to('de');
// Create i18n object and set its target language to Russian
var i18n = new locale.i18n();
i18n.to('ru');
// Enable deferred translation mode
i18n.to(null);
```
## Translation
---

This methods are accessible for *locale* and *i18n* objects.

### For simple form:
```javascript
__(phrase)
__(phrase, ...)
__(phrase, array)
__(phrase, object)
```
, where:

- *phrase* (string) - string for translation (can contains placeholders);
- *...* (mixed) - any number of additional args;
- *array* (Array) - array of args;
- *object* (object) - map of args.

Returns translated string or deferred translation data (in case of deferred translation target).

### For plural form:
```javascript
__(phrase, n)
__(phrase, n, ...)
__(phrase, n, array)
__(phrase, n, object)
```
, where:

- *phrase* (array) - array of strings each of them is a plural form for translation (can contains placeholders);
- *n* (numeric) - base number of plural form;
- *...* (mixed) - any number of additional args;
- *array* (Array) - array of args;
- *object* (object) - map of args.

Returns translated string or deferred translation data (in case of deferred translation target).

### Example:
```javascript
// Simple:

// Hello!
__('Hello!');
// Hello, anonymous!
__('Hello, %s!', 'anonymous');
// Hello, John Smith!
__('Hello, %(1)s %(0)s!', [ 'Smith', 'John' ]);
// Hello, Foo (bar)!
__('Hello, %(name)s (%(login)s)!', { name: 'Foo', login: 'bar' });

// Plural:

// Inbox: 1 unreaded message.
__([ 'Inbox: %n unreaded message.', 'Inbox: %n unreaded messages.' ], 1);
// Inbox: 7 unreaded messages.
__([ '%(0)s: %n unreaded message.', '%(0)s: %n unreaded messages.' ], 7, 'Inbox');
// Anonymous, you have 365 unreaded messages in the "Spam" folder.
__([
    '%(0)s, you have %n unreaded message in the "%(1)s" folder.',
    '%(0)s, you have %n unreaded messages in the "%(1)s" folder.'
   ], 365, [ 'Anonymous', 'Spam' ]);
// The second way
__([
    '%(login)s, you have %n unreaded message in the "%(folder)s" folder.',
    '%(login)s, you have %n unreaded messages in the "%(folder)s" folder.'
   ], 365, { login: 'Anonymous', folder: 'Spam' });

// Deferred:

// { __i18n: true, phrase: 'Hello!', n: undefined, args: {} }
__('Hello!');
```
See also: [Deferred translation](#deferred-translation).

## Strings format
---

### Phrase, id and comment

Each string consists of *phrase*, *id* and *comment*.
Both *id* and *comment* are optional, *id* separates from *phrase* by '#' sign, *comment* separates from *id* by space.
```
<phrase>[#[<id>][ <comment>]]
```
*phrase* + *id* is unique key for translation dictionary. You can create several variants of translation of the same *phrase* using different *id*s.

Also you can place any useful info for translator to *comment*.

### Example:
```javascript
// No id, no comment
__('Hello!');
// Both id and comment
__('Hello!#another This is a comment for translator');
// Only id
__('Hello!#another');
// Only comment
__('Hello!# This is a comment for translator');
```
### Placeholders

Phrase can contains placeholders.

Placeholder's format is:

```
%[(<name>)][<flag>][<width>][.<precision>]<type>
```

, where:

- *\<name>* - argument name or number;
- *\<flag>* - one or more special flags;
- *\<width>* - field width;
- *\<precision>* - precision for numeric or max string length for string;
- *\<type>* - conversion specifier.


#### The name

Argument number for additional arguments and array arguments, argument name for map arguments.

#### The flag characters

- *'-'* - The converted value is to be left adjusted on the field boundary (the default is right).
- *' '* - A space should be left before a positive number or empty string.
- *'+'* - A plus sign should always be placed before a non-negative number. Overrides a space if both are used.

#### The field width

An optional decimal digit string (with nonzero first digit) specifying a minimum field width. If the converted value
has fewer characters than the field width, it will be padded with spaces (on the left by default or on the rigth
if '-' flag has been given).

#### The precision

An optional precision, in the form of a period ('.') followed by an optional decimal digit string.
This gives the number of digits to appear after the radix character for numeric conversions or the
maximum number of characters for string conversions.

#### The conversion specifier

A character that specifies the type of conversion to be applied. The conversion specifiers and their meanings are:

- **s** - string;
- **d** - decimal number;
- **e** - decimal number in scientific notation;
- **b** - binary number with 'b' suffix;
- **h** - hex number with 'h' suffix and lowercased 'a-f' digits;
- **x** - hex number with '0x' prefix and lowercased 'a-f' digits;
- **X** - hex number with '0x' prefix and uppercased 'A-F' digits;
- **n** - same as 'd' but with value from n parameter;
- **%** - A '%' is written. No argument is converted. The complete conversion specification is '%%'.

### Example:

```javascript
__('%+10.2d%%', 13); // '    +13.00'
__('Hello, %(who)-5.3s!', { who: 'world'} ); // 'Hello, wor  !'
__('%(1)s is %(0)b (%(0)h)', 42, 'value'); // 'value is 1101b (Dh)'
```

See also: [Translation](#translation) example.

### Special characters

Special characters are '#' and '%'. If you need it in your string you should type it twice.
```javascript
// 40%
__('%s%%', 40);
// It's # not a comment!
__('It\'s ## not a comment!);
```

## Deferred translation
---

### Search for deferred translation data in the *obj* and replace it by translated string:
```javascript
i18n.tr(obj)
```
, where *obj* (object) - object for translation.

### Deferred translation data structure:
```
{
  __i18n: true,
  phrase: <phrase>,
  n: <number>,
  args: <arguments>
}
```
See also: [Creating i18n object](#creating-i18n-object).

### Example:
```javascript
// Server:

// Create i18n object for the deferred mode
var i18n = new locale.i18n(null);
var __ = i18n.__;
submit({
  id: 832367,
  errno: 404,
  error: __('Path not found!')
});

// Client:

var i18n = new locale.i18n('ru');
var msg = receive();
i18n.tr(msg);
/*

  {
    id: 832367,
    errno: 404,
    error: 'Путь не найден!'
  }

*/
```