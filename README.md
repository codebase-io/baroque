# [The Baroque Data Transfer Format](https://baroquedt.io)

*Hey! If you are bored one day, invent a new data transfer format. No "strings" attached.*

> [!WARNING]  
> This is very early days and you should not use this spec in production.

### About

**Baroque** is a data interchange format, for the web platform, that focuses on taking advantage of rich data types present in
today's high level programming languages. Starting from a well-defined base of standard types, serializers/parsers
can progressively add support for more specific and rich types.
Vendors are welcomed to implement their own types.

### Proposal 

- UTF-8 as default encoding
- 5 base data types, not nullable
- rich data types, derived from base ones
- support for tabulated formats
- [space] character as separaror between names and values, or multiple values when tabulated
- support for comments
- backwards compatibility between versions
- easy to extend, with vendor specific types

## Base types (v0.1)

 - *bool*: `1` or `0`
 - *int*: -2<sup>10</sup> to 2<sup>10</sup>
 - *string*: `"should be double-quoted"`
 - *float*: floating point `0.9999` . Trimming allowed.
 - *double*: fixed point `198.876`. Defaults to (6,2) if not specified. Trimming not allowed.

### Derived types (v0.2)

These types have own keywords, but are derived from the base ones.
If the parser does not supports the type, it should convert to the one it derives from.

 - `blob`: derived from string
 - `date` and `datetime`: derived from string
 - `enum`: derived from string
 - `array`: derived from string
 - `object`: derived from string

### Composed/Rich types (v0.3)

Although for presentation pusposes we have used separate categories for this part, we can consider them derived types,
however it's more clear that where these ones come from:

 - `int(32)` and `int(64)`
 - `string(email)`, `string(multiline)` or `string(uuid)`
 - `datetime(ISO8601)` and `date('Y-m-d')`
 - `enum(string)` or `enum(int(64))`
 - `array(string(int))` or `array(int(object))`
 - check the full list here (TBD)

Parsers should implement a "strict mode" feature, on by default, which will prevent converting from derived to 
base types and throw error early. 
When strict mode is off, parsers should convert all unrecognized types to the "string" type.

## Versioning

As you might have noticed, type support is part flagged with a semantic version, with the purpose of indicating 
when new types are added by increasing the version. `0.3.1` will likely be the next. 

## The exchange format (Spec)

Although we started discussing the types, as they are first class citizens, the format derives 
from a very simple and familiar approach: `variable value`.

As noted previously, space is used to denote that something ends, for this reson variable names are limited
to: `a-Z`, `0-9`, `_` and `-`:

 - Valid names: `myFirstVariable`, `__debug` or `--debug`. 
 - Invalid names: `my var` - contains space or `*flag` - contains forbidden character
 
 
Lines starting with `#` should always be treated as comments, and ignored when parsing values. 
Parsers may add extensions that treat subsequent shebang `!#` lines as configuration parameters.   
 
### Baroque documents

A *baroque* document can be presented in two forms: sigle and tabular. 

#### Single document format

```
> baroque(v0.3)
> charset(utf8mb4)
> copyright('Space Force'/2024)

# System document id
_id:string(uuidv4) "abcd-efgh-ijkl-mnop"

# The document version
_version:float 0.3

# Can be downloaded
is_public:bool 1

date_updated:datetime('Y-m-d H:i:s') "2024-03-20 15:10:10"
```

The first line, the shebang indicates this is a `baroque` document and declares the version.
This line is optional, however baroque documents should always have a version decared: either 
using this format, or specifying it via a HTTP Headers: `X-Baroque-Ver: 0.3`.
If both the document and the HTTP header are present, the version declared in the document should 
be used.     

When a single document is parsed new lines `\n` or `\r\n` should be ignored. 
For multiline strings, the type `string(multiline)` should be used as it's more flexible, and prevents 
parsers from assuming where the lines should be.
 
The heading of the document may also declare extensions. Each declaration should also start with `>`. 
New lines and trailing spaces in the heading should be ignored.  

#### Tabular document format

```
> baroque(v0.3/tabulated)

id:int(id) firstName:string lastName:string email:string(email) isAuthor:bool
21 "Josh" "Konstanza" "josh@baroquefmt.io" 0
47 "Miriam" "Becky-Marr" "miriam@baroquefmt.io" 1
```

Suggestions are welcomed, as we are looking to draft an initial v0.1 parser and extend to rich types.

