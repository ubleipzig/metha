# metha

> The Open Archives Initiative Protocol for Metadata Harvesting (OAI-PMH) is a
> low-barrier mechanism for repository interoperability. Data Providers are
> repositories that expose structured metadata via OAI-PMH. Service Providers
> then make OAI-PMH service requests to harvest that metadata. -- https://www.openarchives.org/pmh/

The metha command line tools can gather information on OAI-PMH endpoints and
harvest data incrementally. The goal of metha is to make it simple to get
access to data, its focus is not to manage it.

[![DOI](https://zenodo.org/badge/56384577.svg)](https://zenodo.org/badge/latestdoi/56384577) [![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)

The metha tool has been developed for [project finc](https://finc.info) at
[Leipzig University Library](https://ub.uni-Leipzig.de) ([lab](https://lab.ub.uni-leipzig.de/metha/)).

## Why yet another OAI harvester?

* I wanted to crawl [Arxiv](http://export.arxiv.org/oai2) but found that existing tools would timeout.
* Some harvesters would start to download all records anew, if I interrupted a running harvest.
* There are many OAI
  [endpoints](https://raw.githubusercontent.com/miku/metha/master/contrib/sites.tsv) out
  there. It is a widely used
  [protocol](http://www.openarchives.org/OAI/openarchivesprotocol.html) and
  somewhat worth knowing.
* I wanted something simple for the command line; also fast and robust - metha
  as it is implemented now, is relatively robust and more efficient than
  requesting all record one-by-one (there is one
  [annoyance](https://github.com/miku/metha/issues/6) which will hopefully be
  fixed soon).

## How it works

The functionality is spread accross a few different executables:

* metha-sync for harvesting
* metha-cat for viewing
* metha-id for gathering data about endpoints
* metha-ls for inspecting the local cache
* metha-files for listing the associated files for a harvest

To harvest and endpoint in the default *oai_dc* format:

```sh
$ metha-sync http://export.arxiv.org/oai2
...
```

All downloaded files are written to a directory below a base directory. The base
directory is `~/.cache/metha` by default and can be adjusted with the `METHA_DIR`
environment variable.

When the `-dir` flag is set, only the directory corresponding to a harvest is printed.

```
$ metha-sync -dir http://export.arxiv.org/oai2
/home/miku/.metha/I29haV9kYyNodHRwOi8vZXhwb3J0LmFyeGl2Lm9yZy9vYWky
```

```sh
$ METHA_DIR=/tmp/harvest metha-sync -dir http://export.arxiv.org/oai2
/tmp/harvest/I29haV9kYyNodHRwOi8vZXhwb3J0LmFyeGl2Lm9yZy9vYWky
```

The harvesting can be interrupted at any time and the HTTP client will
automatically retry failed requests a few times before giving up.

Currently, there is a limitation which only allows to harvest data up to the
last day. Example: If the current date would be *Thu Apr 21 14:28:10 CEST
2016*, the harvester would request all data since the repositories earliest
date and *2016-04-20 23:59:59*.

To stream the harvested XML data to stdout run:

```sh
$ metha-cat http://export.arxiv.org/oai2
```

You can emit records based on datestamp as well:

```sh
$ metha-cat -from 2016-01-01 http://export.arxiv.org/oai2
```

This will only stream records with a datestamp equal or after 2016-01-01.

To just stream all data really fast, use `find` and `zcat` over the harvesting
directory.

```sh
$ find $(metha-sync -dir http://export.arxiv.org/oai2) -name "*gz" | xargs unpigz -c
```

To display basic repository information:

```sh
$ metha-id http://export.arxiv.org/oai2
```

To list all harvested endpoints:

```sh
$ metha-ls
```

Further examples can be found in the metha [man page](https://github.com/miku/metha/blob/master/docs/metha.md):

```
$ man metha
```

## Installation

Use a deb, rpm [release](https://github.com/miku/metha/releases), or the go tool:

```sh
$ go install -v github.com/miku/metha/cmd/...@latest
```

## Limitations

Currently the endpoint URL, the format and the set are concatenated and base64
encoded to form the target directory, e.g:

```
$ echo "U291bmRzI29haV9kYyNodHRwOi8vY29wYWMuamlzYy5hYy51ay9vYWktcG1o" | base64 -d
Sounds#oai_dc#http://copac.jisc.ac.uk/oai-pmh
```

If you have very long set names or a very long URL and the target directory
exceeds e.g. 255 chars (on ext4), the harvest won't work.

## Harvesting Roulette

```sh
$ URL=$(shuf -n 1 <(curl -Lsf https://git.io/vKXFv)); metha-sync $URL && metha-cat $URL
```

In 0.1.27 a `metha-fortune` command was added, which fetches a random article
description and displays it.

```shell
$ metha-fortune
Active Networking is concerned with the rapid definition and deployment of
innovative, but reliable and robust, networking services. Towards this end we
have developed a composite protocol and networking services architecture that
encourages re-use of protocol functions, is well defined, and facilitates
automatic checking of interfaces and protocol component properties. The
architecture has been used to implement common Internet protocols and services.
We will report on this work at the workshop.

    -- http://drops.dagstuhl.de/opus/phpoai/oai2.php

$ metha-fortune
In this paper we show that the Lempert property (i.e., the equality between the
Lempert function and the Carathéodory distance) holds in the tetrablock, a
bounded hyperconvex domain which is not biholomorphic to a convex domain. The
question whether such an equality holds was posed by Abouhajar et al. in J.
Geom. Anal. 17(4), 717–750 (2007).

    -- http://ruj.uj.edu.pl/oai/request

$ metha-fortune
I argue that Gödel's incompleteness theorem is much easier to understand when
thought of in terms of computers, and describe the writing of a computer
program which generates the undecidable Gödel sentence.

    -- http://quantropy.org/cgi/oai2

$ metha-fortune
Nigeria, a country in West Africa, sits on the Atlantic coast with a land area
of approximately 90 million hectares and a population of more than 140 million
people. The southern part of the country falls within the tropical rainforest
which has now been largely depleted and is in dire need of reforestation. About
10 percent of the land area was constituted into forest reserves for purposes
of conservation but this has suffered perturbations over the years to the
extent that what remains of the constituted forest reserves currently is less
than 4 percent of the country land area. As at today about 382,000 ha have been
reforested with indigenous and exotic species representing about 4 percent of
the remaining forest estate. Regrettably, funding of the Forestry sector in
Nigeria has been critically low, rendering reforestation programme near
impossible, especially in the last two decades. To revive the forestry sector
government at all levels must re-strategize and involve the local communities
as co-managers of the forest estates in order to create mutual dependence and
interaction in resource conservation.

    -- http://journal.reforestationchallenges.org/index.php/REFOR/oai
```

## Scrape all metadata in a best-effort way

Use an endless loop with a timeout to get out of any hanging connection (which
happen). Example scrape, converted to JSON (326M records, 60+ GB:
[2023-11-01-metha-oai.ndjson.zst](https://archive.org/download/oai_harvest_2023-11-01/2023-11-01-metha-oai.ndjson.zst)).

```shell
$ while true; do \
    timeout 120 metha-sync -list | \
    shuf | \
    parallel -j 64 -I {} "metha-sync -base-dir ~/.cache/metha {}"; \
done
```

Alternatively, use a
[metha.service](https://raw.githubusercontent.com/miku/metha/master/extra/linux/metha.service)
file to run harvests continuously.

metha stores harvested data in one file per interval; to combine all XML files
into a single JSON file you can utilize the
[xmlstream.go](https://github.com/miku/metha/blob/master/extra/largecrawl/xmlstream.go) (adjust the harvest directory):

```shell
$ fd . '/data/.cache/metha' -e xml.gz | parallel unpigz -c | xmlstream -D
```

For notes on parallel processing of XML see: [Faster XML processing in Go](https://golangleipzig.space/posts/faster-go-xml-processing/).

![](docs/metha-net-zenith.png)

## Errors this harvester can somewhat handle

* responses with resumption tokens that lead to empty responses
* gzipped responses, that are not advertised as such
* funny (illegal) control characters in XML responses
* repositories, that won't respond unless the dates are given with the exact granualarity
* repositories with endless token loops
* repositories that do not support selective harvesting, use `-no-intervals` flag
* limited repositories, metha will try a few times with an exponential backoff
* repositories, which throw occasional HTTP errors, although most of the responses look good, use `-ignore-http-errors` flag

## Authors

* Martin Czygan <martin.czygan@uni-leipzig.de>
* Natanael Arndt, [https://github.com/white-gecko](https://github.com/white-gecko)
* Gunnar Þór Magnússon, [https://github.com/gunnihinn](https://github.com/gunnihinn)
* Thomas Gersch, [https://github.com/titabo2k](https://github.com/titabo2k)
* [Andreas Czerniak](https://github.com/ACz-UniBi)
* [David Glück](https://github.com/dvglc)
* [Justin Kelly](https://github.com/justinkelly)

## Misc

Show formats of random repository:

```shell
$ shuf -n 1 <(curl -Lsf https://git.io/vKXFv) | xargs -I {} metha-id {} | jq .formats
```

A snippet from a 2010 publication:

> The Open Archives Protocol for Metadata Harvesting
(OAI-PMH) (Lagoze and van de Sompel, 2002) is currently implemented by more
than 1,700 digital library reposi- tories world-wide and enables the exchange
of metadata via HTTP. -- [Interweaving OAI-PMH Data Sources with the Linked Data Cloud](http://eprints.cs.univie.ac.at/73/1/ijmso2010_haslhofer_schandl.pdf)

## Metha elsewhere

* [The finc project](https://finc.info/de/datenquellen)
* [UB LEIPZIG LAB](https://lab.ub.uni-leipzig.de/metha/)
* [Getting a dump of arXiv metadata](https://academia.stackexchange.com/questions/38969/getting-a-dump-of-arxiv-metadata) at [academia.stackexchange.com](https://academia.stackexchange.com/)
* [Keyword Extraction from arXiv - Part 1](http://web.archive.org/web/20191111162743/http://akumano.site/posts/arxiv-keyword-extraction-part1/)
* [Openrefine use case: Automated workflow for harvesting, transforming and indexing of bibliographic metadata](https://groups.google.com/forum/#!topic/openrefine/RqQwlF-ll1c)
* [Sammeln und Finden. Über das Sichtbarmachen von Open Science in Hamburg](https://opus4.kobv.de/opus4-bib-info/files/3645/HOS+Bibliothekartag.pdf) (PDF)
* [acohan/arxiv-tools](https://github.com/acohan/arxiv-tools)
* [Arxiv on Archive](https://archive.org/details/arxiv-bulk-metadata)
* [Metadata analysis of 80,000 arxiv:physics/astro-ph articles](https://quantumdynamics.wordpress.com/2016/06/12/metadata-analysis-of-80000-arxivastro-ph-articles-reveals-biased-moderation/)
* [Arxiv Harvesting](https://trislee.com/arxiv/)
* [Orcid](https://trello.com/c/3OrWa2ZY/5771-load-issn-metadata-into-registry-db-8)
* [Tutorial on Topological Data Analysis](https://github.com/shizuo-kaji/TutorialTopologicalDataAnalysis/blob/master/NLP_example.md)
* [Connectome](https://cms.www.switch.ch/fr/connectome/) (linked open research data, [#28](https://github.com/miku/metha/issues/28#issuecomment-1144526453)), [From MARCXML to Records in Contexts](https://zenodo.org/record/7400442/files/SWITCH%20Patrinum%20RiC_20221205_Wildi.pdf)
* [Comparison w/ other OAI tools](https://github.com/Deutsche-Digitale-Bibliothek/ddblabs-ometha#geschwindigkeit) (de)
* [biblio.ai](https://github.com/biblio-ai/extract)

## Asciicast

[![asciicast](https://asciinema.org/a/271660.svg)](https://asciinema.org/a/271660)
