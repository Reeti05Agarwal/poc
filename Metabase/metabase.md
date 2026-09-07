
# Methods to Setup Metabase

## 1. Run Docker

docker run -d -p 3000:3000 \
  --name metabase \
  metabase/metabase-enterprise:v1.60.0

OSS images are metabase/metabase:v0.XX.Y
Enterprise/Pro images are metabase/metabase-enterprise:v1.XX.Y

## 2. Docker with real database

```
name: metabase-cve-2026-72900-lab

services:
  metabase:
    image: metabase/metabase-enterprise:v1.60.16
    ports:
      - "127.0.0.1:3000:3000"
    environment:
      MB_DB_TYPE: postgres
      MB_DB_DBNAME: metabase
      MB_DB_PORT: 5432
      MB_DB_USER: metabase
      MB_DB_PASS: labpassword
      MB_DB_HOST: postgres
      # Deliberately NOT setting MB_ENCRYPTION_SECRET_KEY.
      # This is the default state and is what makes stored
      # connection details readable in cleartext.
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - lab

  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: metabase
      POSTGRES_USER: metabase
      POSTGRES_PASSWORD: labpassword
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U metabase"]
      interval: 5s
      timeout: 5s
      retries: 10
    networks:
      - lab
    # No volume - state dies with the container, which is what
    # you want for a disposable lab.

  # Optional: a second Postgres to add as a "data source" in the
  # Metabase UI, so you can observe how its credentials are stored.
  target-db:
    image: postgres:16
    environment:
      POSTGRES_DB: sales
      POSTGRES_USER: fakeuser
      POSTGRES_PASSWORD: fake-credential-do-not-reuse
    networks:
      - lab

networks:
  lab:
    driver: bridge
```

```
docker compose up -d          # start
docker compose logs -f        # watch startup — first boot takes a minute or two
docker compose down           # stop, keeps the pgdata volume
```

Metabase lands on http://localhost:3000.


## 3. JAR

```bash
curl -O https://downloads.metabase.com/v1.60.0/metabase.jar
java -jar metabase.jar
```

export.txt
```txt
MB_DB_TYPE: postgres
MB_DB_DBNAME: metabase
MB_DB_PORT: 5432
MB_DB_USER: metabase
MB_DB_PASS: password12
MB_DB_HOST: postgres
POSTGRES_DB: metabase
POSTGRES_USER: metabase
POSTGRES_PASSWORD: password123
```

```bash
export -r export.txt
```
