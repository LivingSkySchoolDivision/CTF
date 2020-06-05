# Components

 - **Multi-Juicer** (https://github.com/iteratec/multi-juicer)
   - Multi-user OWASP Juice Shop (https://github.com/bkimminich/juice-shop)
 - **RootTheBox** for scoreboard

# Installing

## Server

This will require a fully working kubernetes server, which is well beyond the scope of this document.
It can easily run on a single-node server, with enough resources.

For resources, I would recommend:
 - At least 2 CPU, plus at least 0.25 CPU for each player
 - At least 1.5GB RAM, plus 200MB RAM for each player
 - Negligable storage space, over-and-above kubernetes requirements. 10GB is lots.

This setup requires a kubernetes ingress controller to exist on your kubernetes server.

## Set up Scoreboard

## Set up Multi-Juicer