# MongoDB on Wodby

What Wodby sets up for this database service, which runs the official MongoDB image. Check it before creating databases or users by hand, or writing a connection string into an application.

## Database, user and passwords

Wodby manages the database and its user; the application does not create them.

- One database and one user are created for the environment, both named after the application and the environment.
- The user is created in the `admin` database and has the `readWrite` role on the environment's database only. A client therefore authenticates against `admin` (`authSource=admin`), not against the application database. Authenticating against the application database is the usual reason a correct password is refused.
- Wodby creates the database by creating a collection named `__wodby` in it. Leave that collection in place.
- The administrator is `root` (`MONGO_INITDB_ROOT_USERNAME`, `MONGO_INITDB_ROOT_PASSWORD` in the container). Its password and the user's password are generated once per environment (tokens `root_password` and `password`).

## How a linked service reaches it

- Host: the name of this app service inside the environment. Port: `27017`.
- A service linked to this one receives the connection details as environment variables defined by its own link. Read those in the application; do not hardcode them and do not use `root` from the application.
- A connection URL that Wodby builds for a link already carries `authSource=admin`. A URL assembled in the application from host, port, user and password has to add it.

## Data, backups and imports

- Data is on the `data` volume, mounted at `/data/db`. The service runs a single instance.
- The backup is a `.tar.gz` holding a gzipped `mongodump` archive of the environment's database and a restore script.
- The database import takes a `.tar.gz` or `.tgz` backup made by this service.
- The manifest declares no configuration settings; the server runs with the image's defaults.

## Check the result

In the database container, `mongosh --quiet -u "$MONGO_INITDB_ROOT_USERNAME" -p "$MONGO_INITDB_ROOT_PASSWORD" --authenticationDatabase admin --eval 'db.adminCommand({listDatabases: 1})'` lists the databases.
