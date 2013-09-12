# XLSX Template

This module provides a means of generating "real" Excel reports (i.e. not CSV
files) in NodeJS applications.

The basic principle is this: You create a template in Excel. This can be
formatted as you wish, contain formulae etc. In this file, you put placeholders
using a specific syntax (see below). In code, you build a map of placeholders
to values and then load the template, substitute the placeholders for the
relevant values, and generate a new .xlsx file that you can then serve to the
user.

## Placeholders

Placeholders are inserted in cells in a spreadsheet. It does not matter how
those cells are formatted, so e.g. it is OK to insert a placeholder (which is
text content) into a cell formatted as a number or currecy or date, if you
expect the placeholder to resolve to a number or currency or date.


### Scalars

Simple placholders take the format `${name}`. Here, `name` is the name of a
key in the placeholders map. The value of this placholder here should be a
scalar, i.e. not an array or object. The placeholder may appear on its own in a
cell, or as part of a text string. For example:

    | Extracted on: | ${extractDate} |

might result in (depending on date formatting in the second cell):

    | Extracted on: | Jun-01-2013 |

Here, `extractDate` may be a date and the second cell may be formatted as a
number.

### Columns

You can use arrays as placeholder values to indicate that the placeholder cell
is to be replicated across columns. In this case, the placeholder cannot appear
inside a text string - it must be the only thing in its cell. For example,
if the placehodler value `dates` is an array of dates:

    | ${dates} |

might result in:

    | Jun-01-2013 | Jun-02-2013 | Jun-03-2013 |

### Tables

Finally, you can build tables made up of multiple rows. In this case, each
placeholder should be prefixed by `table:` and contain both the name of the
placeholder variable (a list of objects) and a key (in each object in the list).
For example:

    | Name                 | Age                 |
    | ${table:people.name} | ${table:people.age} |

If the replacement value under `people` is an array of objects, and each of
those objects have keys `name` and `age`, you may end up with something like:

    | Name        | Age |
    | John Smith  | 20  |
    | Bob Johnson | 22  |

If a particular value is an array, then it will be repeated accross columns as
above.

## Generating reports

To make this magic happen, you need some code like this:

    var XlsxTemplate = require('xlsx-template');

    // Load an XLSX file into memory
    fs.readFile(path.join(__dirname, 'templates', 'template1.xlsx'), function(err, data) {

        // Create a template
        var template = new XlsxTemplate(data);

        // Replacements take place on first sheet
        var sheetNumber = 1;

        // Set up some placeholder values matching the placeholders in the template
        var values = {
                extractDate: new Date(),
                dates: new Date("2013-06-01"), new Date("2013-06-02"), new Date("2013-06-03"),
                people: [
                    {name: "John Smith", age: 20},
                    {name: "Bob Johnson", age: 22}
                ]
            };

        // Perform substitution
        template.substitute(sheetNumber, values);

        // Get binary data
        var data = template.generate();

        // ...

    });

At this stage, `data` is a string blob representing the compressed archive that
is the `.xlsx` file (that's right, a `.xlsx` file is a zip file of XML files,
if you didn't know). You can send this back to a client, store it to disk, 
attach it to an email or do whatever you want with it.

## Caveats

* The spreadsheet must be saved in `.xlsx` format. `.xls`, `.xlsb` or `.xlsm`
  won't work.
* Merged cells are complicated. Things may go wrong if you have placeholders
  inside, to the left of, or above merged cells.
* Placeholders only work in simple cells, not data tables or pivot tables or
  other such things.