# Hasura 2.0 `on_conflict` breaking change

First, we will examine the behaviour in Hasura 1.3.3.

- Run `docker compose up -d` to bring up the Postgres database and Hasura instance.
- Run `hasura console` to launch the Hasura console
- Observe:
  - The table `names` contains two rows, with `unique_name`s `foo` and `bar`
  - The `user` role has insert permissions on the `unique_name` column only, and `update` is permitted, but not on any of the specific columns.
- Go to the GraphiQL editor and set the `X-Hasura-Role` header to `user`
- Run the following query:

  ```graphql
  mutation InsertName {
    insert_names_one(
      object: { unique_name: "foo" }
      on_conflict: { constraint: names_unique_name_key, update_columns: [] }
    ) {
      id
    }
  }
  ```

- Observe that the response is:

  ```json
  {
    "data": {
      "insert_names_one": null
    }
  }
  ```

- Kill the Hasura console process and run `docker compose stop` to stop the containers.

Now we will upgrade to Hasura 2.0 and try the same query.

- Run `docker compose --file docker-compose.2.0.yml up -d` to bring the Docker containers up with Hasura 2.0 instead.
- Run `hasura console` to launch the Hasura console
- Go to the GraphiQL editor and set the `X-Hasura-Role` header to `user`
- Try to run the same GraphQL mutation as above. Observe that the following error is returned:
  ```json
  {
    "errors": [
      {
        "extensions": {
          "path": "$.selectionSet.insert_names_one",
          "code": "validation-failed"
        },
        "message": "\"insert_names_one\" has no argument named \"on_conflict\""
      }
    ]
  }
  ```
