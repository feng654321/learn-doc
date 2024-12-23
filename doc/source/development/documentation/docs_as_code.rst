*********************
Documentation as Code
*********************

Current situation for RCP Software documentation
################################################

What is Documentation as Code approach?
#######################################

The idea behind this approach is to treat documentation as an intrinsic part of the software source code,
and therefore it should be written and updated with the same tools like below and procedures that developers
use on a daily basis for coding-related activities.

Daily tools for programmer:

- Issue Trackers
- Version Control (Git)
- Coding IDE(VS Code, Eclipse, Vim...)
- Plain Text Markup (Markdown, reStructuredText, Asciidoc)
- Code Reviews
- CI/CD

Way of working with Doc as Code approach
########################################

Write document with plain text
******************************

The most important idea behind of Doc as Code is that document is written with plain text, one advantage of
plain text formats is that they are compatible with all IDEs used by developers, so developer can read and
edit such documents directly from IDE which they prefer as the same way for coding.

Another important advantage of this type of file is that it helps developers to only focus on technical
content, that means the team in charge of writing the documentation will not have to worry about its layout.

Since the documentation is intended to be read by a human audience, it is advisable to adopt a plain text
format that allows using basic formatting. The main formatting features required for a technical paper are:

- Text highlighting (bold, italic, and underlined)
- Hypertext links
- Bullet-ed and Numbered lists
- Code blocks and snippets
- Tables
- Headings

By taking advantage of these simple features, the readability of the content will be significantly improved,
while at the same time, the writing experience will not be overly burdening.


Document version management
***************************

If documents are written with plain text, then it's very easy that developers can use same version control
system(VCS) to manage the documentation, that is the same way for coding, because all our code is written
with plain text format.

Using such VCS tools, developers can easily keep track of all documents changes, knowing by whom they were
made and when with the way they are familiar.

Git has had a great fortune and considerable diffusion thanks to its relative ease of use, and because it
is open source. As a matter of fact, the most worldwide known and used platforms, namely GitHub and GitLab,
are based on Git.

One great advantage offered by these tools is the possibility of working asynchronously with several
people on the same file. Thanks to branches, anyone can create a version of the files on their device, make
changes, and then create a Pull/Merge Request to have the changes implemented. All this without directly
affecting the original file or the other people who are working on other versions of the same file. The
platforms also provide an interface to easily report and resolve conflicts (i.e., two different, competing
edits to the same section of a file).

Publishing document format as you like
**************************************

Once the documentation has been written, version-ed, and saved, it is ready to be published. Then you would
ask what's the good format for users? Here maybe you must think that publishing plain text document with
basic formatting directly to users is not a good idea.

Yes, indeed not a good idea, however we have tools, using a static-site generator (SSG) can easily transform
plain text files into HTML, allows you to add CSS style-sheets to enhance and brand the look and feel, and, if
needed, lets you add dynamic JavaScript sections as well: everything will then be rendered server-side, providing
the client with static pages that are very fast to load.

Thanks to an SSG, you can easily decouple the content and its final graphical representation: the development
team will be responsible only for the technical content, while responsibility for the visual aspect may be given
to another team. This way, developers don't have to worry about selecting the correct font, formatting the content,
or any other such activity: they will only care about adding the basic formatting directly into the plain text file.
Besides, in this way any graphic change can be cascaded to all pages, editing just one style sheet.

Most SSGs also offer several additional features that greatly enhance the usability of the content, but do not
necessarily require manual intervention by the development team. Examples of these additional features include:

- Homepage with links to sub-sections;
- Sidebar navigation menus;
- Search engine;
- Anchor links to headings;
- Table of Contents for each page;
- Release Notes management;
- Managing content in different languages;
- Displaying documentation for previous software versions.

These features are pretty simple in their own right, but together they make it easy to find and consume content,
helping users to quickly find what they're looking for.

Also you can even use other tools to transforms plain text files into other formats like pdf, word etc.

Automation
**********

