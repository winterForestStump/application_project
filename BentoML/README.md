# Serving YOLOv5 model with BentoML 

Install required dependencies:

```bash
pip install -r ./requirements.txt
```

The model is divided in parts and archieved into the 'yolo5m_model' folder. After unzipping you should have a `yolov5s.pt` file created in current directory (don't be confused by the 's' at the name of the file, it is still medium 'm' sized model).

## Run the service

Launch the service locally:
```bash
bentoml serve service.py:svc
```


## Test the endpoint

Visit http://127.0.0.1:3000 to submit input images via the web UI.

The sample service provides two different endpoints:
* `/invocation` - takes an image input and returns predictions in a tabular data format
* `/render` - takes an image input and returns the input image with boxes and labels rendered on top


To test the `/invocation` endpoint:

```bash
curl -X 'POST' \
  'http://localhost:3000/invocation' \
  -H 'accept: image/jpeg' \
  -H 'Content-Type: image/jpeg' \
  --data-binary '@data/bus.jpg'
```


## Build Bento

The `bentofile.yaml` have configured all required system packages and python dependencies. 

```bash
bentoml build
```

Once the Bento is built, containerize it as a Docker image for deployment:

```bash
bentoml containerize yolo_v5_demo:latest
```
