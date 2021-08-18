
# You might need this if you: 
#### have a [LanguageTool Server](https://dev.languagetool.org/http-server)  (version 5.4 or higher) running locally on your mac
#### want to turn on advanced features (word2vec, fastText, ngram) 

## Jump to 
* [word2vec](#word2vec)

* [FastText](#fasttext)

* [n-gram data](#n-gram)

<a name="word2vec"></a>
## word2vec [(LanguageTool Neural Network)](https://github.com/gulp21/languagetool-neural-network)

> Neural network based rules for confusion pair disambiguation using the word2vec model. Available in English, German, and Portuguese.

Download the desired language archive from https://languagetool.org/download/word2vec/. 

  ```
  # get language data 
  mkdir word2vec
  cd word2vec
  wget -q --show-progress https://languagetool.org/download/word2vec/en.zip
  unzip *.zip
  
  # update config 
  echo 'word2vecModel=${HOME}/LanguageTool-5.4/word2vec' >> ${HOME}/.languagetool.cfg
  ```

<a name="fasttext"></a>
## FastText
> FastText makes automatic language detection [much better](https://github.com/languagetool-org/languagetool/blob/master/languagetool-standalone/CHANGES.md#http-api--lt-server-4) than the built-in one.

Here we need the [fasttextModel](https://fasttext.cc/docs/en/language-identification.html), and the [fasttextBinary](https://fasttext.cc/docs/en/support.html).

  ```
  # get the latest version of fasttextModel
  cd ${HOME}/LanguageTool-5.4/
  wget -q --show-progress https://languagetool.org/download/fasttext.tar.gz
  tar -xf https://languagetool.org/download/fasttext.tar.gz
  # rename the folder to avoid confusion
  mv ./fasttext ./fasttextModel 

  # clone and build fasttextBinary
  git clone https://github.com/facebookresearch/fastText.git
  cd fastText
  make

  # update config
  {echo 'fasttextModel=${HOME}/LanguageTool-5.4/fasttextModel/lid.176.bin'; echo 'fasttextBinary=
  /LanguageTool-5.4/fastText/fasttext';} >> ${HOME}/.languagetool.cfg
  ```
This will put the fasttext model in `${HOME}/LanguageTool-5.4/fasttextModel`, and the main binary `fasttext` in `${HOME}/LanguageTool-5.4/fastText`. 

<a name="n-gram"></a>
## n-gram data

From official doc: [Finding Errors using n-gram data](https://dev.languagetool.org/finding-errors-using-n-gram-data)

> Make sure you have a fast disk, i.e. an SSD. Without an SSD, using this data can make LanguageTool much slower.

> Download n--gram for your language from http://languagetool.org/download/ngram-data/ NOTE: As of 09/2021 only English, German, French, Spanish (plus some data for untested languages).

  ```
  # create a new directory in SSD named ngram-data
  mkdir /PATH/TO/YOUR/SSD/ngram-data
  cd /PATH/TO/YOUR/SSD/ngram-data
  
  # download files for selected language: en (>8GB)
  wget -q --show-progress http://languagetool.org/download/ngram-data/ngrams-en-20150817.zip | unzip http://languagetool.org/download/ngram-data/ngrams-en-20150817.zip 
  
  # update config 
  echo 'languageModel=/PATH/TO/YOUR/SSD/ngram-data' >> ${HOME}/.languagetool.cfg
  ```

if everything builds successfully, your `.languagetool.cfg` should look like this (Make one if it doesn't exist.):
  ```
  # LanguageTool configuration (5.4/2021-06-25 10:24:16 +0000)
  
  word2vecModel=${HOME}/LanguageTool-5.4/word2vec
  fasttextModel=${HOME}/LanguageTool-5.4/fasttextModel/lid.176.bin
  fasttextBinary=${HOME}/LanguageTool-5.4/fastText/fasttext
  languageModel=/PATH/TO/YOUR/SSD/ngram-data
  ```


## Test it out!
Now you can test it out, depending on how you use LanguageTool:

* Command line: 
  ```
  # this starts the commandline interface 
  # and reads input 'put on the breaks'
  echo "Put on the breaks" | java -jar languagetool-commandline.jar -l en-US --languagemodel "/PATH/TO/YOUR/SSD/ngram-data" --word2vecmodel "./word2vec" --fasttextmodel "./fasttextModel/lid.176.bin" --fasttextbinary "./fastText/fasttext" - 
  ```

* Server mode:
  ```
  java -cp languagetool-server.jar org.languagetool.server.HTTPServer --port 8081 --allow-origin "*" --config "${HOME}/.languagetool.cfg"
  ```

* LT as LibreOffice/OpenOffice add-on: open the `Options` dialog and set the n-gram directory.

+ LT GUI (languagetool.jar): `java -jar languagetool.jar` --> Text Checking --> Options --> General --> set ngram data directory as `/PATH/TO/YOUR/SSD/ngram-data`and word2vec data directory as `${HOME}/LanguageTool-5.4/word2vec`. Make sure to choose that directory (which contains the subfolders “de”, “en” etc.)


Test with sentence "Put on the breaks", which is an error that can only be detected using the n-gram rule:
http://localhost:8081/v2/check?language=en-US&text=Put+on+the+breaks