Another benefit of the Documentation as Code approach is that you can leverage automation with CI/CD (Continuous
Integration and Continuous Delivery) system with same way as code.

Concerning CI, you can create automated tests that check the content each time a Pull/Merge Request is
made on the main branch: such tests can detect technical errors (i.e., invalid links, invalid code snippets,
etc.) or formal errors (i.e., typos, missing punctuation, etc.) within the documentation. If the tests fail,
the team is notified of the error and can take action to resolve it; if the tests pass, the CD phase can begin.

As an example of how you can leverage CD, you will be able to create automation that detects all changes within
the main branch, and automatically the new content is published. This way, you won't have to spend time on the
publishing process, which traditionally takes a lot of time.


Benefit for Documentation as Code
#################################

- Easier to learn for plain text document

  Plain text framework provides basic syntax like heading, Hypertext links, Text highlighting and list etc. these
  syntax are very easy to be used, at same time plain text framework like reStructuredText or Markdown provides nice
  guide for basic syntax, users could be easy to learn from examples.

- Developers write document more quickly and efficiently

  Using the same tools they use every day to write code, developers will be able to focus only on the content,
  producing it faster.

- Better and more accurate documentation

  By being faster in the writing phase, developers will produce more accurate documentation, because the writing
  phase will occur shortly after the development of the functionality they want to document.
  Also if the software document is in same repository with code, the software change and document change can be
  in same merge request, then it is easier for reviewers to find out miss-matches between software and document
  and give comments timely.

- Format of the published documents are more flexible

  Documentation as Code approach decouple of contents and graphics, developers only are responsible for the
  accuracy of the content, they don't need care about the final document form, there are lots of tools could
  be used to transform plain text to different nice face format as you need like SSG, and such tools could be
  easy to integrate into CI/CD system to leverage automation of document publishing.

- Full version control

  With Documentation as Code approach, you'll be able to easily track every change to the documentation, and in
  case of error you'll be able to easily roll back. Furthermore, version also lets you check that the
  documentation is up-to-date.

- Easier collaboration

  With Documentation as Code approach, these used tools are designed to encourage collaboration, and by being very
  familiar with them, it will be easier for developers to collaborate and they will be more inclined to suggest
  changes and improvements.

Tools usage for Documentation as Code
#####################################

Plain text documentation tools
******************************

Markdown
--------

Markdown is a lightweight markup language that you can use to add formatting elements to plaintext text documents.
Created by John Gruber in 2004, Markdown is now one of the world's most popular markup languages.

Please click `Get start for Markdown`_.

.. _Get start for Markdown: https://www.markdownguide.org/getting-started/

Markdown tools
--------------

You could find all relate tools from `Tools`_

.. _Tools: https://www.markdownguide.org/tools/


Markdown in VS Code
-------------------

You could find relate plugin in VS Code marketplace, `MarkDown all in one`_ is strongly recommended
as editor and previewer for developers.

`Markdown Preview Enhanced`_ is another strongly recommended useful VS Code extension which can
directly preview review plantUML scripts as diagram.

.. _MarkDown all in one: https://marketplace.visualstudio.com/items?itemName=yzhang.markdown-all-in-one
.. _Markdown Preview Enhanced: https://marketplace.visualstudio.com/items?itemName=jebbs.plantuml


Markdown best practices
-----------------------

You could find the best practices from `markdown best practices`_

.. _markdown best practices: https://docs.ext.net.nokia.com/csf/bp/markdown/latest/index.html

Restructure Text
----------------

reStructuredText is an easy-to-read, what-you-see-is-what-you-get plaintext markup syntax and parser system.
It is useful for in-line program documentation (such as Python docstrings), for quickly creating simple web
pages, and for standalone documents.

- `Step 1: Getting started with RST <https://sphinx-tutorial.readthedocs.io/step-1/>`_
- `reStructuredText User Documentation <https://docutils.sourceforge.io/rst.html>`_
- `reStructuredText (RST) Tutorial <https://www.devdungeon.com/content/restructuredtext-rst-tutorial-0>`_
- `rst-cheatsheet <https://github.com/ralsina/rst-cheatsheet/blob/master/rst-cheatsheet.rst>`_

