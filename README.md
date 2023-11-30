# Supabase deployment with Traefik 2.10
This is an example/template for production deployment of the [Supabase](https://supabase.com/) stack with Traefik as a reverse proxy.

## Directory structure
The projects are contained in their own directories. Each project has an `.example.env` file and a `docker-compose.yaml` file.

The `/supabase/volumes` directory contains the scripts and configurations to initialize the Supabase services and database with.

```
.
├── supabase
│   ├── docker-compose.yaml
│   ├── .env.example
│   └── volumes
└── traefik
    ├── docker-compose.yaml
    └── .env.example
```

## Prerequisites
- A VM (or any other machine) capable of running docker compose.
- A domain that points the VM you want to use 
    - Can be a wildcard domain `*.example.domain`
    - Or single records, one for Supabase (e.g. `supabase.example.domain`) and an optional domain for Traefik GUI/Dashboard (e.g. `traefik.example.domain`)

## Usage

1. Clone the git repository

    `git clone https://github.com/nejcVe/supabase-traefik.git`

2. Go to the `/traefik` directory and setup the environment variables. Then start the traefik container.
```
cd traefik
cp .example.env .env
nano .env
docker compose up -d
```

3. Now go to the `/supabase` directory and do the same there. Don't forget to also change the `/supabase/volumes/logs/vector.yml` with the key you set in `.env`.

```
cd supabase
cp .example.env .env
nano .env
nano volumes/logs/vector.yml
docker compose up -d
```

4. All the services and their routers should be visible in the Traefik Dashboard (e.g. traefik.example.domain). And the Supabase Studio should be available on the `supabase.example.domain` link.

### OAuth Providers

If you would like to add OAuth providers for your app, you have to edit the `auth` service environment variables in the `/supabase/docker-compose.yaml`. There is an example present for GoogleAuth.

```
GOTRUE_EXTERNAL_GOOGLE_ENABLED: true
GOTRUE_EXTERNAL_GOOGLE_REDIRECT_URI: ${SITE_URL}/auth/v1/callback
GOTRUE_EXTERNAL_GOOGLE_CLIENT_ID: id
GOTRUE_EXTERNAL_GOOGLE_SECRET: secret
```

Documentation for which providers are available and how to add them is available on the official [Supabase GitHub](https://github.com/supabase/gotrue#external-authentication-providers)
