# Replacer

Unknown word replacer in Neural Machine Translation (NMT)

[![wercker status](https://app.wercker.com/status/f85bf4841c6422cd5ddfba7bdf635318/s/ "wercker status")](https://app.wercker.com/project/byKey/f85bf4841c6422cd5ddfba7bdf635318)

## Requirements
- Python >=3.3

## Install
Currently unk-replacer is not registered PyPI, so you need to
get it from github.

```
pip install git+https://github.com/hitochan777/unk-replacer.git
```

If you plan on modifying the code, it is better to do a "editable" installation:

```
git clone https://github.com/hitochan777/unk-replacer.git
cd unk-replacer
pip install -e .
```

## Basic Usage

1. Build source and target vocabulary from the training data with the following command

    ```bash
    unk-rep build_vocab \
        word \
        --source-file /path/to/source/training/data \
        --target-file /path/to/target/training/data \
        --src-vocab-size 50000 \
        --tgt-vocab-size 50000 \
        --output-file /path/to/json/vocab/file
    ```

2. Get word alignment for the parallel corpora and lexical translation tables for both direction.
   
   Typically you can obtain the lexical translation tables as byproducts 
   of word alignment. 
   You can use GIZA++ or mgiza because they are fast.
   However, we recommend that you use [Nile](https://TODO.com),
   which is a supervised alignment model rather than GIZA++, 
   because it produces much better alignment.

2. Train source and target Word2vec models
   
   For example, you can use `gensim` module to train a word2vec model from `TRAIN`
   and save it to `MODEL_NAME`.
 
   ```bash
   python -m gensim.models.word2vec \
   -train TRAIN \
   -output MODEL_NAME
   ```
   
   There are many parameters you can change.
   For more information, type
   
   ```bash
   python -m gensim.models.word2vec -h
   ```

3. Replace unknown words in the training data with the
    following command.
   
    ```bash
    unk-rep replace-parallel \
        --root-dir /path/to/save/artifacts \
        --src-w2v-model /path/to/source/word2vec/model \
        --tgt-w2v-model /path/to/target/word2vec/model \
        --lex-e2f /path/to/target/to/source/lex/dict \
        --lex-f2e /path/to/source/to/target/lex/dict \
        --train-src /path/to/source/training/data \
        --train-tgt /path/to/target/training/data \
        --train-align /path/to/word/alignment/for/training/data \
        --vocab /path/to/json/vocab/file \
        --memory /path/to/save/replacement/memory \
        --replace-type multi
    ```

    If you also want to replace unknown words in **development data**,
    you can specify the paths to the source development data (`--dev-src`), target development data (`--dev-tgt`), 
    word alignment (`--dev-align`).
    
    If you want to replace unknown words in one-to-one alignment only, you can set `--replace-type` to `1-to-1`.

4. Train NMT model with the replaced training data from step 3

5. Replace unknown words in the test data with the following command.

    ```bash
    unk-rep replace-input \
        --root-dir /path/to/save/artifacts \
        --w2v-model /path/to/source/word2vec/model \
        --input /path/to/input/data \
        --vocab /path/to/json/vocab/file \
        --replace-log /path/to/save/replace/log/file
    ```

6. Translate the replaced test data with the trained NMT model.

    We recommend that you ensemble several models because it normally
    leads to the better attention.

7. Restore the final translation with the following command.

    ```bash
    unk-rep restore \
        --translation /path/to/translation \
        --orig-input /path/to/original/input \
        --replaced-input /path/to/replaced/input \
        --output /path/to/save/final/translation \
        --lex-e2f /path/to/target/to/source/lex/dict \
        --lex-f2e /path/to/source/to/target/lex/dict \
        --replace-log /path/to/replace/log \
        --attention /path/to/attention \
        --lex-backoff
    ```
