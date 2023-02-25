FROM python:3.9.1

RUN pip install pandas

WORKDIR /app
COPY pipeline.py pipeline.py

ENTRYPOINT [ "python", "pipeline.py" ]


#docker build -t test:pandas .
#docker run -it test:pandas 2021-01-15

#docker compose: a way of running multiple images

#volumes: a way of mapping folder in the file system on the host machine to a folder in the container --> dont want to lose the state when we run the docker everytime
#thats why we need to map a folder on our host machine to a folder in the container--> called mounting