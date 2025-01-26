## Known issues

postgres-deployment by default will create an empty database with the user's name,in our case user "odoo" database "odoo".
This will cause our odoo service to throw "internal server error" database  "odoo" not initialized.
To fix this, we need to delete the database manually from postgres pods.

## STEPS
make sure volume permission in hosts is set to read/write for everyone,unless pods cannot read/write on the volumes.
