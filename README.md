# Magento Elasticsearch Italian Stemmer Fix

This patch enables the correct Italian stemmer for Magento stores using Elasticsearch with the it_IT locale.

--------------------------------------------------

PROBLEM

In Magento installations using Elasticsearch, the Italian store view (it_IT) may incorrectly fall back to the default stemmer configuration.

As a result, the search index can be generated with:

"language": "english"

instead of the correct:

"language": "italian"

This leads to poor search relevance for Italian queries, especially when handling singular/plural variations.

Examples:
- abrasivo → abrasivi
- utensile → utensili
- lucidatrice → lucidatrici

--------------------------------------------------

ROOT CAUSE

The Magento Elasticsearch configuration file:

vendor/magento/module-elasticsearch/etc/esconfig.xml

does NOT include the Italian locale mapping by default:

<it_IT>italian</it_IT>

--------------------------------------------------

SOLUTION

Add the following line inside the <stemmer> node:

<it_IT>italian</it_IT>

--------------------------------------------------

PATCH INCLUDED

This repository provides a ready-to-use patch:

patches/magento-elasticsearch-it-stemmer.patch

--------------------------------------------------

TESTED ENVIRONMENT

- Magento: 2.4.6-p13
- Elasticsearch: 7.17.x
- Locale: it_IT

--------------------------------------------------

VERIFICATION

After applying the patch and reindexing, verify the Elasticsearch index settings:

curl -s http://127.0.0.1:9200/INDEX_NAME/_settings?pretty | grep -i -A 8 -B 3 default_stemmer

Expected result:

"filter.default_stemmer" : {
  "type" : "stemmer",
  "language" : "italian"
}

--------------------------------------------------

REINDEX COMMANDS

php bin/magento cache:flush
php bin/magento indexer:reindex catalogsearch_fulltext
php bin/magento mirasvit:search:reindex --store=1

--------------------------------------------------

NOTES

- This fix significantly improves search accuracy for Italian language stores.
- Verified by comparing search results before and after applying the patch.
- Example:
  Before → abrasivo ≠ abrasivi
  After  → both return the same result set


  ## Live Example

This fix has been tested on a real Magento store:

👉 https://ladecormarmi.com

You can test search behavior directly using Italian queries such as:

- abrasivo / abrasivi
- utensile / utensili
- lucidatrice / lucidatrici
