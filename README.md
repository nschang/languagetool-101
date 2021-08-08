This is a step-by-step instruction to setup and enable advanced features (FastText, word2vec, ngram) of LanguageTool on MacOS. 


## You might need this if you: 
* have a [LanguageTool Server](https://dev.languagetool.org/http-server)  (version 5.4 or higher) running locally. 
* want to turn on advanced features (word2vec, fastText, ngram) with more statistics rules


## word2vec [(LanguageTool Neural Network)](https://github.com/gulp21/languagetool-neural-network)

> Neural network based rules for confusion pair disambiguation using the word2vec model. Available in English, German, and Portuguese.

Download the desired language archive from https://languagetool.org/download/word2vec/. 

Here we use language `en` as example. 
  ```
  # get language data 
  mkdir word2vec
  cd word2vec
  wget -q --show-progress https://languagetool.org/download/word2vec/en.zip
  tar -xf *.zip
  
  # update config 
  echo 'word2vecModel=~/LanguageTool-5.4/word2vec' >> ~/.languagetool.cfg
  ```
## FastText
> FastText makes automatic language detection [much better than the built-in one.](https://github.com/languagetool-org/languagetool/blob/master/languagetool-standalone/CHANGES.md#http-api--lt-server-4)

Here we need the [fasttextModel](https://fasttext.cc/docs/en/language-identification.html), and the [fasttextBinary](https://fasttext.cc/docs/en/support.html).

  ```
  # get the latest version of fasttextModel
  wget -q --show-progress https://languagetool.org/download/fasttext.tar.gz
  tar https://languagetool.org/download/fasttext.tar.gz
  # rename the folder to avoid confusion
  mv ./fasttext ./fasttextModel 

  # clone and build fasttextBinary
  git clone https://github.com/facebookresearch/fastText.git
  cd fastText
  make

  # update config
  {echo 'fasttextModel=~/LanguageTool-5.4/fasttextModel/lid.176.bin'; echo 'fasttextBinary=~/LanguageTool-5.4/fastText/fasttext';} >> ~/.languagetool.cfg
  ```
This will put the fastnet model in ./fasttextModel, and the main binary `fasttext` in ./fastText. 

## [n-gram data](https://dev.languagetool.org/finding-errors-using-n-gram-data)

> Make sure you have a fast disk, i.e. an SSD. Without an SSD, using this data can make LanguageTool much slower.

> Download the data (8GB!) from http://languagetool.org/download/ngram-data/ - note: data is currently only available for English, German, French, and Spanish (plus some data for untested languages).

> Unzip it and put it in its own directory named `en`, `de`, `fr`, or `es`, depending on the language. The path you need to set in the next step is the directory that the `en` etc. directory is in, not that directory itself.

> if you use LT as LibreOffice/OpenOffice add-on: open the Options dialog and set the n-gram directory.

> if you use LT GUI (languagetool.jar): `java -jar languagetool.jar` --> Text Checking --> Options --> General --> set ngram data directory as `<path-to-your-SSD>/ngram-data`and word2vec data directory as `~/LanguageTool-5.4/word2vec`. Make sure to choose that directory (which contains the subfolders “de”, “en” etc.)


  ```
  # create a new directory in SSD named ngram-data
  mkdir <path-to-your-SSD>/ngram-data
  cd <path-to-your-SSD>/ngram-data
  
  # download files for selected language (>8GB)
  wget -q --show-progress http://languagetool.org/download/ngram-data/ngrams-en-20150817.zip | unzip http://languagetool.org/download/ngram-data/ngrams-en-20150817.zip 
  
  # update config 
  echo 'languageModel=<path-to-your-SSD>/ngram-data' >> ~/.languagetool.cfg
  ```

if everything builds successfully, your `~/.languagetool.cfg` should look like this: 
  ```
  word2vecModel=~/LanguageTool-5.4/word2vec
  fasttextModel=~/LanguageTool-5.4/fasttextModel/lid.176.bin
  fasttextBinary=~/LanguageTool-5.4/fastText/fasttext
  languageModel=<path-to-your-SSD>/ngram-data
  ```

Then, depending on how you use LanguageTool:

* Command line: `--config "~/.languagetool.cfg"`

* Server mode: `java -cp languagetool-server.jar org.languagetool.server.HTTPServer --port 8081 --allow-origin "*" --config "~/.languagetool.cfg"`

Test with sentence "Put on the breaks", which is an error that can only be detected using the n-gram rule, as of September 2020: 
http://localhost:8081/v2/check?language=en-US&text=Put+on+the+breaks

you should now see a new match "brake" instead of "break".
