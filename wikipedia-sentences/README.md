# Semantic Wikipedia Search with Transformers and DistilBERT

![](https://docs.jina.ai/_images/jinabox-wikipedia.gif)

| item   | content                                          |
|--------|--------------------------------------------------|
| Input  | 1 text file with 1 sentence per line             |
| Output | *top_k* number of sentences that match input query |

This is an example of using [Jina](http://www.jina.ai)'s neural search framework to search through a [selection of individual Wikipedia sentences](https://www.kaggle.com/mikeortman/wikipedia-sentences) downloaded from Kaggle. It uses the [`distilbert-based-uncased`](https://huggingface.co/distilbert-base-uncased) language model from [Transformers](https://huggingface.co).

## 🐳 Run in Docker

To test this example you can run a Docker image with 30,000 pre-indexed sentences:

```sh
docker run -p 45678:45678 jinahub/app.example.wikipedia-sentences-30k:0.2.10-1.0.10
```

You can then query by running:

```sh
curl --request POST -d '{"top_k": 10, "mode": "search",  "data": ["hello world"]}' -H 'Content-Type: application/json' 'http://0.0.0.0:45678/api/search'
```

## 🐍 Setup

```sh
pip install -r requirements.txt
```

## 📇 Index

We'll start off by indexing a [small dataset of 50 sentences](data/toy-input.txt) to make sure everything is working:

```sh
python app.py -t index
```

To index the [full dataset](https://www.kaggle.com/mikeortman/wikipedia-sentences) (almost 900 MB):

1. Set up [Kaggle](https://www.kaggle.com/docs/api#getting-started-installation-&-authentication)
2. Run the script: `sh ./get_data.sh`
3. Set the input file: `export JINA_DATA_FILE='data/input.txt'`
4. Set the number of docs to index `export JINA_MAX_DOCS=30000` (or whatever number you prefer. The default is `50`)
5. Delete the old index: `rm -rf workspace`
6. Index your new dataset: `python app.py -t index`

## 🔍 Search

### With REST API

```sh
python app.py -t query_restful
```

Then:

```sh
curl --request POST -d '{"top_k": 10, "mode": "search",  "data": ["hello world"]}' -H 'Content-Type: application/json' 'http://0.0.0.0:45678/api/search'
````

Or use [Jinabox](https://jina.ai/jinabox.js/) with endpoint `http://127.0.0.1:45678/api/search`

### From the Terminal

```sh
python app.py -t query
```

## 👷 Build a Docker Image

This will create a Docker image with pre-indexed data and an open port for REST queries.

1. Run all the steps in setup and index first. Don't run anything in the search step!
2. If you want to [push to Jina Hub](#push-to-jina-hub) be sure to edit the `LABEL`s in `Dockerfile` and fields in `manifest.yml` to avoid clashing with other images
3. Run `docker build -t <your_image_name> .` in the root directory of this repo
5. Run it with `docker run -p 45678:45678 <your_image_name>`
6. Search using instructions from [Search section](#search) above

### Image name format

Please use the following name format for your Docker image, otherwise it will be rejected if you want to push it to Jina Hub. 

```
jinahub/type.kind.image-name:image-version-jina_version
```

For example:

```
jinahub/app.example.my-wikipedia-sentences-30k:0.2.10-1.0.10
```

## ⬆️ Push to [Jina Hub](https://github.com/jina-ai/jina-hub)

1. Ensure hub is installed with `pip install jina[hub]`
2. Run `jina hub login` and paste the code into your browser to authenticate
3. Run `jina hub push <your_image_name>`

### Next Steps

- [Enable incremental indexing](https://github.com/jina-ai/examples/tree/master/wikipedia-sentences-incremental)
- [Developer Guide: Build a similar text search app](https://docs.jina.ai/chapters/my_first_jina_app/)
