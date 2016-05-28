# PostgreSQL text-search utils

A collection of files and patterns to improve the default PostgreSQL text search.

## What's this?

The PostgreSQL text search engine is one of the best available across both commercial and open-source RDBMs, but it does miss a number features when it comes to supporting languages other than English.

This repository is aiming to provide useful patterns and files that are potentially missing in a default PostgreSQL distribution.

## Note

The location of `$SHAREDIR` is OS/PostgreSQL distribution specific. On a Debian/Ubuntu-based box with PostgreSQL 9.5, you'll find it at `/usr/share/postgresql/9.5/`. Ask around on stackoverflow if you can't find yours. Please refrain from raising issues in this repo to ask about it.

## Updated unaccent rules

The `unaccent.rules` file on your system may lack a number of UTF8 characters, depending on how old is your PostgreSQL version. If necessary, you can replace the one in your `$SHAREDIR/tsearch_data` folder with the latest one from the PostgreSQL repository:

    cd $SHAREDIR/tsearch_data
    curl -O https://raw.githubusercontent.com/postgres/postgres/master/contrib/unaccent/unaccent.rules

## Language-specific stop words

In a standard PostgreSQL 9.5 package there are 14 `*.stop` files covering Danish, Dutch, English, Finnish, French, German, Hungarian, Italian, Norwegian, Portuguese, Russian, Spanish, Swedish and Turkish.

If your language is not among those, there's a chance you'll find the missing file in this repository.

    cd $SHAREDIR/tsearch_data
    curl -O https://raw.githubusercontent.com/icflorescu/postgresql-tsearch-utils/master/romanian.stop

If the file you're looking for is not here, then by all means feel free to contribute with a useful pull-request. Simply raising an issue to ask for it will probably not help, but I'll gladly accept pull-requests with `greek.stop`, `bulgarian.stop`, `czech.stop`, etc.

## Useful information around the web

- [Controlling Text Search](https://www.postgresql.org/docs/current/static/textsearch-controls.html) in the official PostgreSQL manual
- [Postgres full-text search is Good Enough!](http://rachbelaid.com/postgres-full-text-search-is-good-enough/)
- [Full text search in milliseconds with PostgreSQL](https://blog.lateral.io/2015/05/full-text-search-in-milliseconds-with-postgresql/)
- [PostgreSQL: A full text search engine](http://shisaa.jp/postset/postgresql-full-text-search-part-1.html)
- [Indexing for full text search in PostgreSQL](https://www.compose.io/articles/indexing-for-full-text-search-in-postgresql/)
- [From LIKE to Full-Text Search](http://www.nomadblue.com/blog/django/from-like-to-full-text-search-part-I/)

## Example

Here's how to create an improved search configuration for Romanian language:

    /* Make sure you've updated the unaccent.rules file
     * before creating extension
     */
    CREATE EXTENSION unaccent;

    /* Create a Romanian "snowball" dictionary that
     * takes into account stopwords defined in romanian.stop
     */
    CREATE TEXT SEARCH DICTIONARY ro (
        TEMPLATE  = snowball
      , LANGUAGE  = romanian
      , STOPWORDS = romanian
    );

    /* Create a new text search configuration
     * based on the newly created dictionary and unaccent.rules
     */
    CREATE TEXT SEARCH CONFIGURATION RO (COPY = romanian);
    ALTER TEXT SEARCH CONFIGURATION RO
      ALTER MAPPING FOR asciiword, asciihword, hword_asciipart, hword, hword_part, word
      WITH unaccent, ro;

    /* This will give you wrong results:
     * 'autom':10 'cu':3 'cut':7 'de':8 'integral':5 'limuzin':1 'tracțiun':4 'verd':2 'vitez':9 'și':6
     */
    SELECT to_tsvector('romanian', 'limuzină verde cu tracțiune integrală și cutie de viteze automată');

    /* But using the new search configuration will give you better results:
     * 'autom':10 'cut':7 'integral':5 'limuzin':1 'tractiun':4 'verd':2 'vitez':9
     */
    SELECT to_tsvector('ro', 'limuzină verde cu tracțiune integrală și cutie de viteze automată');

## Read carefully before raising issues

I'm getting lots of questions from people just learning to do web development or simply looking to solve a very specific problem they're dealing with. While I will answer some of them for the benefit of the community, please understand that open-source is a shared effort and it's definitely not about piggybacking on other people's work. On places like GitHub, that means raising issues is encouraged, but **coming up with useful pull-requests is a lot better**. If I'm willing to share some of my code for free, I'm doing it for a number of reasons: my own intellectual challenges, pride, arrogance, stubbornness to believe I'm bringing a contribution to common progress and freedom, etc. Your particular well-being is probably not one of those reasons. I'm not in the business of providing free consultancy, so if you need my help to solve your specific problem, there's a fee for that.

## Asking for help or a new feature

See the note above. If you need help and are willing to pay for it, drop me a message. If you have an idea about a new feature that doesn't break existing ones and you're willing to invest effort to make it happen, have a look at the code and feel free to make a pull-request.

## Credits

See contributors [here](https://github.com/icflorescu/postgresql-tsearch-utils/graphs/contributors).

If you find this repo useful, don't hesitate to give it a star and [spread the word](http://twitter.com/share?text=Checkout%20this%%20PostgreSQL%20text%20search%20utils%20repo!&amp;url=http%3A%2F%2Fgithub.com/icflorescu/postgresql-tsearch-utils&amp;hashtags=PostgreSQL,database,textsearch&amp;via=icflorescu).

# License

The [ISC License](https://github.com/icflorescu/postgresql-tsearch-utils/blob/master/LICENSE).
