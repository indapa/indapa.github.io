---
layout: post
title: "My first GitHub Actions"
date: 2025-04-08
author: "Amit Indap"
tags:
  - Python
  - GitHub Actions
  - CI/CD
---

I've been meaning to learn how to use [GitHub Actions](https://docs.github.com/en/actions/about-github-actions/understanding-github-actions) for a while now. I recently re-factored some code to run in a Docker container. Each time I updated the Python code, I had to re-build the Docker image and push to Docker Hub. A couple of times I forgot to do this, and I would run the code in the container, only to find out that my changes were not reflected in the image I was using. 

After some Google searching, I found this very helpful [video](https://youtu.be/x7f9x30W_dI?feature=shared) from [Rishab in Cloud](https://www.youtube.com/@rishabincloud) building and pushing Docker images with GitHub Actions. GitHub will actually recommend workflows based on your code. I just modified the template based on the video I watched, and it worked great. 

Two things to note. First,  make sure and add your Docker Hub credentials to your GitHub secrets. You can do this by going to your repository, clicking on Settings, then Secrets and variables, then Actions. Click on New repository secret. Then I simply added my Docker Hub PAT. Second, make sure your branch name is correct. I was using the default branch name of `main`, but my branch is actually `master`. Once I fixed that, it worked great.

Here is the [YAML file](https://github.com/indapa/indapa-CellXGene/blob/master/.github/workflows/docker-image.yml) I used to build and push the Docker image. 

```yaml

name: Docker Hub CI

on:
  push:
    branches:
      - master
    paths:
      - 'Dockerfile'
      - 'bin/**'

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Build the Docker image
        run: docker build . --file Dockerfile --tag indapa/indapa-cellxgene:latest

      - name: Push image to Docker Hub
        run: |
          docker login -u indapa -p ${{ secrets.DOCKERHUB_PAT }}
          docker push indapa/indapa-cellxgene:latest

```

My workflow log is  [here](https://github.com/indapa/indapa-CellXGene/actions) and you can see I've had one successful run so far.



