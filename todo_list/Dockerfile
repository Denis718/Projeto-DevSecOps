# Stage 1: Build dependencies
FROM python:3.12.2-slim-bullseye AS builder

# Install system dependencies
RUN apt-get update \
    && apt-get -y install postgresql-server-dev-all gcc python3-dev musl-dev libffi-dev openssl cargo libjpeg-dev zlib1g


# Copy project
COPY . .

#Install dependencies
RUN pip install config
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /home/todo/wheels -r requirements.txt

# Stage 2: Final image
FROM python:3.12.2-slim-bullseye

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Install system dependencies
RUN apt-get update && apt-get install -y libpq-dev

# Create todo user and directories
RUN mkdir -p /home/todo \
    && addgroup --system todo \
    && adduser --system --group todo \
    && mkdir -p /home/todo/web/staticfiles

# Set work directory
WORKDIR /home/todo/web

# Copy project files
COPY --from=builder /usr/local/lib/python3.12/site-packages /usr/local/lib/python3.12/site-packages
COPY --from=builder /usr/local/bin /usr/local/bin
COPY --from=builder /home/todo/wheels /home/todo/wheels
COPY --chown=todo:todo . .

# Install pip and dependencies
RUN pip install --upgrade pip \
    && pip install --no-cache /home/todo/wheels/* \
    && pip install config

# Copy and set permissions for entrypoint script
COPY --chown=todo:todo entrypoint.sh .
RUN chmod +x entrypoint.sh

# Change to the app user
USER todo

# Define entrypoint
ENTRYPOINT ["./entrypoint.sh"]
CMD ["gunicorn", "--config", "todo_project.wsgi"]

# pull official base image
# FROM python:3.10.10-slim-bullseye

# # set environment variables
# ENV PYTHONDONTWRITEBYTECODE 1
# ENV PYTHONUNBUFFERED 1

# # install psycopg2 and cryptography dependencies
# RUN apt-get update \
#     && apt-get -y install postgresql-server-dev-all gcc python3-dev musl-dev libffi-dev openssl cargo libjpeg-dev zlib1g

# # create directory for the todo user
# RUN mkdir -p /home/todo

# # create the todo user
# RUN addgroup --system todo && adduser --system --group todo

# # create the appropriate directories
# ENV HOME=/home/todo
# ENV APP_HOME=/home/todo/web
# RUN mkdir $APP_HOME
# RUN mkdir $APP_HOME/staticfiles
# #RUN mkdir $APP_HOME/mediafiles
# WORKDIR $APP_HOME

# # Install pip
# RUN pip install --upgrade pip

# # Copy project
# COPY . .

# #Install dependencies
# RUN pip install config
# RUN pip wheel --no-cache-dir --no-deps --wheel-dir /home/todo/wheels -r requirements.txt

# # install dependencies
# RUN apt-get update && apt-get install -y libpq-dev
# RUN pip install --no-cache /home/todo/wheels/*


# # copy entrypoint.sh
# COPY ./entrypoint.sh .
# RUN sed -i 's/\r$//g' $APP_HOME/entrypoint.sh
# RUN chmod +x $APP_HOME/entrypoint.sh

# # chown all the files to the app user
# RUN chown -R todo:todo $APP_HOME

# # change to the app user
# USER todo

# # run entrypoint.sh
# ENTRYPOINT ["/home/todo/web/entrypoint.sh"]
# CMD ["gunicorn", "--config", "todo_project.wsgi"]
