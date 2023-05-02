--------------------------------------------------------------------------------------------------------------------------------------------------------------------
                                                          Session
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
    var session = require('express-session');
    var FileStore = require('session-file-store')(session);
    var fileStoreOptions = {};
    import nextSession from "next-session";
    import { promisifyStore } from "next-session/lib/compat";
    const connectStore = new FileStore(fileStoreOptions);
    const getSession = nextSession({
        store: promisifyStore(connectStore),
        secret: 'keyboard cat',
        resave: true,
        saveUninitialized: true,
    });
    export async function getServerSideProps({ req, res }) {
        const session = await getSession(req, res);
        session.views = session.views ? session.views + 1 : 1;
    }

--------------------------------------------------------------------------------------------------------------------------------------------------------------------
                                                New file access after build - using Server.js
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
    const express = require("express");
    const next = require("next");

    const port = parseInt(process.env.PORT, 10) || 2121;
    const dev = process.env.NODE_ENV !== "production";
    const app = next({ dev });
    const handle = app.getRequestHandler();

    app.prepare().then(() => {
        const server = express();

        server.use("/media", express.static(__dirname + "/public"));

        server.all("*", (req, res) => {
            return handle(req, res);
        });
        server.listen(port, (err) => {
            if (err) throw err;
            console.log(`> Ready on http://localhost:${port}`);
        });
    });

    => package.json
        "scripts": {
            "dev": "node server.js",
            "build": "next build",
            "start": "NODE_ENV=production node server.js",
            "lint": "next lint"
        }
    => disallow server-side routes, To disable, open next.config.js and disable the useFileSystemPublicRoutes config:
        module.exports = {
            useFileSystemPublicRoutes: false,
        }
    => disallow client-side redirects to filename routes[https://nextjs.org/docs/api-reference/next/router#routerbeforepopstate]
        router.beforePopState(cb)
        
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
                                                New file access after build
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
    => Files // for file storage
    => Pages/_middleware.js // for rewrite url
        import { NextResponse } from 'next/server'
        export function middleware(request) {
          const url = request.nextUrl.clone()
          if((request.nextUrl.pathname).includes('/stuff/')){
            url.pathname = '/api' + request.nextUrl.pathname
            return NextResponse.rewrite(url)
          }
        }

    => Pages/api/stuff/[...slug].js // for access images
        export const config = {
            api: { externalResolver: true }
        }
        import express from 'express';
        const handler = express();
        const serveFiles = express.static('public');
        handler.use('/stuff', serveFiles);
        export default handler;

    => Load this url : http://localhost:3000/images/forgot_password.png
