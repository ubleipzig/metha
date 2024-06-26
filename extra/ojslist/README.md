# OJS list journals

[OJS](https://pkp.sfu.ca/ojs/) is an open source software application for
managing and publishing scholarly journals.

It seems to have OAI enabled by default. OJS allows to manage multiple journals
for a site, e.g.
[https://www.aaai.org/ocs/index.php](https://www.aaai.org/ocs/index.php).

Given such a supersite, extract all OAI endpoints quickly.

```
$ ./ojslist.sh -i https://www.aaai.org/ocs/index.php
https://www.aaai.org/ocs/index.php/AAAI/index/oai
https://www.aaai.org/ocs/index.php/AIIDE/index/oai
https://www.aaai.org/ocs/index.php/DC/index/oai
https://www.aaai.org/ocs/index.php/EAAI/index/oai
https://www.aaai.org/ocs/index.php/ECP/index/oai
https://www.aaai.org/ocs/index.php/FLAIRS/index/oai
https://www.aaai.org/ocs/index.php/FSS/index/oai
https://www.aaai.org/ocs/index.php/HCOMP/index/oai
https://www.aaai.org/ocs/index.php/IAAI/index/oai
https://www.aaai.org/ocs/index.php/ICAPS/index/oai
https://www.aaai.org/ocs/index.php/ICCCD/index/oai
https://www.aaai.org/ocs/index.php/ICWSM/index/oai
https://www.aaai.org/ocs/index.php/IJCAI/index/oai
https://www.aaai.org/ocs/index.php/INT/index/oai
https://www.aaai.org/ocs/index.php/KR/index/oai
https://www.aaai.org/ocs/index.php/SARA/index/oai
https://www.aaai.org/ocs/index.php/SOCS/index/oai
https://www.aaai.org/ocs/index.php/SSS/index/oai
https://www.aaai.org/ocs/index.php/WS/index/oai
```

More examples:

* http://www.uel.br/revistas/uel/index.php
* https://www.uni-hildesheim.de/ojs/index.php/

Find more OJS instances:

* [https://www.google.com/search?q=inurl:"ojs/index.php"](https://www.google.com/search?q=inurl:"ojs/index.php")
* [https://www.google.com/search?client=ubuntu&channel=fs&q=inurl%3A%22article%2Fview%22+inurl%3A%22index.php%22&ie=utf-8&oe=utf-8](https://www.google.com/search?client=ubuntu&channel=fs&q=inurl%3A%22article%2Fview%22+inurl%3A%22index.php%22&ie=utf-8&oe=utf-8)

Some exceptions:

```
$ curl -s https://royalliteglobal.com/ | \
    pup 'a attr{href}' | \
    sort -u | \
    grep -Eo "https://royalliteglobal.com/[^/]*" | \
    sort -u | \
    grep -v index | \
    awk '{ print $0"/oai" }'
```

More exceptions:

* https://biblioteca.ucp.edu.co/ojs/index.php

```
$ OPENSSL_CONF=~/.openssl_allow_tls1.0.cnf curl -k -vL "https://biblioteca.ucp.edu.co/ojs/index.php" | grep "Ver Revista" | grep -o "https://[^']*
```