Restructure Text tools
----------------------

- `Sphinx`_
- `Sphinx template`_

.. _Sphinx: https://www.sphinx-doc.org
.. _Sphinx template: https://gitlabe2.ext.net.nokia.com/rcp/templates/sphinx-template

Restructure Text in VS Code
---------------------------

`reStructuredText`_ extension provides rich reStructuredText language support for Visual Studio
Code, you can preview your reStructuredText file via it.

`reStructuredText Syntax highlighting`_ extension provides rich syntax highlighting and
non-intrusive section navigation for reStructuredText.

.. _reStructuredText: https://marketplace.visualstudio.com/items?itemName=lextudio.restructuredtext

.. _reStructuredText Syntax highlighting: https://marketplace.visualstudio.com/items?itemName=trond-snekvik.simple-rst

MarkDown vs Restructure Text
----------------------------

Diagram as code
***************

The idea behind of diagram as code is that you could generate a diagram from plain text with some basic syntax.
if you want draw some pictures in your document, then you just write piece of plain text content in document,
after your document is published, such plain text content will be generate to real image in document with behind
tools.

Usually, once you provided a picture (like jpg, png etc.) of your diagram directly in document, other ones can't
change it any more unless they re-draw it and replace your's in this document if they want to do some updating.
Also with this way you can't do version control for such picture.

Diagram as code also has some benefits:
- Designers and developers needn't to care of diagram format
- Full version control together with document
- Easy to change and maintain
- Change could be tracked

PlantUML
--------

PlantUML is an open-source tool allowing users to create diagrams from a plain text language. Besides various UML
diagrams, PlantUML has support for various other software development related formats (such as Archimate, Block
diagram, BPMN, C4, Computer network diagram, ERD, Gantt chart, Mind map, and WBD), as well as visualization of
JSON and YAML files.

The language of PlantUML is an example of a domain-specific language. Besides its own DSL, PlantUML also
understands AsciiMath, Creole, DOT, and LaTeX. It uses Graphviz software to layout its diagrams and Tikz for
LaTeX support. Images can be output as PNG, SVG, LaTeX and even ASCII art. PlantUML has also been used to allow
blind people to design and read UML diagrams.

Please click `Get start for PlantUML`_

.. _Get start for PlantUML: https://plantuml.com/

Please click `Guideline for using plantUML via VS code and some useful tips`_

.. _Guideline for using plantUML via VS code and some useful tips: https://confluence.ext.net.nokia.com/display/RCP/Guideline+for+using+plantUML+via+VS+code+and+some+useful+tips

Draw.io
-------

SVG is a vector image format used for creating 2D graphics and animations using XML, and Draw.io is
a web-based tool that supports working with SVG files and includes a library of scalable icons.

Although PlantUML is efficient, it has a certain learning curve. In VS Code, you can manually draw
diagrams using `Draw.io Integration`_ without much difficulty.

Of course, you can also directly access this official link for `Draw.io`_ in your browser to do the
same drawing work.

.. _Draw.io Integration: https://marketplace.visualstudio.com/items?itemName=hediet.vscode-drawio
.. _Draw.io: https://app.diagrams.net/

Mermaid
-------

Mermaid is a similar tool like PlantUML, but written in JavaScript, it renders MarkDown-inspired text definition
to create and modify diagrams dynamically. in terms of feature it is less powerful than PlantUML.

Please click `Get start for Mermaid`_

.. _Get start for Mermaid: https://mermaid-js.github.io/mermaid/

PlantUML vs Mermaid
-------------------


API documentation
*****************

Doxygen
-------

Doxygen is the de facto standard tool for generating documentation from annotated C++ sources, but it also supports
other popular programming languages such as C, Objective-C, C#, PHP, Java, Python, IDL (Corba, Microsoft, and
UNO/OpenOffice flavors), Fortran, and to some extent D. Doxygen also supports the hardware description language
VHDL.

