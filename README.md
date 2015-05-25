# Relevant Cards Documentation

This document explains how to create cards for the [Relevant iOS app](http://relevant.ai).

A *card* is just a JSON file hosted somewhere on the web. This JSON file directs the Relevant servers on how to gather the data and display it using templates such as banner images, footers, etc.

A great way to host a card is using Dropbox.

## Intro and Hello World

Here is a sample *"Hello World"* card:

```json
{
    "id": "hello-world",
    "title": "Hello World Card",
    "icon_url": "http://relevant.ai/hello_world.png",
    "summary": "Card's summary for the library.",
    "credits": "Relevant",
    "settings_type": "NONE",
    "_LOAD": {
        "_RETURN":[
            [
                {
                    "description":{
                        "title": "Hello World",
                        "body": "Hello World? There's a card for that."
                    }
                },
                {
                    "footer":{
                        "caption": "Relevant - The Missing Home Screen"
                    }
                }
            ]
        ]
    }
}
```

The keys `id`, `title`, `icon_url`, `summary`, `credits`, and `settings-type` are metadata. Of these, only the `title` value is visible on the card itself.

The `"_LOAD"` key represents the content and appearance of the card.

**To test this card;** launch the [Relevant iOS app](http://relevant.ai), shake your device, and when prompted, insert the following URL into the input box:

`https://gist.githubusercontent.com/wircho/f250e0ae9f818637c9c5/raw/d2ebeb8306f7520f60f5f318bc721eb378dd4fe1/card`

You should see the following card:

![Hello World Sample Card](https://raw.githubusercontent.com/relevant-ai/RelevantCardsDocumentation/master/hello_world_card.png)

Think of the object `_LOAD` as a function that is called every time the card needs to refresh. The `_RETURN` of this function is an array with **only one element**. That means this card has **only one page** (swiping left and right will do nothing). This page is itself an array of **templates**: A `description` template with parameters `title` and `body`, and a `footer` template with parameter `caption`.

**Remark:** The `_RETURN` of the `_LOAD` function must always be an array of arrays. Each element of the outer array is a *page* and each element of the inner array is a *template*.

**Remark:** As of Relevant 1.0, cards **do not update automatically** when you change the code. After making changes to your hosted JSON, you need to delete the current card (tap the "i" button, then REMOVE) and then add it again by shaking the device.

## The Rel Language

Everything inside the `_LOAD` function must follow the syntax of the Rel language. Reserved words/keys for the Rel language are uppercase and preceded by an *underscore* (`_`). Everything else is what it looks like: a JSON object, array, string, or number.

### Variables

You can define variables inside the `_LOAD` function as follows:

```json
"_LOAD":{
    "foo":5,
    "var":[1,"a",{"b":"c"}],
    "norf":{"x":50.1,"y":-70},
    "_RETURN": ...
}
```

Normally, unless used somewhere in `_RETURN`, these variables will be ignored. This makes Rel *lazy*; it only does things if needed and when needed.

The easiest way to use a variable is using the syntax `"{variable_name}"`. For example, the Hello World card from before is equivalent to the one below:

```json
{
    "id": "hello-world",
    "title": "Hello World Card",
    "icon_url": "http://relevant.ai/hello_world.png",
    "summary": "Card's summary for the library.",
    "credits": "Relevant",
    "settings_type": "NONE",
    "_LOAD": {
        "hello":"Hello World",
        "foo":{
            "description":{
                "title": "{hello}",
                "body": "{hello}? There's a card for that."
            }
        },
        "var":{
            "footer":{
                "caption": "Relevant - The Missing Home Screen"
            }
        },
        "_RETURN":[
            [
                "{foo}",
                "{var}"
            ]
        ]
    }
}
```

In this card the `_LOAD` function has three intermediate variables, named `hello`, `foo`, and `var`.

### Inserting/Concatenating String Variables

As in the previous example, it is possible to insert a string-valued variable into another string using the syntax `"... {variable_name} ..."`. For example `"The value of the variable foo is {foo}"`.

This is only possible when `foo` is a string, and will not work otherwise.

### Built-In Functions

The Rel language comes packaged with a set of built-in functions to help you easily manipulate JSON objects. All built-in function names follow the *underscore uppercase* syntax (e.g. `_CONCAT`) to differentiate it from your own variables and JSON objects.

A built-in function call is usually represented by a one-key object whose only key is the function name. For example:

```json
{
    "_FUNC_NAME": ...
}
```

The value of the key is the parameter of the function. This could be a string, an array, an object, or in some cases a set of parameters with specific names, as follows:

```json
{
    "_FUNC_NAME": {
        "_FIRST_PARAM_NAME": ...,
        "_SECOND_PARAM_NAME": ...,
        "_THIRD_PARAM_NAME": ...
    }
}
```

Most of the rest of this documentation is devoted to introducing Rel's built in functions.

### The `_PATH` function

We often need to dig deep inside a JSON object or array, in order to fetch a specific value. This will be more useful when we get JSON objects following all kinds of different standards from web services (See the `_URL` function below).

The `_PATH` function takes an array of strings and integers. The first string is the name of a variable. Every subsequent string or integer represents either an object's key, or an array's index. The result of the function call is obtained by recursively looking up these keys and indices into the variable.

In the example bellow, the variable `var` takes the value `"Hello World"`

```json
"foo":{
    "byes":[
        {"value":"Adios Mundo","language":"Spanish"},
        {"value":"Bye World","language":"English"}
    ],
    "hellos":[
        {"value":"Hola Mundo","language":"Spanish"},
        {"value":"Hello World","language":"English"}
    ]
},
"var":{
    "_PATH":["foo","hellos",0,"value"]
}
```

**Remark:** Observe that `{"_PATH":["variable_name"]}` is equivalent to `"{variable_name}"`.

### The `_PATH_EXISTS` Function

Like `_PATH`, this function takes an array of strings and integers. It returns `true` (`1`) if this array represents a path that exists, and `false` (`0`) otherwise. For example:

```json
"object": {
    "key_1": [
        "element #1",
        [
            "element",
            "#",
            "2"
        ]
    ],
    "key_2": {
        "red": 255,
        "green": 255
    }
},
"another_object": {
    "something": "{object}"
},
"a": {
    "_PATH_EXISTS":["another_object","something","key_1",1,1]
},
"b": {
    "_PATH_EXISTS":["another_object","something","key_1",1,2]
},
"c": {
    "_PATH_EXISTS":["another_object","something","key_2","blue"]
}
```

In the code above, the variables `a` and `b` are `1` (`true`), and the variable `c` is `0` (`false`).

### The `_IF`, `_THEN`, `_ELSE` Function

Unlike other built-in functions, this one is represented by an object with three keys, rather than one. It works just as you would expect it in any programming language. In the example below, the variable `city` takes the value `"Montreal"`.

```json
"foo":true,
"cities":["Montreal","Toronto"],
"city":{
    "_IF":"{foo}",
    "_THEN":{"_PATH":["cities",0]},
    "_ELSE":{"_PATH":["cities",1]}
}
```

It is important to keep in mind that `_IF,_THEN,_ELSE` is a function, and not a construct, as in other programming languages. That means `_THEN` and `_ELSE` contain **values** to be *returned*, rather than **statements** to be *executed*.

However, you may still wish to use intermediate variables inside `_THEN` and `_ELSE`. This is possible as long as they have a `_RETURN` key, as in the example below:

```json
"foo":true,
"cities":["Montreal","Toronto"],
"city":{
    "_IF":"{foo}",
    "_THEN":{
        "city":{"_PATH":["cities",0]},
        "_RETURN":"{city}"
    },
    "_ELSE":{"_PATH":["cities",1]}
}
```

**Remark for scope geeks:** Rel uses block dynamic scope, and all variables are set only once (this is called *inmutability*). In particular that means that the variable `city` inside `_THEN` is a local variable, and does not overwrite the other equally named variable.

### The `_URL` Function

Allows you to make a call to a web service that returns a JSON object. To test this function we're using this JSON file:

`https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities`

The card below exemplifies the use of the `_URL` function. You can test it by shaking your phone and adding the card `https://gist.githubusercontent.com/wircho/327fef8392c1865dec5e/raw/5aa7e97fa166888c3bd8d0f38c4b062deebc6819/city_card`

```json
{
    "id": "city-card",
    "title": "City",
    "icon_url": "http://relevant.ai/city.png",
    "summary": "Card's summary for the library.",
    "credits": "The Web",
    "settings_type": "NONE",
    "_LOAD": {
        "json_info":{
            "_URL":"https://gist.githubusercontent.com/wircho/d6c606350f8a6dca29ee/raw/b7872ecebfcc868ff825c88fe1c5e06d13ad618c/cities"
        },
        "first_city":{
            "_PATH": ["json_info","cities",0]
        },
        "_RETURN":[
            [
                {
                    "banner":{
                        "image": {"_PATH":["first_city","picture"]}
                    }
                },
                {
                    "description":{
                        "title": {"_PATH":["first_city","name"]},
                        "body": {"_PATH":["first_city","nickname"]}
                    }
                },
                {
                    "footer":{
                        "caption": {"_PATH":["first_city","province"]}
                    }
                }
            ]
        ]
    }
}
```

![City Sample Card](https://raw.githubusercontent.com/relevant-ai/RelevantCardsDocumentation/master/montreal_card.png)

As the Hello World card, this card has only one page. See the `_LOOP` function below to learn how to display all cities from the web service.

You can also make richer requests using `_URL`, as exemplified below:

```json
"var":{
    "_URL":{
        "_ADDRESS":"http://some/web/address.html",
        "_METHOD":"POST",
        "_BODY":"a=something&b=something%20else",
        "_HEADERS": {
            "Some Header": "Some Value",
            "Another Header": "Another Value"
        }
    }
}
```

### The `_MERGE` Function

Takes an array of objects, strings, or arrays, and returns an array containing each of those objects, strings, and array elements. The syntax is as in the following example:

```json
{
    "_MERGE":[
        "String 1",
        [
            "String 3",
            "String 4",
            "String 5"
        ],
        "String 6",
        "String 7",
        [
            "String 8",
            "String 9"
        ]
    ]
}
```

### The `_LOOP` Function

Allows you to loop an array to get a new array of the same size but with modified contents. Its parameter keys are `"_ARRAY"` and `"_EACH"`. The first one is the array that is to be looped (you can make a reference to it using `_PATH`, the `"{var}"` syntax, or any other function). Inside the `"_EACH"` object you can use the local variables `"{_ITEM}"` and `"{_INDEX}"` representing the current array element and its index respectively. For example:

```json
{
    /*...
    METADATA
    ...*/
    "_LOAD": {
        "info": {
            "useless_stuff": {
                "a": ["x","y","z"],
                "b": "c"
            },
            "important_stuff": [
                {
                    "head": "My Header 1",
                    "foot": "My Footer 1"
                },
                {
                    "head": "My Header 2",
                    "foot": "My Footer 2"
                },
                {
                    "head": "My Header 3",
                    "foot": "My Footer 3"
                }
            ]
        },
        "_RETURN": {
            "_LOOP": {
                "_ARRAY": {"_PATH":["info","important_stuff"]},
                "_EACH": {
                    "headline": {"_PATH":["_ITEM","head"]},
                    "footer": {"_PATH":["_ITEM","foot"]}
                }
            }
        }
    }
}
```

**Remark:** The example above would make a lot more sense, of course, if the variable `"info"` came for a web service (`_URL`).

The `_EACH` object may contain variables itself, as long as it has a `"_RETURN"` key. For example, the `_EACH` object above may be replaced with:

```json
{
    "_ARRAY": {"_PATH":["info","important_stuff"]},
    "_EACH": {
        "h": {"_PATH":["_ITEM","head"]},
        "f": {"_PATH":["_ITEM","foot"]},
        "_RETURN": {
            "headline": "{h}",
            "footer": "{f}"
        }
    }
}
```
### Basic String Functions

Basic string functions are built in functions that take a string and return a string. Some available basic string functions are listed below:

**`_URL_ENCODE`:** Percent encoding for URL parameters.

**`_URL_DECODE`:** Inverse of above.

**`_HTML_ENCODE`:** HTML special characters encoding.

**`_HTML_DECODE`:** Inverse of above.

**`_UPPERCASE`:** Convert a String to uppercase.

### Logic/Boolean Functions

These are functions that take the role of common logic operators, returning a boolean (`true` or `false`). Some of these are listed below:

**`_NOT`:** Takes a boolean and returs its opposite.
**`_AND`:** Takes an array and returns whether all of its elements are `true`.
**`_OR`:** Takes an array and returns whether any of its elements are `true`.
**`_EQUAL`:** Takes an array of two elements and returns whether they are equal.

### The `_REPLACE` Function

This function allows you to replace all ocurrences of the string parameter keyed by `"_SEARCH"` with the string parameters keyed by `"_REPLACEMENT"` in the string parameters keyed by `"_STRING"`. You can also perform regular expressions replacements by having a parameter `"_REGEX"` equal `true` or `1`. For example:

```json
{
    "_REPLACE": {
        "_STRING":"03/17/2015",
        "_SEARCH":"/",
        "_REPLACEMENT":"-"
    }
}
```

### The `_DATE` Function

This function always returns a string representing a date in a given format. The only required parameter is `"_FORMAT_OUT"`. The example below returns today's date in the format yyyy-MM-dd:

```json
{
    "_DATE": {
        "_FORMAT_OUT":"yyyy-MM-dd",
    }
}
```

[Click here for a list of all available formats](http://www.unicode.org/reports/tr35/tr35-31/tr35-dates.html#Date_Format_Patterns).

You can use the parameter `"_OFFSET"` to offset the current date by any number of seconds. This example returns yesterday's date:

```json
{
    "_DATE": {
        "_OFFSET": -86400,
        "_FORMAT_OUT": "yyyy-MM-dd",
    }
}
```

Instead of using today's date, you can also input a fixed date using the parameter "`_VALUE`", which is either a timestamp (seconds since 1970), or a formatted date string. In the latter case, it is also necessary to have a parameter "`_FORMAT_IN`". For example:

```json
{
    "_DATE": {
        "_VALUE": "01-11-2022",
        "_FORMAT_IN": "MM-dd-yyyy",
        "_FORMAT_OUT": "yyyy-MM-dd",
    }
}
```

### The `_SPLIT` Function

This function takes a string `"_STRING"` and a separator string `"_SEPARATOR"`. It returns the array that results from splitting the string by the separator. For example

```json
{
    "_SPLIT": {
        "_STRING": "a,b,c,d",
        "_SEPARATOR": ",",
    }
}
```

### The `_MATH` function

This function allows you to parse mathematical expressions from a string. For example:

```json
{
    "_MATH": "5*2^10 + 1"
}
```

```json
{
    "_MATH": "5*2^10 + {some_var}^{another_var}"
}
```

### The `_SORT` Function

This function sorts an array parameter keyed by `"_ARRAY"` by either a path or a function. Ordering by path is simpler, as you just write an array of keys or indexes that represents a path, and the array is sorted either alphabetically or numerically. For example:

```json
{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_PATH": ["info","number"]
    }
}
```

You can also reverse the order as follows:

```json
{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_PATH": ["info","number"],
        "_REVERSE": true
    }
}
```

Sorting by function is just as easy, except the key is now `"_BY_FUNCTION"` and it holds a function (see User Defined Functions), i.e., an expression depending on two local parameters. The local parameters are keyed `"_A"` and `"_B"` and represent two arbitrary items in the `"_ARRAY"`. The function/expression must return `1` (`true`) if the items should be ordered `"_A","_B"` and `0` (`false`) if they should be ordered `"_A","_B"`

```json
{
    "_SORT": {
        "_ARRAY": [{"info":{"number":5}},{"info":{"number":3}},{"info":{"number":8}}],
        "_BY_FUNCTION": {
            "_MATH":{
                "_CONCAT":[
                    {"_PATH":["_A","info","number"]},
                    "<",
                    {"_PATH":["_B","info","number"]}
                ]
            }
        }
    }
}
```

### Filtering Arrays Using `_LOOP`

It is important to know that `null` values get removed entirely from computed objects. Thus one could for example filter out every other element of an array by simply doing:

```json
{
    "_LOOP": {
        "_ARRAY": /* ... */,
        "_EACH" : {
            "_CONDITIONAL": {
                "_IF":{"_MATH":"{_INDEX}%2==0"},
                "_THEN":null,
                "_ELSE":"{_ITEM}"
            }
        }
    }
}
```

### The `_WEB_CALLBACK` Function (Web View for User Input or Authentication)

It is possible to pop up a webview when a relevant card is loaded, be it for user authentication, or any other form of user input. This is achieved with the `_WEB_CALLBACK` function. This function will return either when the user dismisses the webview manually, or when a certain base URL is loaded as a result of navigation. This functions looks like this:

```json
{
    "_WEB_CALLBACK": {
        "_ADDRESS":"http://relevant.ai/",
        "_CALLBACK_BASE":"relevant.ai/callback"
    }
}
```

Notice that the callback base URL **does not contain the protocol http://**. If the user dismisses the webview, then the function returns `null`. Otherwise it returs an object of the form:

```json
{
    "url":/* URL ABSOLUTE STRING */,
    "body":{
        /* URL PARAMETERS AND VALUES */
    },
    "method":/* "GET" OR "POST" OR OTHER */,
    "headers":{ 
        /* HEADER TITLES AND VALUES */
    }
}
```

Notice that this return only contains information about the request that was made. This is because the webview is dismissed before its content is loaded.

## Templates (Card Appearance):

Let us go back to the first example of a fully functional card:

```json
{
    "id": "card-id-string",
    "title": "My Card's Title",
    "icon_url": "https://url/to/icon.png",
    "summary": "My card's description.",
    "credits": "Some Website or Source",
    "templates": [
        "headline",
        "footer"
    ],
    "settings_type": "NONE",
    "_LOAD": {
        "_RETURN": [
            {
                "headline": {"text": "My Header 1"},
                "footer": {"text": "My Footer 1"}
            },
            {
                "headline": {"text": "My Header 2"},
                "footer": {"text": "My Footer 2"}
            }
        ]
    }
}
```

This card has two templates `headline` and `footer`, in that order. This is why the `_LOAD` object returns an array of objects of the form `{"headline":{...},"footer":{...}}`. In general, if a card is using templates `template_1`, `template_2`, ..., `template_n`, then the `_LOAD` object must return an array of objects of the form `{"template_1":{...},...,"template_n":{...}}`, where the `{...}`'s hold parameters that skin each template's labels and images. Below is a list of all available templates and their parameters: (Check out Template-Structures)

```json
{
    "title":{"text":/*...*/},
    "title-two-lines":{"text":/*...*/},
    "headline":{"text":/*...*/},
    
    "short-description":{"text":/*...*/},
    "long-description":{"text":/*...*/},
    
    "large-image-square":{"image":/*...*/},
    "large-image-3x4":{"image":/*...*/},
    "banner":{"image":/*...*/},
    "thumbnail":{"image":/*...*/},
    "profile-image":{"image":/*...*/},
    
    "footer":{"text":/*...*/},
    
    "description":{
        "title":/*...*/,
        "subtitle":/*...*/,
        "thumb":/*...*/
    },
    "profile":{
        "title":/*...*/,
        "subtitle":/*...*/
    },
    "large-stats":{
        "title-1":/*...*/,
        "title-2":/*...*/,
        "title-3":/*...*/,
        "title-4":/*...*/,
        "title-5":/*...*/,
        "value-1":/*...*/,
        "value-2":/*...*/,
        "value-3":/*...*/,
        "value-4":/*...*/,
        "value-5":/*...*/
    },
    "stats":{
        "title-1":/*...*/,
        "title-2":/*...*/,
        "title-3":/*...*/,
        "value-1":/*...*/,
        "value-2":/*...*/,
        "value-3":/*...*/
    },
    "image-stats":{
        "stats":/*...*/,
        "image":/*...*/
    },
    "match":{
        "team1": {
            "image":/*...*/,
            "name":/*...*/,
            "score":/*...*/
        },
        "team2": {
            "image":/*...*/,
            "name":/*...*/,
            "score":/*...*/
        },
        "middle":/*...*/,
        "time":/*...*/
    },
    "nearby":{
        "image":/*...*/,
        "place":/*...*/,
        "address":/*...*/,
        "latitude":/*...*/,
        "longitude":/*...*/
    },
    "thumb-scalar":{
        "number":/*...*/,
        "units":/*...*/,
        "title":/*...*/,
        "subtitle":/*...*/,
        "image":/*...*/
    },
    "transit":{
        "number":/*...*/,
        "place":/*...*/,
        "address":/*...*/,
        "time-1":/*...*/,
        "time-2":/*...*/,
        "time-3":/*...*/
    }
}
```

## Actions:

Relevant cards allow you to perform actions by holding down your finger on the card. The default actions are *refresh*, *share* and *zoom-in*. You can add more actions to the library using the `actions` template. For this you don't need to add `"actions"` to your `"templates metadata"`. You simply need to add an `"actions"` key to each element of the array returned by the `_LOAD` object. The value of this key will be an array of added actions. All of the elements of this array look as follows:

```json
{
    "_ACTION":/*...*/,
    "_ICON":/*...*/
}
```

The value of `"_ICON"` can be `"done-icon"`, `"eye-icon"`, `"fav-icon"`, `"heart-icon"`, `"repost-icon"`, `"source-icon"`, `"downvote-icon"`, `"map-icon"`, or `"upvote-icon"`. The action must be one of the following currently available functions:

```json
{
    "_WEBVIEW":/* SOME SOURCE URL */
}
```

```json
{
    "_MAPVIEW":{
        "title":/*...*/,
        "latitude":/*...*/,
        "longitude":/*...*/
    }
}
```






