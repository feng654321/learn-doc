*************************
Spell Checking With Vim
*************************

Introduction
############

`Vim` supports spell checking. Use `:set spell` to turn spell check on and
`:set nospell` to turn spell check off. If you want always the spell checker
to be on, add `set spell` to your `.vimrc`.


Quick Guide
###########

When spell checker is on, misspelled words will be highlighted. Type `]s` to
move to next misspelled word and `[s` to move to previous one. To see a list
of corrected spells, place the cursor on a misspelled word and type "z=". You
can type the number of the word you wish to replace the misspelled word with
and hit <enter> to replace it, or you can just hit enter to leave the word
unchanged. With the cursor on a misspelled word, you can also type "<number>z="
to change to the <number>th correction without viewing the list. Typically you
will use "1z=" if you think vim's first guess is likely to be the correct word.
To set the word list that "vim" will use for spelling checking set the "spelllang"
option. e.g. ":set spelllang=en", ":set spelling=en_us", ":setlocal spell spelllang=<language>"

Sometimes you want spell check to skip some words e.g. you created. In this case,
you could create your own spell file. And use commands to add word there.


.. code-block::

 $ mkdir -p $HOME/.vim/spell
 $ touch $HOME/.vim/spell/en.utf-8.add
 $ # in vim
 $ :set spellfile="~/.vim/spell/en.utf-8.add"

.. code-block::

 zg      Add word under the cursor as a good word to the first
         name in 'spellfile'.  A count may precede the command
         to indicate the entry in 'spellfile' to be used.  A
         count of two uses the second entry.

         In Visual mode the selected characters are added as a
         word (including white space!).
         When the cursor is on text that is marked as badly
         spelled then the marked text is used.
         Otherwise the word under the cursor, separated by
         non-word characters, is used.

         If the word is explicitly marked as bad word in
         another spell file the result is unpredictable.


 zG       Like "zg" but add the word to the internal word list
          |internal-wordlist|.
 
 
 zw       Like "zg" but mark the word as a wrong (bad) word.
          If the word already appears in 'spellfile' it is
          turned into a comment line.  See |spellfile-cleanup|
          for getting rid of those.
 
 
 zW       Like "zw" but add the word to the internal word list
          |internal-wordlist|.
 
 zuw
 zug      Undo |zw| and |zg|, remove the word from the entry in
          'spellfile'.  Count used as with |zg|.
 
 zuW
 zuG      Undo |zW| and |zG|, remove the word from the internal
          word list.  Count used as with |zg|.

There are also some Vim plugins for spell checker like follows. These
normally provides some extended features or different syntax. Check these
when you see that is valuable.

* https://github.com/kamykn/spelunker.vim
* https://github.com/inkarkat/vim-SpellCheck
* https://github.com/dbmrq/vim-dialect

