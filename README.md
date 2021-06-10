# WLO Discipline Classification

A [Docker](https://docker.com/)/[Python](https://www.python.org/)/[Keras](https://keras.io/)/[Tensorflow](https://www.tensorflow.org/) utility to train and predict *subject areas* for the [WLO project](https://github.com/openeduhub/) dataset.

We provide different Dockerfiles for [Development and Training](#development-and-training) and
[Deployment](#deployment):

- `Dockerfile.dev`
    - For development and training
    - Mounts project directories into the Docker container at runtime
    - Includes training dependencies
- `Dockerfile.prod`
    - For deployment
    - Burns required data into the Docker image
    - Runs the web service on startup

## Development and Training
 
### Prerequisites

- Install [Docker](https://docker.com/).
- (The training script `runTraining.sh` requires the Nvidia Docker runtime installed. For processing without a GPU remove the `--runtime=nvidia` parameter in the script's docker command.)

- Build the Docker image.

```
sh build.sh
```

### Training

(The `data` folder already contains a pretrained model.)

- The following script retrieves and processes the latest [dataset](https://github.com/openeduhub/oeh-wlo-data-dump), which results in the `data/wirlernenonline.oeh.csv` file containing the relevant documents (documents with a discipline property).

```
sh prepareData.sh
```

- This script initiates the training, which results in the model file `data/wirlernenonline.oeh.h5`, the file with class labels `data/wirlernenonline.oeh.npy`, and the tokenizer serialization `data/wirlernenonline.oeh.pickle` (existing files will be overwritten without warning).

```
sh runTraining.sh
```

### Prediction

- To test the prediction just query the model with an arbitrary text.

```
sh runPrediction.sh "Der Satz des Pythagoras lautet: a^2 + b^2 = c^2."
```

The result is a list of tuples of a score and its corresponding class name (name of discipline). Only the top three items are retrieved, in descending order.

### Webservice

- To run the subject prediction tool as a simple REST based webservice, the following script can be used:

```
sh runService.sh
```

- The scripts deploys a CherryPy webservice in a docker container listening at `http://localhost:8080/predict_subject`.

- To retrieve the recommendations, create a POST request and submit a json document with a text as for example: 

```
curl -d '{"text" : "Der Satz des Pythagoras lautet: a^2 + b^2 = c^2."}' -H "Content-Type: application/json" -X POST http://0.0.0.0:8080/predict_subjects
```	

## Deployment

### Build

```sh
# Build local image
docker build -f Dockerfile.prod -t openeduhub/discipline-classification:local .
```

Online builds are built and uploaded automatically using [Github Actions](https://github.com/openeduhub/discipline-classification/actions).

### Run

```sh
# Run local build
docker run --rm --name discipline-classification -p 8080:8080 openeduhub/discipline-classification:local

# Run online build
docker run --rm --name discipline-classification -p 8080:8080 openeduhub/discipline-classification:main
```