Please click `Get start for Doxygen`_

.. _Get start for Doxygen: https://doxygen.nl/

Doxygen in VS Code
------------------

`Doxygen Documentation Generator`_ is a extension provides Doxygen Documentation generation on the
fly by starting a Doxygen comment block and pressing enter.

.. _Doxygen Documentation Generator: https://marketplace.visualstudio.com/items?itemName=cschlosser.doxdocgen

OpenAPI
-------


Spell Checking
**************

Spell checking is a feature provided in integrated development environments (IDEs) which
automatically checks and corrects spelling errors. The advantages of spell checking in IDEs include
error detection and correction, improved readability, increased efficiency, reduced maintenance
costs, and support for multiple languages.

Spell checking in VS Code
-------------------------

For native VS Code spell checking, you can refer to :ref:`Spell Checking With VSCode Guidelines`.
Here we strongly recommend you try out `LTeX - LanguageTool grammar/spell checking`_.

LTeX provides offline grammar checking of various markup languages in Visual Studio Code using
LanguageTool (LT). LTeX currently supports BibTEX, ConTEXt, LATEX, Markdown, Org, reStructuredText,
R Sweave, and XHTML documents. In addition, LTEX can check comments in many popular programming
languages (optional, opt-in).

The difference to regular spell checkers is that **LTEX not only detects spelling errors, but also
many grammar and stylistic errors**.

.. _LTeX - LanguageTool grammar/spell checking: https://marketplace.visualstudio.com/items?itemName=valentjn.vscode-ltex

Document Column Length
######################

When writing documentation with markup language, it's important to ensure that
your code are easy to read and understand. Managing column length is a key
aspect of maintaining readability.

Limit Line Length
*****************

It's recommended to limit the line length to around 80 characters. This ensures
that the document remains visible and readable even on smaller screens or when
printed. Longer lines can be difficult to follow and may require horizontal
scrolling.

Consistent line lengths contribute to improved readability, making it easier for
you and your collaborators to understand the content at a glance.

Rewrap VS Code plugin
---------------------

`Rewrap VS Code plugin`_ offers a lot of benefits that enhance your text
formatting journey. From saving time and increasing productivity to improving
document readability and enforcing coding standards, Rewrap simplifies the way
you work with text, code, and documentation.

.. _Rewrap VS Code plugin: https://marketplace.visualstudio.com/items?itemName=stkb.rewrap

VS Code's Code Snippet
######################

VS Code's Code Snippet is a shortcut that allows developers to quickly input commonly used code
blocks. It can be thought of as a "template" for any reusable code block. By defining a code
snippet, developers can automatically insert a pre-defined code segment when typing a specific
abbreviation, saving time and reducing typing errors. It also provides the ability to parameterize
code blocks, allowing users to customize values in a code block by filling in specific fields. Code
Snippets is one of the key features of VS Code's activation master, allowing developers to work more
efficiently in their daily development tasks.

Absolutely! With the Code Snippet feature in VS Code, developers can define a range of reusable code
snippets for various purposes such as Markdown, reStructuredText, PlantUML, and more. This allows
the developers to quickly input commonly used code blocks, organize their documentation more
effectively, and reduce typing errors. Additionally, developers can customize the code snippets
further by parameterizing them for specific use cases, which makes their documentation even more
efficient and useful. By leveraging this feature correctly, developers can significantly improve
their documentation workflow and productivity.

Code Snippet in VS Code
***********************

`Easy Snippet`_ is an extension given an easy way to manage snippet.

.. _Easy Snippet: https://marketplace.visualstudio.com/items?itemName=inu1255.easy-snippet

Reference
#########

`Documentation as Code, how does it improve developer experience`_

`Documentation as code guideline`_

.. _Documentation as Code, how does it improve developer experience: https://www.cncf.io/blog/2022/04/25/docs-as-code-how-does-it-improve-developer-experience/
.. _Documentation as code guideline: https://www.writethedocs.org/guide/docs-as-code/
