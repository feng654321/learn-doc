*************************
Spell Checking With Emacs
*************************

Introduction
############

`Emacs <https://www.gnu.org/software/emacs/>`_ supports spell checking
*out-of-the-box* if a spelling checker program, one of Hunspell, Aspell,
Ispell or Enchant, is installed.

Installing hunspell in Fedora:
::

 sudo dnf install hunspell-en-US

Manuals
#######

`Checking and Correcting Spelling <https://www.gnu.org/software/emacs/manual/html_node/emacs/Spelling.html>`_
contains detailed instructions for using all the Emacs spell checking and
correcting features.

`Spell Checking in Emacs <https://www.tenderisthebyte.com/blog/2019/06/09/spell-checking-emacs/>`_
shows how to use
`Flyspell <https://www.emacswiki.org/emacs/FlySpell>`_ for *on-the-fly*
spelling.

Quick Guide
###########

Check and correct spelling in the buffer:
::

 M-x ispell-buffer

Check and correct spelling in the region:
::

 M-x ispell-region

Check and correct spelling of comments and strings in the buffer or region:
::

 M-x ispell-comments-and-strings

Enable Flyspell mode, which highlights all misspelled words:
::

 M-x flyspell-mode

Enable Flyspell mode for comments and strings only:
::

 M-x flyspell-prog-mode
