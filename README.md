worklog-tools
=============

This software is part of a system for recording academic output and reporting
it in documents like CVs or publication lists. 

I was motivated to write these scripts for two reasons:

* I want to have my CV and publications list availabled as both nice printable
  PDFs *and* slick web-native webpages. And I don’t want to have to maintain
  multiple fancy documents by hand.
* I want the computer to automatically do things like count citations and
  calculate my *h*-index, because those are the kinds of things that we
  invented computers to do.

Because of the *first* issue, I don’t feel it’s sufficient to simply maintain
an ever-growing LaTeX document. You can convert it to HTML, or just post the
PDF online, but that doesn’t give results that are nearly as good as they can
be — that is, they’re not at all close to what you’d create if you decided to
make a slick, web-native list of publications.

Because of the *second* issue, I want to record my academic output in some
kind of simple data format so that I can easily write scripts to pull out
useful information and statistics.

In these tools, I log activities in a *very simple* structured text format —
[“ini file”][inifile] format, in particular. Python scripts can read these
files, do things like compute *h*-indices, and then fill in LaTeX/HTML
templates with information such as lists of talks, courses taught, or
publications.

[inifile]: http://en.wikipedia.org/wiki/INI_file

No unusual Python modules are required, and the scripts should be compatible
with any recent Python 2.x.


Batteries included
------------------

This system has three pieces:

* The text files logging academic output
* The LaTeX/HTML templates used to generate output documents
* The scripts used to fill the latter using data gathered from the former.

The scripts are found in the same directory as this file. The `example`
subdirectory contains sample copies of templates (in `templates/`) and log
files (in `2012.txt`, `2013.txt`).

To get started, you might want to take a look at the log files to see how they
look. Entries are line-oriented and come in paragraphs headed by a word
encased in square brackets. A typical entry from `examples/2013.txt` is:

    [talk]
    date = 2013 Apr
    where = OIR seminar, Harvard/Smithsonian Center for Astrophysics
    what = Magnetic Activity Past the Bottom of the Main Sequence
    invited = n

(The precise file format is defined among the “Technical details” below.)
Again, a big point of emphasis is that this data format is very simple.
Whenever I give a talk, it takes almost no brainpower to append another
`[talk]` entry to the current year’s log file.

The template files, on the other hand, are complicated since they go to some
effort to create attractive output. (Well, currently, this is much more true
of the LaTeX templates than the HTML templates.) Most of this effort is in
initialization, so the ends of the files are where the action is. For
instance, toward the bottom of `example/templates/cv.tmpl.tex` you’ll find:

    FORMAT \item[|date|] \emph{|where|} \\ ``|what|''

    \section*{Professional Talks --- Invited}
    \begin{datelist} % this is a custom environment for nice spacing
    RMISCLIST_IF talk invited
    \end{datelist}

    \section*{Professional Talks --- Other}
    \begin{datelist}
    RMISCLIST_IF_NOT talk invited
    \end{datelist}

The lines beginning with ALL_CAPS trigger actions in the templating scripts.
The `RMISCLIST_IF` directive writes a sequence of `talk` data items in
reversed order, filtering for an `invited` field equal to `y`, filling in a
template specified by the most recent `FORMAT` directive. Strings between
pipes (`|what|`) in the `FORMAT` are replaced by the corresponding values from
the data files. (Again, the precise functionalities of the various directives
are defined in the “Technical details” below.)

Finally, the `Makefile` in the `example` directory wires up commands to
automatically create or update the output files using the standard `make`
command. If you run `make` in the `example` directory, the four outputs
`cv.pdf`, `pubs.pdf`, `cv.html`, and `pubs.html` will get created. You can
take a look to see the general visual style and the sections that the default
templates include. It’s not complicated to adjust the sections, or to
copy the templating commands into your preexisting CV LaTeX file.


Getting started
---------------

To get started using this system for yourself in the absolutely quickest way,
you can just personalize the example templates and start populating
`<year>.txt` files from your existing CV.

There are only very loose constraints on how you want to name or arrange your
files, though. The only place in which the relatively locations of the data
files (templates and `<year>.txt`) and the scripts (`cvlib.py` and friends)
matter is in the `Makefile` that invokes the scripts.

The main processing script reads in data from every file in the current
directory whose name ends in `.txt`. The files are processed in alphabetical
order.

A few of the templating directives read in small, standalone sub-template
files. These are searched for first in a directory named `templates` below the
current directory, and then in one named `templates` below the directory
containing the scripts.


More details and customization
------------------------------

The set of sections that I use in my CV and publication list probably don’t
match exactly with the ones that you’d like to use. The aim of this system
is to make it easy to take slightly different approaches.

(The visual appearance of my documents may not match your preferences either.
That’s entirely up to the LaTeX and HTML templates and your level of interest
in futzing with them. This angle won’t be discussed further here.)

### Generic lists

The main method for filling the templates with data is a combination of
`RMISCLIST` and `FORMAT` directives. These provide almost complete flexibility
because you can define whatever fields you want in your `<year>.txt` files
and get them inserted by writing a `FORMAT` directive.

The main constraint is in the ability to *filter* and *reorder* records: the
built-in facilities for doing so are limited, though so far they’ve been
sufficient for my needs. The only supported ordering is “reversed”, where
that’s relative to the order that the data files are read in: alphabetically
by file name, beginning to end. Since I name my files chronologically and
append records to them as I do things, this works out to reverse chronological
order, which is generally what you want.

As for filtering, the `RMISCLIST` directives select log records of only a
certain type, where the “type” is defined by the word inside square brackets
beginning each record (e.g., `[talk]`). The `RMISCLIST_IF` and
`RMISCLIST_IF_NOT` directives further filter checking whether an item in each
record is equal to `y`, with a missing item being treated as `n`.

To extend this behavior, you’re going to need to edit `cvlib.py`. See the
`cmd_rev_misc_list*` functions and `setup_processing`, which activates
different directives.

### Publication lists

Publications (`[pub]` records) follow a more detailed format to allow
automatic fetching of citation counts, generation of reference lists with
links, and computation of statistics such as the *h*-index.

Publication records are read in and then “partitioned” into various groups
(i.e., “refereed”, “non-refereed”) by the `partition_pubs` function in
`cvlib.py`. The `PUBLIST` directive causes one of these groups to be output,
with the crucial wrinkle that each record is augmented with a variety of extra
records to allow various special effects, including a lame-brained
`bibtex`-type functionality. This augmentation is done in the `cite_info`
function, which is by far the most complicated part of `cvlib.py`.

If you want to group your publications differently (i.e. “refereed
first-author”), then, you’ll need to edit `partition_pubs`. To change citation
text generation or do more processing, you’ll need to dive into `cite_info`.

The details of publication processing are documented farther below.


Best practices
--------------

To be written.


Technical details: publication records
--------------------------------------

To be written.


Technical details: script invocation
------------------------------------



Technical details: template commands
------------------------------------

To be filled in.

* CITESTATS
* MARKUP (if kept)
* FORMAT
* PUBLIST
* RMISCLIST
* RMISCLIST_IF
* RMISCLIST_IF_NOT
* TODAY.


Technical details: the ini file format
--------------------------------------

To be written.