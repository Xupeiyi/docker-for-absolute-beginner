Dockerfile example:

```
# All dockerfiles must start with a FROM command
# Start from a base OS or another image
FROM Ubuntu

# install all dependencies
RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

# copy source code
COPY . /opt/source-code

# specify a command to run when the image is run as a container
ENTRYPOINT opt/source-code/app.py flask run
```
run 
```
docker build <context_dir> -t <tag name>
```
to build the image.

Each line of instruction creates a new layer with just the changes from the previous ones.
run 
```
docker history <image-name>
```
to see each layer. All the layers built are cached, so if one step fails, you don't have to start over again.