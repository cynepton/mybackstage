# [Backstage Test](https://backstage.io)


## Running this Locally

To start the app, clone the repo and run:
```sh
docker compose up
```
to run the `docker-compose.yml` file that runs a postgres database and exposes it on port `5433` on the host.

Export the required environment variables:
```sh
export POSTGRES_HOST=localhost
export POSTGRES_PORT=5433
export POSTGRES_USER=postgres
export POSTGRES_PASSWORD=postgres
```
The postgres credentials above are for the docker compose postgres database and can be updated if the values are changed from the defaults in the [docker-compose.yml](docker-compose.yml).

Create a `github-app-backstage-cynepton-org-test-credentials.yaml` file with the necessary credentials for the github app. The contents should look like this:
```yaml
# Name: App name
appId: 123456
webhookUrl: <url>
clientId: 123
clientSecret: 123
webhookSecret: 123
privateKey: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----
```

Run:
```sh
yarn install
yarn dev
```
to install node dependencies and start the application.

Import this catalog item: https://github.com/cynepton-org/backstage-test-app-1

## Setup Steps for a first time Backstage project

A simple documentation of the steps followed from the [Backstage Getting Started Docs](https://backstage.io/docs/getting-started/configuration#).

1. Create the app using the CLI. Run the command in the parent directory of the location we want to create the app in. This creates the app directory and initialises git and all the code for the backstage app. Italso installs the node packages.

    ```bash
    npx @backstage/create-app@latest
    ```

2. Follow the [docs to configure the app to use postgreSQL](https://backstage.io/docs/getting-started/configuration#install-and-configure-postgresql) as the database. By default, production uses Postgres and the config for production is added in the [app-config.production.yaml](app-config.production.yaml) file. The General config lives in the [app-config.yaml](app-config.yaml) file.

    ```bash
    # From your Backstage root directory
    yarn add --cwd packages/backend pg
    ```
    The command above installs the node [pg](https://www.npmjs.com/package/pg) package for postgres. Edit the [app-config.yaml](app-config.yaml) file and add the postgres config below to the `backend.database` object:

    ```yaml
    backend:
      database:
        client: pg
        connection:
          host: ${POSTGRES_HOST}
          port: ${POSTGRES_PORT}
          user: ${POSTGRES_USER}
          password: ${POSTGRES_PASSWORD}
    ```

3. Setup Github App for Authentication and Github Integration. Using [Github for authentication](https://backstage.io/docs/auth/github/provider) is one of the [available authentication provider options](https://backstage.io/docs/auth/). The [Github Integration](https://backstage.io/docs/integrations/github/locations) is one of the [available integrations](https://backstage.io/docs/integrations/) with Backstage as well. In this setup, we are using a single [github app](https://docs.github.com/en/apps/using-github-apps) to authenticate for both purposes, but separate github apps can be used as well. A dedicated [Oauth App](https://docs.github.com/en/apps/oauth-apps) can be used for the authentication also.

    To setup backstage with a Github app, there is a simple to use command that sets up the app and downloads the required credentials to a `github-app-backstage-cynepton-org-test-credentials.yaml` file. This file is secret and MUST NOT be commited to version control.
    
    ```bash
    yarn backstage-cli create-github-app <github org>
    ```

    The command above would prompt the user to select permissions for the app and redirect to the browser to create the app. When the github app is created successfully, the `github-app-backstage-cynepton-org-test-credentials.yaml` file would be created. The contents of this file are secret and would look like this:

    ```yaml
    # Name: App name
    appId: 123456
    webhookUrl: <url>
    clientId: 123
    clientSecret: 123
    webhookSecret: 123
    privateKey: |
        -----BEGIN RSA PRIVATE KEY-----
        ...
        -----END RSA PRIVATE KEY-----
    ```

    At this point, the app is setup for integration. Edit the Github app that was created, and add the Homepage URL (locally, this is: `http://localhost:3000`) and callback URL (locally, this is `http://localhost:7007/api/auth/github/handler/frame`). This sets up the app for authentication. See the [Add sign-in option to the frontend ection in the docs](https://backstage.io/docs/auth/#sign-in-configuration) to add the SignInPage React component to require users to signin into the code.

    Update the [app-config.yaml](app-config.yaml) file to include configuration for:

    **Authenticattion** - update the `auth.providers.github.development` field to extend the contents of the `github-app-backstage-cynepton-org-test-credentials.yaml` file.

    ```yaml
    auth:
    # see https://backstage.io/docs/auth/ to learn about auth providers
    providers:
      github:
        development:
          $include: github-app-backstage-cynepton-org-test-credentials.yaml
    ```

    **Integration** - update the `integrations.github[].apps[]` field to extend the contents of the `github-app-backstage-cynepton-org-test-credentials.yaml` file as well.

    ```yaml
    integrations:
      github:
        - host: github.com
          apps:
            - $include: github-app-backstage-cynepton-org-test-credentials.yaml
    ```