# Notes Database (MongoDB)

This container provides a MongoDB instance for the Personal Notes Application. It stores user accounts and notes data.

Overview:
- Database name: myapp (configurable)
- Default admin user: appuser / dbuser123 (configurable)
- Default port: 5000 (configurable)
- Canonical connection URL: mongodb://appuser:dbuser123@localhost:5000/?authSource=admin
- Application DB: myapp
- Collections used:
  - users
  - notes

Environment variables:
- MONGODB_URL: MongoDB connection string (driver format)
  - Example: mongodb://appuser:dbuser123@localhost:5000/?authSource=admin
- MONGODB_DB: Target database name
  - Example: myapp

Database connection details
- Local connection via mongosh:
  - mongosh mongodb://appuser:dbuser123@localhost:5000/myapp?authSource=admin
- Driver connection (recommended for backend services):
  - URL: mongodb://appuser:dbuser123@localhost:5000/?authSource=admin
  - DB: myapp

Required collections:
- users
- notes

Recommended indexes
To ensure correct behavior and performance, create the following indexes:

1) users collection
- Unique email
  - db.users.createIndex({ email: 1 }, { unique: true, name: "uniq_email" })

2) notes collection
- user_id index (to quickly fetch notes per user)
  - db.notes.createIndex({ user_id: 1 }, { name: "idx_user_id" })
- Text index on title and content (to support basic search)
  - db.notes.createIndex({ title: "text", content: "text" }, { name: "text_title_content" })

Optional: Ensure indexes on first run
The startup.sh script can optionally ensure these indexes exist. This is provided as a safe idempotent operation (it will only create the indexes if missing). See startup.sh for details.

Initial setup
- The startup.sh script will:
  - Start mongod on port 5000, bind to localhost
  - Create admin user appuser with readWriteAnyDatabase and userAdminAnyDatabase
  - Create application user appuser (same credentials) scoped to the myapp database with readWrite
  - Write db_connection.txt with a convenient mongosh command
  - Write db_visualizer/mongodb.env with canonical env variables for the visualizer tool
  - Optionally (if enabled), create the recommended indexes

How to run
- From this directory, run:
  - bash startup.sh

Connection info file
- After startup, db_connection.txt will contain:
  - mongosh mongodb://appuser:dbuser123@localhost:5000/myapp?authSource=admin

DB visualizer helper
- Source the environment for the simple DB viewer:
  - source db_visualizer/mongodb.env

.env.example for backend
- See .env.example in this directory. Copy these envs into your backend container’s .env file if needed.

Security notes
- Change default credentials and port in production.
- Do not expose MongoDB directly to the internet. Use network policies and firewalls.

Troubleshooting
- If mongod is already running on another port, the startup.sh will attempt to stop it or exit with a message.
- Log file is written to /var/lib/mongodb/mongod.log
- You can re-run startup.sh safely; user and DB creation are idempotent.

Index validation
- To list current indexes:
  - db.users.getIndexes()
  - db.notes.getIndexes()
