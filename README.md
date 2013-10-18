JSNN
====

This is a fork of [zserge's jsmn library][2] that adds a few (much needed,
in my opinion) goodies. I did not officially fork using github
because the mirrors seem to be pretty stagnant; [noct/jsmn](https://github.com/noct/jsmn)
for one.

I have added the ability to extract data from the list of tokens created
by the parser using friendly 'ol javascript syntax. For example, given
the json string:

```json
{
    "dogs": [
        {
            "name": "spot",
            "breed": "terrier"
        },
        {
            "name": "gracie",
            "breed": "golden retriever"
        }
    ],
    "cats": [
        {
            "name": "pickles",
            "breed": "sphynx"
        }
    ]
}
```

After parsing with `jsnn` (assuming the tokens are stored in `tokens`
and the json string is stored in `json`), you can extract the breed of Gracie with
the following:

```c
jsnntok_t *breed;
breed = jsnn_get(tokens, "dogs[1].breed", tokens, json);
```

This works on subpaths, too. For example, you can grab the token that points to
the Gracie object, then take attributes from that.

```c
jsnntok_t *gracie, *breed;
gracie = jsnn_get(tokens, "dogs[1]", tokens, json);
breed = jsnn_get(gracie, "breed", tokens, json);
```

Below is the documentation from jsmn.

JSMN
====

jsmn (pronounced like 'jasmine') is a minimalistic JSON parser in C.  It can be
easily integrated into resource-limited or embedded projects.

You can find more information about JSON format at [json.org][1]

Library sources are available at [bitbucket.org/zserge/jsmn][2]

Philosophy
----------

Most JSON parsers offer you a bunch of functions to load JSON data, parse it
and extract any value by its name. jsmn proves that checking the correctness of
every JSON packet or allocating temporary objects to store parsed JSON fields
often is an overkill. 

JSON format itself is extremely simple, so why should we complicate it?

jsmn is designed to be	**robust** (it should work fine even with erroneous
data), **fast** (it should parse data on the fly), **portable** (no superfluous
dependencies or non-standard C extensions). An of course, **simplicity** is a
key feature - simple code style, simple algorithm, simple integration into
other projects.

Features
--------

* compatible with C89
* no dependencies (even libc!)
* highly portable (tested on x86/amd64, ARM, AVR)
* about 200 lines of code
* extremely small code footprint
* API contains only 2 functions
* no dynamic memory allocation
* incremental single-pass parsing
* library code is covered with unit-tests

Design
------

The rudimentary jsmn object is a **token**. Let's consider a JSON string:

	'{ "name" : "Jack", "age" : 27 }'

It holds the following tokens:

* Object: `{ "name" : "Jack", "age" : 27}` (the whole object)
* Strings: `"name"`, `"Jack"`, `"age"` (keys and some values)
* Number: `27`

In jsmn, tokens do not hold any data, but point to token boundaries in JSON
string instead. In the example above jsmn will create tokens like: Object
[0..31], String [3..7], String [12..16], String [20..23], Number [27..29].

Every jsmn token has a type, which indicates the type of corresponding JSON
token. jsmn supports the following token types:

* Object - a container of key-value pairs, e.g.:
	`{ "foo":"bar", "x":0.3 }`
* Array - a sequence of values, e.g.:
	`[ 1, 2, 3 ]`
* String - a quoted sequence of chars, e.g.: `"foo"`
* Primitive - a number, a boolean (`true`, `false`) or `null`

Besides start/end positions, jsmn tokens for complex types (like arrays
or objects) also contain a number of child items, so you can easily follow
object hierarchy.

This approach provides enough information for parsing any JSON data and makes
it possible to use zero-copy techniques.

Install
-------

To clone the repository you should have mercurial installed. Just run:

	$ hg clone http://bitbucket.org/zserge/jsmn jsmn

Repository layout is simple: jsmn.c and jsmn.h are library files; demo.c is an
example of how to use jsmn (it is also used in unit tests); test.sh is a test
script. You will also find README, LICENSE and Makefile files inside.

To build the library, run `make`. It is also recommended to run `make test`.
Let me know, if some tests fail.

If build was successful, you should get a `libjsmn.a` library.
The header file you should include is called `"jsmn.h"`.

API
---

Token types are described by `jsmntype_t`:

	typedef enum {
		JSMN_OBJECT,
		JSMN_ARRAY,
		JSMN_STRING,
		JSMN_PRIMITIVE
	} jsmntype_t;

**Note:** Unlike JSON data types, primitive tokens are not divided into
numbers, booleans and null, because one can easily tell the type using the
first character:

* <code>'t', 'f'</code> - boolean 
* <code>'n'</code> - null
* <code>'-', '0'..'9'</code> - number

Token is an object of `jsmntok_t` type:

	typedef struct {
		jsmntype_t type; // Token type
		int start;       // Token start position
		int end;         // Token end position
		int size;        // Number of child (nested) tokens
	} jsmntok_t;

**Note:** string tokens point to the first character after
the opening quote and the previous symbol before final quote. This was made 
to simplify string extraction from JSON data.

All job is done by `jsmn_parser` object. You can initialize a new parser using:

	struct jsmn_parser parser;
	jsmntok_t tokens[10];

	// js - pointer to JSON string
	// tokens - an array of tokens available
	// 10 - number of tokens available
	jsmn_init_parser(&parser, js, tokens, 10);

This will create a parser, that can parse up to 10 JSON tokens from `js` string.

Later, you can use `jsmn_parse(&parser)` function to process JSON string with the parser.
If something goes wrong, you will get an error. Error will be one of these:

* `JSMN_SUCCESS` - everything went fine. String was parsed
* `JSMN_ERROR_INVAL` - bad token, JSON string is corrupted
* `JSMN_ERROR_NOMEM` - not enough tokens, JSON string is too large
* `JSMN_ERROR_PART` - JSON string is too short, expecting more JSON data

If you get `JSON_ERROR_NOMEM`, you can re-allocate more tokens and call
`jsmn_parse` once more.  If you read json data from the stream, you can
periodically call `jsmn_parse` and check if return value is `JSON_ERROR_PART`.
You will get this error until you reach the end of JSON data.

Other info
----------

This software is distributed under [MIT license](http://www.opensource.org/licenses/mit-license.php),
 so feel free to integrate it in your commercial products.

[1]: http://www.json.org/
[2]: https://bitbucket.org/zserge/jsmn
