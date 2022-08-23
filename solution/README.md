# KWT22 Docker Workshop - Solutions

This directory contains example solutions.

Note that you either have to copy the Dockerfiles to the root of your build context or specify them in your build context like

```
docker build -t kwt22-image:prod --target prod -f solution/step4/Dockerfile .
```