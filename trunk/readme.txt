
 Experiments with inputenc
===========================


This is an experimental extension of inputenc for Unicode TeXs.  It is
called xinputenc here for LPPL reasons and to allow (with care) side
by sode comparison with the current inputenc. (Most of the .def files
have the smae names when generated, so you need to note input paths
when doing such a comparison.

It is _not_ intended (ever) for public release, it is just checked in
to a public source control system for safety.


inputenc doesn't really work with lulatex/xelatex, the 2014 release
improved things by giving a clearer warning or error, but this tries
to make things work, taking ideas from luainputenc and xetex-inputenc.



* luainputenc
=============

luainputenc (Manuel Pegourie-Gonnard and Elie Roux)

Works quite well as an inputenc replacement for lualatex but it is
currently unmaintained and it pulls in a lot of other packages which
is fine for a contributed package but perhaps not if this were to be
pulled in to the core.

lualatex \\RequirePackage{luainputenc} \\stop 

produces

(...tex/lualatex/luainputenc/luainputenc.sty
(...tex/generic/oberdiek/ifluatex.sty)
(...tex/generic/ifxetex/ifxetex.sty)
(...tex/luatex/luatexbase/luatexbase.sty
(...tex/generic/oberdiek/luatex.sty
(...tex/generic/oberdiek/infwarerr.sty)
(...tex/latex/etex-pkg/etex.sty)
(...tex/generic/oberdiek/luatex-loader.sty
(...scripts/oberdiek/oberdiek.luatex.lua)))
(...tex/luatex/luatexbase/luatexbase-compat.sty)
(...tex/luatex/luatexbase/luatexbase-modutils.sty
(...tex/luatex/luatexbase/luatexbase-loader.sty
(...tex/luatex/luatexbase/luatexbase.loader.lua))
(...tex/luatex/luatexbase/modutils.lua))
(...tex/luatex/luatexbase/luatexbase-regs.sty)
(...tex/luatex/luatexbase/luatexbase-attr.sty
(...tex/luatex/luatexbase/attr.lua))
(...tex/luatex/luatexbase/luatexbase-cctb.sty
(...tex/luatex/luatexbase/cctb.lua))
(...tex/luatex/luatexbase/luatexbase-mcb.sty
(...tex/luatex/luatexbase/mcb.lua)))
(...tex/lualatex/luainputenc/luainputenc.lua))


The largest part of this additional code is to handle the luatex base
module loading system. The current luatex support removes these
dependencies by using the primitive \directlua and callback.register
directly. (This may be a simplification too far)

Hoever the code to enable 8bit processing used here is copied directly
from luainputenc.


luainputenc has two mode (actually three)

8bit mode: lua code is used to process the input 1 byte at a time
           active characters are used as in clasic TeX to implemnt the
           encodings. 

unactive mode: full utf8 input is accepted by the file reading, fonts
             are assumed to be Unicode. Active characters are no used.

mixed mode: switches between these depending on the font encoding.

In luainputenc lua rather than TeX is mainly used to switch catcodes.


In this version I have just kept the lua code to enable 8bit mode,
removing the module loader and the catcode changing features. The
handling of utf8 input in 8bit mode is modified so that hopefully a
third mixed mode is not needed.  the unactivate option name is copied
from this package to enable the unactive mode.



* xetex-inputenc
================
xetex-inputenc (Will Robertson) was never distributed to ctan but
aimed to be an inputenc replacement for xetex.

It uses the \XeTeXinputencoding to get xetex to directly interpret
incoming bytes in the specified encoding. (xetex supports dozens of
encodings) No active characters are used. As a result all inpt is seen
to teh macro layer as full Uniocde input and effectively works as if
it was utf-8 input to xetex, without the package. In particular it
more or less mandates the use of Unicode fonts as there is no mapping
to traditional 8 or 7 bit font ranges.







* xinputenc
============

While investigating updating inputenc for the 2014/05/01 release and
in particular commenst on the preview of that release, I was looking
at this again in detail and needed to make code and record thoughts in
order that I might remember next time this comes up. this package is
_not_ stable or intended for use on anything but the test documents
contained in the distributiion.

Plan:

2 modes

default "8 bit" mode

  should be compatible with the traditional use of inputenc with
  (pdf)latex.  

  All input is read 1 byte=1c haracter (with the ssame character code)
  No characters higer than hex FF are seen in the input stream
  All mapping done via active characters.

  This should mean that most existing documents using inputenc "work
  out of the box" with (xe/lua)latex although many of the features of
  those engines will be effectively disabled.

  8bit input is enabled in luatex with some lua taken from luainputenc
  and in xetex via \XeTeXinputencoding "iso-8859-1" (which is a bit of
  an abuse of that encoding, but whatever)


Unicode/unactivate mode

   enabled with explict unactivate option 
   (possibly should be enabled by default with some combinations of
   engine and encoding)

   No active characters are used.

   file stream is parsed by the engine to return unicode characters
   (just 0-FF in the case of classic TeX)

   Currently in lualatex  this means input must be utf-8, although one
   could in principle emulate more of \XeTeXinputencoding behaviour
   with more lua (I think). In xetex it could, but the code doesn't
   currently use \XeTeXinputencoding with any supported encoding.


* Test files
============

   A range of test files with different input and font encodings is
   supplied. The (x)inputenc use is _not_ guarded by any tests on the
   engine in use, however tests using Unicode fonts (currently just
   Arial) have a guard selecting fontenc for pdftex, so that there is
   some fallback. Wit this all the 8bit tests should work on all
   engines. The unactve versions mostly don't work yest (and probably
   can't work in pdftex, they should however give sensible error
   messages.


* Issues
========

  If using utf8 input and Unicode fonts it isn't really feasible to
  have ASCII LICR representations of every character, you really want
  to be able to pass the character through rather than raise an error.
  luainputenc did this by special casing EU2 font encoding and
  turning off the active character definitions, but switching catcodes
  mid document, even via lua has problems.
  here (in 8bit mode) I let the macro layer re-construct the utf-8
  representation as in inputenc but then if the font encoding is EU1/2
  instead of making an error I generate that character.

  luatex has the expandable primitive \(luatex)Uchar which constructs
  the character from its number.

  in xetex I am currently using \char but that is not expandable so
  will mess up ligatures etc (I think, not sure how this works in
  xetex). It's possible to generate things like ^^^410 (for the
  cyrillic examples) and the code for that is there, but It's not
  clear that that can be used without \scantokens so ens up again not
  being expandable.


  unactive mode is mostly unimplemented and just a sketch.

  For latin1 input and Uniocde fonts in 8bit mode accented characters
  go via LICR such as \'{e} and so work being mapped back to EU2
  encoding.  For cyrillic though as shown in test-cp1251-eu2.tex
  The input gets mapped to things like \CYRA which would be defined if
  X2 or OT2 had been loaded but are not defined for EU2 so currently
  that file needs a helper file eu2x2enc.def which is a sort of subset
  eu2enc.def covering the X2 range. It is basically an inverse of
  x2enc.dfu other than complication over the way accented characters
  need to be declared (which isn't done yet, those lines are commented
  out.

  \XeTeXinputencoding is not currently used at all other than for
  latin1 to enable 8bit processing. It should probably be used to
  enable multiple encodings in unactive mode. If we wanted to make it
  work in luatex as well, I think it would need extending

    function luainputenc.byte_to_utf(ch)
      return utfchar(byte(ch))
    end

  so that it used mapping tables for supported encodings (there may be
  an existing lua library for this?)
  Or we could just support this on xelatex but I'm not sure the
  feature is useful enough to introduce incompatobilities.

  The code is only tested on the supplied files, not with real
  documents and not in combination with any pther packages.

  None of the current test files use auxiliary files. Once these are
  working we need to test aux toc ind bbl etc.

