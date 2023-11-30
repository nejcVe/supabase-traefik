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

## Environment

### Traefik

- `ACME_EMAIL` - email address used with Let's Encrypt to generate SSL certificates
- `BASIC_AUTH_CREDS` - credentials for Basic Authentication to access Traefik dashboard
  - Must not be in plain text format, so generate them with a program like `htpasswd`
  - example command: `echo $(htpasswd -nbB username "password") | sed -e s/\\$/\\$\\$/g`)
- `WHITELIST_IPS` - IPs that are whitelisted to access Treafik Dashboard, delimited by a comma
- `TRAEFIK_HOST` - the address you want to use for Traefik Dashboard (e.g. `traefik.example.net`)

### Supabase

#### Secrets

- `POSTGRES_PASSWORD` - password for the Postgres user
- `JWT_SECRET` - a secret string used to sign the JWT when generated
- `ANON_KEY` - JWT as key for the Anonymous user (use [Supabase's token generation service](https://supabase.com/docs/guides/self-hosting/docker#generate-api-keys) with `ANON_KEY` payload and paste it here)
- `SERVICE_ROLE_KEY` - JWT as key for the Admin user (use [Supabase's token generation service](https://supabase.com/docs/guides/self-hosting/docker#generate-api-keys) with `SERVICE_KEY` payload and paste it here)

#### Traefik

This part is used to configure the Traefik's service labels

- `SUPABASE_HOST` - the address you want to use for Supabase (e.g. `supabase.example.net`)
- `SUPABASE_BASIC_AUTH_CREDS` - credentials for Basic Authentication to access Supabase Studio
  - generated the same way as `BASIC_AUTH_CREDS` in the Traefik setup

#### Database

You can leave these on default values if you are using the provided database. If you want to use your own database, then change these to connect to that one.

- `POSTGRES_HOST` - host where Postgres is running
- `POSTGRES_DB` - which database to use
- `POSTGRES_PORT` - on what port the Postgres instance is running

#### Authentication

- `ADDITIONAL_REDIRECT_URLS` - allowed redirect URLs for the authentication endpoint
- `JWT_EXPIRY` - how long the JWT is valid for
- `DISABLE_SIGNUP` - whether you want to only allow users to register when you send them an invitation
- `MAILER_*` and `SMTP_*` - used for email authentication setup, more info on [Gotrue GitHub - Email setup](https://github.com/supabase/gotrue#e-mail)
- `ENABLE_PHONE_SIGNUP` and `ENABLE_PHONE_AUTOCONFIRM` - used for phone authentication, more info on [Gotrue Github - Phone setup](https://github.com/supabase/gotrue#phone-auth)

#### Studio

- `STUDIO_DEFAULT_ORGANIZATION` - name of the default organization for Supabase Studio
- `STUDIO_DEFAULT_PROJECT` - name of the default project for Supabase Studio
- `STUDIO_PORT` - which port the Supabase Studio service is listening on
- `IMGPROXY_ENABLE_WEBP_DETECTION` - if you want imgproxy to support webp
  - this means that when a file extension is omitted in the URL and the browser supports WebP, then it will be the resulting format

#### Functions

- `FUNCTIONS_VERIFY_JWT` - whether to verify JWT when running functions (this can only be enabled for ALL functions, not "per function")

#### Logs

- `LOGFLARE_LOGGER_BACKEND_API_KEY` - Logflare backend API key
- `LOGFLARE_API_KEY` - API key for the Logflare endpoint
  - make sure to also change the `api_key` query parameter for endpoints in the `sinks` section in [vector.yml file](supabase/volumes/logs/vector.yml)
- `DOCKER_SOCKET_LOCATION` - Docker socker location for your OS

These two are only needed if you want to use BigQuery backend for logs. You need a Google Cloud service account with billing enabled.

- `GOOGLE_PROJECT_ID` and `GOOGLE_PROJECT_NUMBER`


### OAuth Providers

If you would like to add OAuth providers for your app, you have to edit the `auth` service environment variables in the `/supabase/docker-compose.yaml`. There is an example present for GoogleAuth.

```
GOTRUE_EXTERNAL_GOOGLE_ENABLED: true
GOTRUE_EXTERNAL_GOOGLE_REDIRECT_URI: ${SITE_URL}/auth/v1/callback
GOTRUE_EXTERNAL_GOOGLE_CLIENT_ID: id
GOTRUE_EXTERNAL_GOOGLE_SECRET: secret
```

Documentation for which providers are available and how to add them is available on the official [Supabase GitHub](https://github.com/supabase/gotrue#external-authentication-providers)
