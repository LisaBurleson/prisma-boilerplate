# Quick Prisma boilerplate project
(note: this requires docker to be installed on the local machine)
- I installed prisma globally

```
npm install -g prisma
```

- in an empty direcotry use the prisma cli and follow the prompts
- I chose to use a docker container and a postgres new database
```
prisma init prisma-boilerplate
```
Three files are generated:
- prisma.yml
- datamodel.prisma (a subset of the GraphQL SDL -Schema Definition Language)
- docker-compose.yml (configured for two images, one for the prisma server and the other for postgres)

I then modified the `docker-compose.yml` to use a specific database/schema.
I don't like the postgres default names, "default/$default". Can't even execute queries...
```yaml
  prisma:
    environmtent:
      PRISMA_CONFIG: |
        database:
          default:
            ...
          database: prisma
          schema: {desired schema name}
```

Update the `prisma.yml` to generate client code every time schema changes are deployed to the prisma server.

```yaml
generate:
  - generator: javascript-client
    output: ./generated/prisma-client/
    
hooks:
  post-deploy:
    - prisma generate
```

Prisma uses a "schema/SDL-first" methodology [see https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3 ]

The development lifecycle will be:
  1. **precond**: servers are running via `docker-compose up -d`
  1. update the schema in datamodel.prisma
  1. push the changes to the prisma server via `prisma deploy`, look for any errors
  1. repeat as often as desired
  1. you can also generate the javascript/typescript client code too via `prisma generate` and it will create the files in the generated dir
  1. **postcond**: when done with development, `docker-compose down`

If the server isn't running but you don't see any errors, peek into the prisma docker container by:
1) `docker ps` to find the prismagraphql container id
2) `docker logs -f {container id}` (only the first few chars are sufficient) 

I wanted to remove my past iterations of database data saved on the local filesystem, but that is manage by docker, so:
  * `docker volume ls` to find the volumes created
  * `docker volume rm {vol 1} {vol 2} ...{vol N}` to remove all the un-needed data

## links
  * https://www.prisma.io/docs/get-started/01-setting-up-prisma-new-database-JAVASCRIPT-a002/
  * https://bit.ly/2VmMuw9

dataloader:
 - https://www.youtube.com/watch?v=ld2_AS4l19g&t=57s
 - https://www.youtube.com/watch?v=_FQ1ZEWIn2s

