# SvelteKit with Mongodb (power by [Mongoose](https://mongoosejs.com/) )

This is simple CRUD in one page. 
You have to copy files to correct location of svelte project 
for running this demo.  
SvelteKit make some change for POST method. 
I use GET method for avoiding break change and remake tutorial.
You can check Form Actions tutorial for POST method.
I don't want to make complex demo so I use similar UI and schema for most of database demo. You can learn another dabase in few minute.


![Product UI](../../../asset/product-ui.png)

## File
- docker-compose.yml
- src/routes/mongoose/product.js
- src/routes/mongoose/+page.server.js
- src/routes/mongoose/+page.svelte

## install
    npm create svelte@latest my-app
    cd my-app
    npm install
    npm install mongoose
    # copy file to project
    docker compose up -d mongo
    code .

Browse to [http://localhost:4000](http://localhost:4000)

## note
I found error mongo not found error some time. I don't know how to fix it yet. Some annoy problem can fix by. 
- npm update
- reload visual studio code

please check [main document](https://github.com/schooltechx/youtube/tree/main/svelte/svelte-kit) for other tutorial
