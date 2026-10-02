+++
title = "Lottom - Grocery Tracker"
date = "2026-10-02"
+++
## Updates

**Version 0.3.0** as the first semi-stable version has been released. More
information on how to use this as a docker container can be found
[here](https://code.siru.ink/siru-ink/lottom/src/branch/main/docker-compose.yml).

## Description

Lottom is a graphically simple, multi-list, multi-user grocery tracking web app.
It's interface is designed to be both usable on wide-monitor desktops as well as
phone screens while shopping on-the-go. The backend of the server is written in
Rust lang based on the Axum web framework coupled together with the sqlx
database management framework. The "frontend" is mostly server-side-rendered
HTML with just a sprinkle of JavaScript for ease-of-use. However, it can
function in fully JavaScript free environments without any functions of the
server not being accessible.

The project is currently still in it's initial stages, hence a less than v1
semantic verisioning number, but is stable enough to be useful in day to day
operations. No guarantees are made at this point about the stability of the
database layout, and future version bumps will most likely not be compatible
with the existing database migrations and will require a full start from
scratch.

## Lineage

Originally, this project was known under the name List
([source code here](https://code.siru.ink/siru-ink/listy)). Listy was based on
a Django multi-app architecture. In the beginning, this worked well, as using
Python as the base language coupled together with a full ORM system made getting
the first verison ready for usage was simple. However, after spending a while
away from the project, I realized that the distributed structure of Django
projects made the mental load of coming back to the project much higher.

Coupled together with this was also the fact that Django is more so designed to
circumvent static file hosting in favor of a reverse proxy filter to send static
asset requests to an S3 based storage host. For large applications this is
undoubtedly a better solution than storing things locally, however for a small
grocery tracker storing 10s of pictures for the shopping list this added
unnecessary complexity overhead.

Based on these two reasons, I decided to rewrite the project with type-gurantees
and with a less obtrusive ORM system using Rust as the language and Sqlx's
`query_as!` macros that can check database type-safety at compile/linting stage
instead.
