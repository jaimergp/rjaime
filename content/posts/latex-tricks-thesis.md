---
title: "(Non-)LaTeX tricks I used while I was writing my thesis with LaTeX"
date: 2018-10-05T18:10:52+02:00
---

Like many, I wrote my PhD thesis using LaTeX. I had used LaTeX before, but not with such a big manuscript. I learnt a few tricks during that process that I think somebody might find useful.

<!--more-->

---

On September 20th, I defended my PhD in Biotechnology against a 3-member jury. All of them had a physical copy of that carefully typeset manuscript that I had been writing and adjusting for weeks with perfectly crafted LaTeX code. Or so it seems in paper. The innards of all those `.tex` documents are scary. Trust me, you can check them in [this GitHub repo](https://github.com/jaimergp/phd-biotechnology-thesis).

My experience is that LaTeX is both powerful and fragile. Obtaining good results is a matter of choosing tens of packages and patches, which can generate confusing results if you do not know what is going on. I still don't know what is going on, and my thesis has a strong focus on software development.

After a while I stopped caring about the (un)elegance of my LaTeX project because I had a deadline to meet, so I ended up collecting a few hacks & tricks that I found during long debugging sessions. Most of them are part of the `.tex` document, so you can just go the [repo](https://github.com/jaimergp/phd-biotechnology-thesis) and read the code. I tried to comment the changes I did to the original template (the excellent [Dissertate](https://github.com/suchow/Dissertate)) for further reference.

However, I also had to resort to non-LaTeX tricks to obtain the desired results.  These are tested in Linux only, but it might work in MacOS and Windows as long as you have the adequate software installed. I hope you find them useful!


## Sharp and clear figures from PowerPoint

I designed a lot of the graphical material using PowerPoint slides. Yes, there are better tools like Inkscape or Illustrator, but I felt that for some cases PowerPoint fits the bill just perfect.

However, obtaining a high-resolution image out that slide is not easy. PowerPoint does support lossless PNG exports, but the resolution is limited to [~300 DPI](https://support.microsoft.com/en-us/help/827745/how-to-change-the-export-resolution-of-a-powerpoint-slide), which is usually not enough for text in figures. Fortunately, it does allow to export PDF documents, which keep the vector graphics used for shapes and text. Then, all you have to do is crop the exported PDF slide to the meaningful part with `pdfcrop` and you are good to go!

## Reduce the size of the generated PDF document

A large manuscript such as a thesis will end up containing lots of high-resolution images which can end up summing hundreds of megabytes. Top-quality images are needed in press so there is no way around that, but if you are just trying to share an online copy of the dissertation (for internal review among your colleagues or something like that) nobody will want to download a 700MB PDF.

You can use this Bash function (inspired in [this answer](https://tex.stackexchange.com/a/19047)) to reduce the file size drastically.

```
compressPDF() {
    gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 -dNOPAUSE -dQUIET -dBATCH -sOutputFile=${1%.pdf}_compressed.pdf $1
}
```

Add it to your `.bashrc` and you can use it like this: `compressPDF thesis.pdf`. It will output a new copy called `thesis_compressed.pdf`. In my case, the size went down from 41 to 15MB.

## The nightmare of citations

One of the more time-consuming tasks I had to do during the writing was to review all my BibTeX citations so the author names, journal abbreviations and other details were accurate. It was so painful that I decided to automate it as much as possible and ended up creating a Python tool to help in the process: [fixbibtex](https://github.com/jaimergp/fixbibtex). It is not perfect, but it can help complete missing metadata if the original entry is good enough.

However, I would not recommend resorting to this approach. Even with the support of CrossRef behind the scenes, you will have to manually amend some entries anyway. My advice is to start fixing entries earlier.

By earlier I mean *as soon as you save it in your `.bib` file*. To do that, the best way is to retrieve the DOI of the article and query the [doi.org](https://doi.org) site.

This Bash function (inspired in [this one](https://users.aalto.fi/~mkouhia/2016/bibtex-from-dx-doi-org/)) will return BibTeX citations for every DOI passed (just the DOI, no prefixes nor URLs).

```
doi2bib() {
    for arg in $@; do
        curl -LH "Accept: application/x-bibtex" "http://dx.doi.org/$arg"
        echo
    done
}

```

Paste it in your `.bashrc` and the call `doi2bib doi1 [doi2] [doi3] [...]`. For example:

```
$> doi2bib 10.1002/jcc.24847 10.1039/C7SC01296A 10.1021/acs.jcim.7b00714
@article{Rodr_guez_Guerra_Pedregal_2017,
	doi = {10.1002/jcc.24847},
	url = {https://doi.org/10.1002%2Fjcc.24847},
	year = 2017,
	month = {jun},
	publisher = {Wiley},
	volume = {38},
	number = {24},
	pages = {2118--2126},
	author = {Jaime Rodríguez-Guerra Pedregal and Giuseppe Sciortino and Jordi Guasp and Martí Municoy and Jean-Didier Maréchal},
	title = {{GaudiMM}: A modular multi-objective platform for molecular modeling},
	journal = {Journal of Computational Chemistry}
}
@article{Mujika_2017,
	doi = {10.1039/c7sc01296a},
	url = {https://doi.org/10.1039%2Fc7sc01296a},
	year = 2017,
	publisher = {Royal Society of Chemistry ({RSC})},
	volume = {8},
	number = {7},
	pages = {5041--5049},
	author = {Jon I. Mujika and Jaime Rodríguez-Guerra Pedregal and Xabier Lopez and Jesus M. Ugalde and Luis Rodríguez-Santiago and Mariona Sodupe and Jean-Didier Maréchal},
	title = {Elucidating the 3D structures of Al(iii){\textendash}A$\upbeta$ complexes: a template free strategy based on the pre-organization hypothesis},
	journal = {Chemical Science}
}
@article{Rodr_guez_Guerra_Pedregal_2018,
	doi = {10.1021/acs.jcim.7b00714},
	url = {https://doi.org/10.1021%2Facs.jcim.7b00714},
	year = 2018,
	month = {mar},
	publisher = {American Chemical Society ({ACS})},
	volume = {58},
	number = {3},
	pages = {561--564},
	author = {Jaime Rodríguez-Guerra Pedregal and Pablo Gómez-Orellana and Jean-Didier Maréchal},
	title = {{ESIgen}: Electronic Supporting Information Generator for Computational Chemistry Publications},
	journal = {Journal of Chemical Information and Modeling}
}

```

Voilà! Ready for inclusion in your `.bib` file. But remember, review each entry first!

## Find help online


All my duels with LaTeX were solved by searching online for my problem. In that matter, [TeX Stack Exchange](https://tex.stackexchange.com) and [latex.org](https://latex.org/forum) are invaluable resources. Search there directly if Google is not helping.