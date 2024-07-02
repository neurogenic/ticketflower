# Stimula Build

## Startup

### Stimula

From the stimula project directory:
```
docker-compose up -d stimula_ticketflower
docker-compose exec stimula_ticketflower /app/stimula/apps/run_stimula_debug.sh
```

Claude coder project folder:

/app/ticketflower/ticketflower/stimula/stimula.json

### Postgres

Start docker for tickertflower server:

from ticketflower_server directory:
```
docker-compose up -d
```

### Django Server

Start server:

```
python manage.py runserver
```

Migrations: (for users app shown. that can be omitted)

```
python manage.py makemigrations users
python manage.py migrate
```