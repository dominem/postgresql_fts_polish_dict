# PostgreSQL Full-Text Search - Polish Dictionary

1. Go to [sjp.pl](https://sjp.pl/slownik/ort/) and download `sjp-ispell-pl-[date]-src.tar.bz2` file.
2. Unpack the file and `cd` to the directory.
3. Download Polish stopwords from [GitHub repo](https://github.com/DominikMagdalenski/stopwords/blob/master/polish.stopwords.txt) and save it in the current directory.
4. Convert encodings to utf-8 and give proper extensions to files:
    ```bash
    $ iconv -f ISO_8859-2 -t utf-8 polish.aff > polish.affix
    $ iconv -f ISO_8859-2 -t utf-8 polish.all > polish.dict
    $ mv polish.stopwords.txt polish.stop
    ```
5. Copy files to `pg_config --sharedir` directory:
    ```bash
    sudo cp polish.affix `pg_config --sharedir`/tsearch_data/
    sudo cp polish.dict `pg_config --sharedir`/tsearch_data/
    sudo cp polish.stop `pg_config --sharedir`/tsearch_data/
    ```
6. Now, in postgres:
    ```postgres
    CREATE TEXT SEARCH DICTIONARY pl_ispell (
      Template = ispell,
      DictFile = polish,
      AffFile = polish,
      StopWords = polish
    );

    CREATE TEXT SEARCH CONFIGURATION pl_ispell(parser = default);

    ALTER TEXT SEARCH CONFIGURATION pl_ispell
      ALTER MAPPING FOR asciiword, asciihword, hword_asciipart, word, hword, hword_part
      WITH pl_ispell;
    ```
7. Test, in postgres:
    ```postgres
    SELECT to_tsvector('pl_ispell', 'Czuję się mniej więcej tak, jak ktoś, kto bujał w obłokach i nagle spadł.');

    output:
                                          to_tsvector                                     
    ------------------------------------------------------------------------------------
    'bujać':9 'czuć':1 'jaka':6 'mniej':3 'nagły':13 'obłok':11 'obłoki':11 'spaść':14
    (1 row)
    ```
