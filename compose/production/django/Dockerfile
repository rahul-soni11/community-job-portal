FROM node:10-stretch-slim as client-builder

WORKDIR /app
COPY ./package.json /app
RUN npm install && npm cache clean --force
COPY . /app
RUN npm run build

# Python build stage
FROM python:3.8-slim-buster

ENV PYTHONUNBUFFERED 1
ENV POETRY_VERSION=1.0.10

RUN apt-get update \
  # dependencies for building Python packages
  && apt-get install -y build-essential \
  # psycopg2 dependencies
  && apt-get install -y libpq-dev \
  # Translations dependencies
  && apt-get install -y gettext \
  # cleaning up unused files
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/* \

RUN pip install "poetry==$POETRY_VERSION"

RUN addgroup --system django \
    && adduser --system --ingroup django django

# Copy only requirements, to cache them in docker layer:
WORKDIR /pysetup
COPY ./poetry.lock ./pyproject.toml /pysetup/

# Project initialization:
RUN poetry config virtualenvs.create false \
  && poetry install --no-dev --no-interaction --no-ansi


COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint
RUN chown django /entrypoint

COPY ./compose/production/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start
RUN chown django /start
COPY ./compose/production/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r$//g' /start-celeryworker
RUN chmod +x /start-celeryworker
RUN chown django /start-celeryworker

COPY ./compose/production/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r$//g' /start-celerybeat
RUN chmod +x /start-celerybeat
RUN chown django /start-celerybeat

COPY ./compose/production/django/celery/flower/start /start-flower
RUN sed -i 's/\r$//g' /start-flower
RUN chmod +x /start-flower
COPY --from=client-builder --chown=django:django /app /app


USER django

WORKDIR /app

ENTRYPOINT ["/entrypoint"]