### How

 - simple rules
 
	- UTF-8 as default encoding
    - first line indicating format and version
		- `tabular` keyword indicates the file should be parsed as a table with the header on the first line
		- `!# baroque(v0.1)` 
		- `!# baroque(v0.1/tabular)`
		- `!# baroque(tabular)` 
		- first line is optional, in case of protocole supporting headers (like HTTP) these may be indicated using a header: `HTTP-Baroque: v0.1/tabular`
		- if the version is missing the parsers should assume the latest supported version
	- space is a separator between values
		- `var value` when single
		- `value value value` when tabulated
	- line starting with `#` should be treated as comments
	- extensions may support commments to indicate content encoding
	- no nullable values: it's up to the parsers to implement null support (if desired, as default values), so is a string is empty it may be converted to null if the programming language has support for it. The serializers should always convert null to empty string for string or 0 for int/float
	- boolian values can be 1 or 0, we dropped true/false as it adds unnecessary overhead, so bool can be treated as int
 - variables and values
 - single or tabulated (like csv)
 - rich data types, derived from basic ones. Ensures bakwards compatibility and progressive enhancements between appplications:

 	- string(multiline) indicates a string that can span multiple lines
	- string(multiline(/r/n)) multiline string, lines separated by \r\n
	- int(64) indicates an int that should reserve 64 
	- json indicates a json
	- string(file(text/html)) indicates a string that should treated as a html file 
	- string(url(application/json)) a url to a json file
	- int(timestamp) this is an int that indicates a timestamp
 

### Examples:

Single baroque file
```
!# baroque(v0.1)

# Comments: this is a configuration file for the broker service
# Please don't use in production
# The simple format:
# {variable}:{type}{space} {value}
# Strings both sinle and multiline should be enclosed in ""
# Variable names should only have alphanumeric, _ or -

document_id:string(uuidv4) "abcd-efgh-ijkl-mnop"
api_endpoint:string(url) "https://api-endpoint-1.io"
api_version:int(32) 22
developer_notes:string(multiline(\r\n)) "There are some notes\r\nSpanning multiple lines\r\n"
html_version:string(url(text/html)) "http://example.com/index.html"
date_updated:string(datetime('Y-m-d H:i:s')) "2024-03-20 15:10:10"
isActive:bool 1
pointsWon:float 6472.33
balanceAmount:decimal(10,2) 1234.34
``` 

Tabulated example (like csv)

```sh
!# baroque(v0.1/tabulated)

id:int(id) firstName:string lastName:string email:string(email) isAuthor:bool
21 "Josh" "Destiny" "josh@baroque.com" 0
47 "Miriam" "Becky-Marr" "miriam@baroque.com" 1
```

Baroque schema file
```sh
!# baroque(v0.1/schema)

# The id of the document
id:int(id) 
# The user who uploaded the document
userEmail:string(email)
# The description of the document
description:string(multiline(\n))
# Date the document was last updated
updatedAt:datetime(ISO8601)
# Research papers linking the document, list of system uuids
papers:enum(string(uuid))
```

Parsers can optionally trim variable names

### Serializing arrays and objects

 - enum type: list of values
 - array for mor complex structures (TBD)
 - objects for vendor supported extensions

```
!# baroque(v0.1)

# This is an example of serializing arrays

# Simple enum
supportedLocales:enum(string) ["en_US", "fr_FR", "fr_BE"]

# Enum of multiline string sepparated by \n
suggestedCharacterNames:enum(string(multiline(\n))) ["Wanda\nSuper-Man", "Developer\nEngineer"]

# Array with string keys and int values
downloadsPerWeek:array(string(int)) ["file_a" 3652, "file_B" 456]

# Array with int keys and any value. Start `*` denotes any tipe. 
# Highly discouraged, but yeah we need a loose end as I am lazy too.
# Parsers should do their best in this case and report errors early.
deprecatedPackages:array(int(*)) [1 "baroque/v.01", 2 ["json/v1", "xml/v2"], 4 873635266.988]

# Although we could go on an on, with nesting arrays, 
# more complex structures should be put into objects. Please!
# Object support should be implemented via extensions. Parsers should be pluggable to allow implementing custom objects, which by the way can be pretty much anything.
websiteStatuses:object(json) {"google.com" "up", "example.com" "down"}

databaseSchema:object(svg)
```

## Custom types

Serializers/parsers should allow for implementing custom types. However this should always be prefixed with a vendor like this:

```
!# baroque(v0.1)

# Examples
databaseSchema:vendor/type(drawing) "base64:"

vaultKey:securityTeam/encrypted(rsa) "qjswqdufoqldnsbeiwfgweifewfuwfee"
```
