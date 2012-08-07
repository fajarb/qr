#  qr tools  ############################################################

These four tools are used to generate QR codes in large batches. The QR
codes encode a url. The QR codes can be laid out on a page, ready to be
printed out onto sticker sheets or labels.

All the tools display a help message when invoked with the `-h` or
`--help` option.


## `bug` ################################################################
(written in Ruby)

`bug` (batch url generator) generates a list of urls that can later be
encoded into QR codes. Each URL in the list consists of a *base URL* that
is the same for all URLs in the list, and an *ID* that increments for
each URL. An example output of bug is

    http://www.google.com/01
    http://www.google.com/02
    ...
    http://www.google.com/99

where the base URL is `http://www.google.com/` (note the slash at the end).

The start and end IDs can be specified with the `-s` and `-e` options. If 
the start ID has fewer digits than the end ID, then some of the IDs will be
padded out with zeroes. (See the example above: note the id `01`. If this
behaviour is not desired, you can disable it with the `--no-lead-zeroes`
flag.

The base URL can be changed with the `-u` option. By default, it points
to a URL on the EECS intranet.

`bug` writes it's output to stdout; you can redirect it to a file.

Example:

    ./bug -s 0 -e 49 > urls.txt

writes this output to urls.txt

    https://intranet.eecs.qmul.ac.uk/staffonly/deskallocation/id/01
    https://intranet.eecs.qmul.ac.uk/staffonly/deskallocation/id/02
    ...
    https://intranet.eecs.qmul.ac.uk/staffonly/deskallocation/id/48
    https://intranet.eecs.qmul.ac.uk/staffonly/deskallocation/id/49


## `qetqr` ##############################################################
(written in Ruby)

`getqr` is used to generate QR codes from a list of strings, such as the
one produced by `bug`. When given a file containing a list of strings,
`qetqr` will encode each string as a QR code and write the QR codes to an 
image.

`getqr` takes a single argument -- a file containing a newline-delimited
list of strings, which can be URLs. The `-d` option specifies the
directory where QR codes should be saved; the directory should already
exist. The `-s` option specifies the length of the side of the QR code, in
pixels. (QR codes are always sqare).

`getqr` saves the QR codes as PNG files. The name of the file is the same
as the *ID* generated by `bug`, i.e. the last sequence of digits in the
string. If getqr is given the file generated by the example for `bug`
above, it will generate images `00.png` through to `49.png`.

Example:

    ./getqr -d /tmp/images -s 512 urls.txt

Encodes all the urls in `urls.txt` as 512x512px QR codes and writes them
to `/tmp/images` as PNG files.


## `QrLabeller` #########################################################
(Written in Java. Should be invoked using the shell script `qrlabeller`.)

`QrLabeller` adds a numeric label to each image in a directory. It is
intended to label QR codes generated by `getqr`, so it expects each image
to be named `XXX.png`, where `XXX` is a (fixed or variable width) number.

`QrLabeller` adds some white space to the right hand side of the QR code,
and then writes the label in this white space. The contents of the label
will be `XXX` (the filename, minus the .png extension).

The shape and size of the label is compiled in. `QrLabeller` will try to
label EVERY file in the directory it is given; if some of the files are
not images, bad things may happen.

Example:

    ./qrlabeller /tmp/images

Applies a label to every file in `/tmp/images`, overwriting the original
image.


## `LabelPrint` #########################################################
(Written in Java. Should be invoked using the shell script `qrlabeller`.)

`LabelPrint` lays out many QR codes onto a page layout that is suitable for
printing onto sticker sheets or labels. It takes a directory of QR code
images and outputs several JPEG images, each of which contains several QR
codes laid out correctly. The output will be named `PageX.jpg`, where `X`
is a number that increments starting at 1.

The layout itself is provided by a Java class that implements the
`PageLayout` interface. This interface provides accessors that are used by
`LabelPrint` to determine the page margins, spacing between images, and
other parameters. Two classes that implement `PageLayout` are provided:
`AverySquareSticker` and `AveryAddressLabel`. In order to lay out QR codes 
on a different sticker layout, create a class that implements `PageLayout`
and use it in the code for `LabelPrint`.

Note that `LabelPrint` processes every file in the directory given to it.
If some of the files are not images, bad things may happen. The output is
saved in the same directory as the imput, so if you run it twice, the
output of the first invocation will be treated as part of the input to
the second invocation -- you probably don't want this.

`LabelPrint` expects every image that it processes to be in the correct
shape for printing onto the label; it simply lays images out, it does not
reshape them. If the input images (eg. generated by `QrLabeller`) are the
wrong shape, the output won't be pretty.

Example:

    ./labelprint /tmp/images

Lays out every QR code in `/tmp/images` onto sheets named `Page1.jpg`,
`Page2.jpg`... which are saved in `/tmp/images`.
